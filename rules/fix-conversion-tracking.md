
# Fix conversion tracking with Zernio

Goal: every ad network the site runs reaches `status: "good"` (server-side verified) in the Conversion Leak Report, with the score higher than at the start.

The Zernio MCP server lists few tools up front. Find the others with `search_tools` and run them with `call_tool`, using the exact names below. Without the MCP server, call the REST API at `https://zernio.com/api/v1` with `Authorization: Bearer <API key>`; each step names its route.

Ask the user before you change production code, deploy, or change an ad account. Never guess an id: read it from a tool result.

## Before you start

- The user needs a Zernio account (https://zernio.com/signup) and the MCP server signed in, or an API key from https://zernio.com/dashboard/api-keys.
- Ad tools need the Ads add-on, included with usage-based billing. A `403` "Ads add-on required" means it is off: tell the user, don't work around it.
- You need write access to the website's code, and the user needs a login to each ad account.

## 1. Scan

`conversion_audit_audit_conversion_tracking` with `url` (REST: `GET /v1/conversion-audit?url=`). Write down:

- `report.score` and `report.scoreRange`.
- For each entry of `report.platforms` with `browserPixel: true`: `key`, `status`, `serverSideStatus` and `ids` (the pixel or tag ids on the page).

A platform with `status: "good"` is done. The others are the ones to fix. `fixPrompt` lists the same findings in prose.

## 2. Pick the profile and connect the ad accounts

1. `profiles_list`, and ask the user which profile the site belongs to when there is more than one.
2. `accounts_list` with that `profile_id` and the ads platform (`metaads`, `googleads`, `tiktokads`, `linkedinads`, `pinterestads`, `xads`, `redditads`, `microsoftads`, `snapchatads`, `openaiads`). Keep each ads account's id.
3. For a network with no ads account: `connect_ads` with `profile_id` and `platform` (`facebook` for Meta, `googleads`, `tiktok`, `linkedin`, `pinterest`, `twitter`, `redditads`, `microsoftads`) returns an `authUrl`. Give it to the user to open in their browser, wait until they say it is done, then list the accounts again. Google Analytics 4 connects with `connect_get_google_analytics_connect_url` instead (then `connect_list_google_analytics_properties` and `connect_select_google_analytics_property`). Snapchat connects with its Snap Pixel ID and a Conversions API token (`connect_snapchat_ads_credentials`), OpenAI Ads with an API key from ChatGPT Ads Manager (`connect_open_ai_ads_credentials`): ask the user for them.

## 3. Create the tracking site

First `conversions_list_tracking_sites` with `profile_id`: reuse a site whose `domains` already hold the scanned host.

Otherwise `conversions_create_tracking_site` with `idempotency_key` (a fresh UUID; reuse it on a retry) and `body`:

- Any website: `{ "profileId": "...", "url": "https://brand.com" }`.
- A Shopify, WordPress (WooCommerce), Wix or Webflow store connected to the profile: `{ "profileId": "...", "storeAccountId": "..." }`. Zernio installs the tag through the store's app, so skip step 4. A `409` `store_already_tracked` carries the existing site in `details.siteId`.

REST: `POST /v1/tracking-sites` with an `Idempotency-Key` header. Keep `site.id` and `site.install.snippet`.

## 4. Install the tag

In the site's code, before `</head>` on every page (the shared layout or document template):

```html
<script>if (typeof window.zernio !== 'function') window.zernio = function () { (window.zernio.q = window.zernio.q || []).push(arguments); };</script>
<!-- then site.install.snippet, exactly as returned -->
```

Where the site converts, send the conversion with the same id its pixel uses, so each network counts it once:

```js
const eventId = 'order_' + order.id;
zernio('track', 'Purchase', { eventId, value: order.total, currency: 'USD', orderId: String(order.id) });
fbq('track', 'Purchase', { value: order.total, currency: 'USD' }, { eventID: eventId });
```

- Conversions are `Purchase`, `Lead`, `CompleteRegistration`, `Subscribe`, `StartTrial`, `SubmitApplication`, `Schedule` and `Contact`. Only these are stored and sent to Google Ads and LinkedIn.
- GA4 and Elevar ecommerce events the site already pushes to `dataLayer` (`purchase`, `generate_lead`, `sign_up`...) are read without code: don't send them twice.
- After a form or login, `zernio('identify', { email, phone, externalId })` raises match quality. LinkedIn drops events without an email, an external id or its click id.
- The tag reads the site's cookie banner and, with the site's `consentModeBridge` on (the default for new sites), writes the banner's answer into Google Consent Mode v2. Without ad storage consent nothing is forwarded: don't bypass the banner.
- A Content Security Policy must allow `edge.zernio-edge.com` in `script-src` and `connect-src`.

Show the user the diff, and deploy only when they agree.

## 5. Serve the tag from the site

Do this when the scan shows cookies Safari cuts to 7 days. A path of the site itself, forwarded to Zernio, sets the cookies from the site's own server, which Safari keeps.

1. `conversions_update_tracking_site` with `site_id` and `body: { "proxyPath": "/zt" }` (REST: `PATCH /v1/tracking-sites/{siteId}`), using a path the site does not serve pages on. Keep `site.siteKey` and `site.proxy.secret`.
2. Add the ready-made proxy for the site's platform (Next.js, Vercel, Cloudflare Workers, Netlify, nginx, WordPress) from https://docs.zernio.com/products/server-side-tracking/own-domain-path, with `ZERNIO_SITE_KEY` and `ZERNIO_PROXY_SECRET` set to those two values. A plain rewrite rule is not enough: it cannot sign the request.
3. Replace the tag with the new `site.install.snippet`, which loads from the site's own path.
4. After the deploy, `conversions_check_tracking_site_proxy` with `site_id` (REST: `POST /v1/tracking-sites/{siteId}/proxy-check`) until `proxy.status` is `verified`.

A Shopify store can do this only when its domain runs through its own Cloudflare zone (the Cloudflare Worker proxy on `store.com/zt/*`). Otherwise skip this step and tell the owner Safari's cap stays.

## 6. Forward to each network's own pixel

`conversions_list_forwarding_rules` with `profile_id` first: a rule to the right destination may exist already.

For each network to fix, `conversions_list_ad_conversion_destinations` with the ads `account_id` (REST: `GET /v1/ads/conversions/destinations?accountId=`) and pick the destination that matches the page:

| Network | Destination to pick |
|---|---|
| Meta | The pixel whose `id` equals the id the scan read (`ids`) |
| TikTok | `pixel:<code>` for the pixel code the scan read, not an Offline Event Set |
| Google Ads | A conversion action per conversion you send, in the account behind the page's `AW-` tag (create one with `conversions_create_ad_conversion_destination` if none fits) |
| Google Analytics 4 | The `G-...` measurement id the scan read. On a page that also runs an `AW-` tag, only a Google Ads rule verifies the network |
| LinkedIn | A conversion rule per conversion you send |
| Others | The pixel the page runs |

Then `conversions_create_forwarding_rule` with `profile_id`, `account_id`, `destination_id` and `sources: ["web"]` (REST: `POST /v1/events/forwarding-rules`). For Google Ads and LinkedIn add `event_names`, such as `["Purchase"]`, one rule per conversion action. A `409` means the rule exists: keep it.

If the destination the page runs is not in the list, the ads account the user connected doesn't own that pixel: ask them to connect the one that does. Never forward to another pixel; the rescan would not verify it, and the network would count the conversions on a pixel the site does not use.

## 7. Load the pixels behind the banner

Do this when the scan shows pixels firing before consent, or no Google Consent Mode v2.

`conversions_update_tracking_site` with `site_id` and `body` holding the site's `pixels`, one per pixel id the scan read, and `consentModeBridge: true`:

```json
{ "pixels": [{ "platform": "meta", "id": "1234567890123456" }, { "platform": "google", "id": "G-TEST1234" }], "consentModeBridge": true }
```

`platform` is `meta`, `google` (`AW-` or `G-` ids), `tiktok`, `linkedin`, `pinterest`, `snapchat`, `reddit`, `x` or `microsoft`. The list replaces the one before.

Then remove the hand-installed copies of those pixels from the page code, Tag Manager and store apps, so each loads once, through the tag. Show the user the diff first.

In the EEA, UK and Switzerland the tag loads them only after the cookie banner grants consent. Visitors who decline stay untracked: nothing recovers their conversions. See https://docs.zernio.com/products/server-side-tracking/pixels.

## 8. Verify the events arrive

After the deploy, ask the user to open the site in a normal browser, accept cookies, and make one test conversion. Headless browsers are flagged as bots and their conversions are never stored or forwarded, so a headless test proves nothing.

1. `conversions_get_tracking_site` with `site_id`: `setup.status` turns `ready` with the first real visit. Still `pending`: the snippet is not on the live page, or no real browser loaded it yet.
2. `conversions_list_events` with `profile_id` and `source: "web"`: the test conversion is listed, with one delivery per network and `status: "sent"`. None listed: no ad storage consent, or the page's host is not in the site's `domains` (`conversions_update_tracking_site`).
3. `conversions_get_event` with the event's `eventId` and `profile_id` shows the platform's error for a failed delivery. `conversions_get_events_overview` with `profile_id` shows each rule's `sentToday` and `lastError`.

Tell the user where to see it on each platform too: Meta Events Manager (events received from Server, about 20 minutes), TikTok Events Manager (about 30 minutes), Google Ads conversion diagnostics (hours).

## 9. Rescan

`conversion_audit_audit_conversion_tracking` again. A network turns `status: "good"` with `serverSideVerified: true` once the site's events reached that network's destination in the last 7 days; Meta, TikTok and GA4 also need it to be the pixel or measurement id on the page. Report the old and new score and each network's status.

A network still at `status: "warn"` with `serverSideStatus: "likely"` and `relayedBy: "zernio-tag"` means the tag is on the page but no matching delivery was found: recheck step 6 (the destination id must equal the page's id) and step 8. Without `relayedBy`, the scan also sees another server-side tool (such as a server Tag Manager container) and cannot verify the Zernio tag alone: tell the user, don't loop.

## How the scan credits it

- On a page whose Zernio tag runs the Consent Mode bridge, the scan passes Consent Mode.
- When every pixel on the page is one the tag loads, and the scan observed no consent timing of its own, it marks the pixels as waiting for consent by the tag's settings, not as observed. What a browser in those countries observes always wins.

## If it fails

- `proxy.status` is `unsigned`: the proxy reaches Zernio without a valid signature. Set the current `proxy.secret` in the proxy and redeploy, then check again.
- `proxy.status` is `unreachable`: the proxy is not deployed, or forwards another path. Open `https://<domain>/zt/z.js` to check.

## What this does not do

- It forwards the site's browser conversions server-side. Conversions that only your backend knows (renewals, offline sales, CRM stages) go to `POST /v1/events` (`conversions_send_events`) with the same `eventId` the browser used; see https://docs.zernio.com/products/conversions-api.
- It cannot see or change an ad network's own settings beyond the pixel and conversion actions above.

Full guide: https://docs.zernio.com/products/server-side-tracking/fix-with-agent
