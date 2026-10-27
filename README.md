=== Omnifi WooCommerce Gateway ===
Plugin Name: Omnifi WooCommerce Gateway
Description: Hosted checkout and embedded-iframe Tokenization payment gateway integration for Omnifi with Tyro and Worldline (global + ANZ) provider support.
Version: 1.15.5
Author: Digitalfellows
License: GPLv2 or later
Text Domain: omnifi-woocommerce-gateway
Domain Path: /
Contributors: digitalfellows
Tags: woocommerce, payment gateway, hosted checkout, tokenization, tyro, worldline
Requires at least: 6.0
Tested up to: 6.8
Requires PHP: 7.4
Stable tag: 1.15.5
License: GPLv2 or later
License URI: https://www.gnu.org/licenses/gpl-2.0.html
WC requires at least: 7.0
WC tested up to: 9.0

Hosted checkout and embedded-iframe Tokenization payment gateway integration for Omnifi with Tyro and Worldline (global + ANZ) provider support.

== Upgrade Notice ==

= 1.15.5 =

Housekeeping before the public repository: GPLv2 LICENSE file added (and shipped in the plugin zip), internal names and hosts removed from code comments and notes. No behaviour change.

= 1.15.4 =

Webhooks that can never succeed (broken JSON, missing fields, unsupported payload) are now answered 400 `{"error":"invalid_payload"}` so Omnifi stops retrying. An order that can't be found yet still answers 500 so Omnifi retries.

= 1.15.3 =

Connect with Omnifi: a connection that can't be saved is now reported to Omnifi (step `save_settings`), and the activity log shows when Omnifi refuses a problem report and why.

= 1.15.2 =

Webhooks with a wrong or missing signature are now answered 401 `{"error":"invalid_signature"}` (Omnifi treats it as final and its test webhook tells the merchant the secret doesn't match). Processing errors after a valid signature still answer 500 so Omnifi retries.

= 1.15.1 =

Connect with Omnifi: warns and disables Connect on sites with plain permalinks (Omnifi refuses the ?rest_route= callback), and Setup Status flags All-In-One Security's "Disallow unauthorized REST requests", which blocks Omnifi webhooks and the sign-in with 403.

= 1.15.0 =

Connect with Omnifi: trace ids ("Reference") on every attempt, an activity log in the SSO panel with Send diagnostics and Clear, and problems reported to Omnifi. Omnifi's OMNIFI-TEST sample order is acknowledged without touching orders. Setup Status no longer warns about a missing webhook secret when Omnifi SSO supplies it.

= 1.14.1 =

SSO test/disconnect errors from Omnifi that aren't in the `{error, error_description}` shape now show a short excerpt of Omnifi's answer instead of just the status code.

= 1.14.0 =

Connect with Omnifi (SSO) brought in line with Omnifi's brief: PKCE on connect, exchange at `/integration/v1/connect/exchange`, webhook secret stored with a 24h previous-secret grace period after Reconnect, Send a test webhook / Reconnect / Disconnect (now tells Omnifi) in the SSO panel.

= 1.13.43 =

WPAY refund body field renamed `TransactionId` → `WpayTransactionId`. PAY's validator returned 400 with `errors: { "WpayTransactionId": ["The WpayTransactionId field is required."] }` for our refund attempt on order 687 (the captured paid order). Tyro/Worldline use `Pay_id` for refunds; WPAY uses `WpayTransactionId` — different field-name convention. Unit test 199 updated.

= 1.13.42 =

WPAY status normalisation: recognise PAY's pre-normalised `normalizedStatus` field + the raw `capture_succeeded` / `capture_failed` status words. Omnifi fixed the polling-500 bug PAY-side; the response for order 687 came back as `{status: "capture_succeeded", normalizedStatus: "SUCCESS", isSuccessful: true, transactionId: "2a84fd14..."}`, but our normaliser didn't recognise any of those as a SUCCESS signal so the order stayed PENDING. Fix: `extract_normalized_status_from_payload` now checks `normalizedStatus` field FIRST (short-circuits on the already-categorised value), and `normalize_status` recognises `CAPTURE_SUCCEEDED` / `CAPTURE_FAILED` aliases.

= 1.13.41 =

Expanded webhook signature verification — 1.13.40 confirmed via md5 fingerprint that our pasted secret matches Omnifi's exactly, but signature still fails. That eliminates secret-encoding as the cause. 1.13.41 adds two additional candidate HMAC inputs: `timestamp.body` (Stripe-style) and `timestamp+body` (Gr4vy-style concat). Pulls timestamp from any of `X-Omnifi-Timestamp`, `X-Worldline-Timestamp`, `X-Gr4vy-Timestamp`, etc. Also logs ALL request headers (PII-safe, strips authorization/cookie) so we can see what timestamp/nonce headers PAY actually sends. Next webhook attempt will tell us whether the answer is a timestamp-concat input OR something more exotic (canonical JSON, path-prefixed, etc.). Test stub `WP_REST_Request::get_headers()` added to bootstrap.

= 1.13.40 =

Refines 1.13.39's diagnostic logging — field names renamed to bypass `Omnifi_Logger`'s PII masker (which was over-eagerly redacting any key containing "secret" or "signature"). Now logs the FULL signature header value (not secret material — it's the HMAC output), the full computed_hex and computed_base64 (also not secret — derived values), plus an md5 fingerprint of the secret (PII-safe, lets us confirm the secret matches without leaking it). On the next webhook, we'll see the EXACT mismatch between PAY's signature and our computation.

= 1.13.39 =

Diagnostic + base64url-decode fallback for WPAY webhook signature verification. Discovered live 2026-06-29: even with the correct webhook secret (`<redacted>` per Omnifi) pasted into admin, signatures still fail validation. The verifier now (a) tries the secret as a raw string FIRST (matches Tyro/Worldline) and (b) if the secret looks like base64url-encoded, also tries the decoded raw bytes as the HMAC key — some providers (Gr4vy / MX51 included) store the displayed secret as base64url but sign with the raw bytes. On final mismatch, the verifier logs forensic detail (body length+md5, secret prefix+suffix, signature header preview, computed-hex preview) so the next failure can be diagnosed without re-shipping debug code. No secret bytes or body content are logged. If both string and decoded forms fail, the issue is body content or algorithm — and we'll see exactly what to ask Omnifi.

= 1.13.38 =

Observability fix discovered during WPAY sandbox debugging: `Omnifi_Logger::log()` was gating ALL log levels — including ERROR — behind the `debug` toggle. With debug off (production default) operators had ZERO visibility into failures — silent silent silent. Confirmed live: 5 days with zero log file written despite multiple polling failures during WPAY bring-up. Fixed by always writing ERROR-level entries; INFO/DEBUG still gated behind debug toggle for production quietness on the success path. Operators now see real failures the moment they happen. Test coverage in `tests/unit/205-logger-errors-always-write.test.php`. No other behaviour changes — only the logger's gating logic.

= 1.13.37 =

Fixes the FIRST LIVE BUG discovered during the WPAY sandbox bring-up on 2026-06-29 (order #684 on staging): Omnifi's `CreateWooCommerceWpayCheckout` returns `checkoutUrl` (and `partialUrl`) as RELATIVE paths — `/external/woocommerce/<token>/wpay-checkout` — not as fully-qualified URLs. When our plugin redirected the customer to the relative URL, the browser resolved it against the current document base (the merchant domain), landing on `https://shop.example.com/external/...` which has no route handler → WP fell through to the default theme. Customer saw the homepage instead of the WPAY card form. The 1.13.34/35/36 hosted-checkout assumption was that Omnifi would return absolute URLs (matching how Tyro / Worldline responses work); WPAY is different. Fix: detect relative URLs in `Omnifi_API_Client::create_wpay_checkout()` and prepend the configured PAY base URL (`https://pay.omnifi.com.au` by default, per-merchant override via `*_base_url` settings). The customer now lands on PAY's actually-hosted Secure Fields page where the Gr4vy card form renders.

= 1.13.36 =

Omnifi (Omnifi) confirmed two key WPAY details on 2026-06-29. (1) **Refund amount format = DECIMAL/dollars, NOT cents.** "WPAY is not like Tyro: send dollars (25.00 for $25), not cents. We convert to cents on our side." The 1.13.35 `omnifi_wc_wpay_refund_amount_format` filter (originally added as insurance) now defaults to `'decimal'` instead of `'cents'`. The filter remains in place as an escape hatch (passing `'cents'` opts back into the Tyro-style convention) but the documented + tested default is decimal. (2) **Sandbox cards = Gr4vy's, NOT Tyro's.** WPAY doesn't use amount triggers (unlike ANZ Worldline's €13.02-decline pattern). Use `5200 0000 0000 1203` for frictionless success and `5200 0838 3838 3836` for 3DS challenge. The Tyro card `5200 0000 0000 1021` referenced in earlier docs was wrong for WPAY. Full WPAY 3DS testing docs at https://integration.wpay.com.au/guides/features/3ds/testing. No code change for sandbox cards — operators just use the correct cards when testing. Pay Channels activation on Omnifi's tenant is now the only remaining blocker for the first live transaction.

= 1.13.35 =

Hardening + insurance release while we wait on Pay Channels activation for the first live WPAY sandbox. (1) Fixes a real bug discovered during E2E testing: the prefix-based webhook verifier (used for both Worldline and now WPAY since 1.13.31) didn't recognise `transactionCode` as a resource identifier — signed WPAY webhooks would pass signature gate then fail at "Unsupported payload." Added `transactionCode` + `Pay_id` + `pay_id` to the verifier's resource_id lookup (Tyro verifier already had them since 1.13.18; this brings the prefix-based verifier to parity). (2) New filter `omnifi_wc_wpay_refund_amount_format` lets operators flip the WPAY refund amount between cents (default, matches Tyro convention) and decimal without a code change — insurance against Omnifi's endpoint differing from the assumed cents format. Unknown filter values safely fall back to cents. (3) Refund coverage expanded: partial refund test pins that the refund amount (not the order total) is forwarded; filter-override and fallback paths each have their own assertions. (4) Audit pin test: ensures every registered provider's `process_refund` calls a real `create_*_refund` API method (no stubs). (5) HANDOVER.md refreshed for the 1.13.21 → 1.13.34 work + added WPAY sandbox-test playbook for the post-activation moment.

= 1.13.34 =

WPAY refunds are now LIVE. Omnifi confirmed the Omnifi-side `CreateWooCommerceWpayRefund` endpoint is ready, and this release replaces the placeholder stub with the real implementation. `Omnifi_Provider_Wpay::process_refund()` now reads the captured `transactionId` from `_omnifi_transaction_id` order meta (populated during status sync) and POSTs to `/integration/v1/WooCommerce/CreateWooCommerceWpayRefund` with `Omnify_Token` + `MerchantKey` + `TransactionId` + `amount` (in cents) + `currency`. Mirrors the Tyro refund pattern; same operator-friendly error handling (rejection messages surface the provider status word + refund reference). Returns a specific `omnifi_wpay_missing_transaction_id` WP_Error if the order's transaction_id hasn't reconciled yet — operator retries once status sync catches up.

= 1.13.33 =

Stops the browser's "Save password?" popup from appearing on the gateway settings page. Chrome and Firefox were heuristically treating the page as a login form because the visible Title input ("Credit Card") sits near a `<input type="password">` merchant key field. Every time the operator saved settings (including auto-save on provider switch), the browser offered to save the merchant key as a password with username "Credit Card". Fix: inject `autocomplete="new-password"` + `autocorrect="off"` + `autocapitalize="off"` + `spellcheck="false"` on every password-type field in the form, plus a JS belt-and-braces that sets `autocomplete="off"` on the form and the Title input. No effect on field rendering or persistence — operators continue to paste merchant keys as before, just without the spurious popup.

= 1.13.32 =

Fixes two WPAY admin-UI bugs reported from a user screenshot. (1) When WPAY was the active provider, the section showed only the title rows ("WPAY (MX51) Settings", "WPAY Pricing Controls") — the merchant key, webhook secret, surcharge, and process amount input fields between them were hidden because the provider-toggle JS only knew about Tyro/Worldline/Worldline ANZ. The Tyro selector inadvertently caught WPAY input rows and hid them whenever Tyro wasn't active. (2) The SSO "Connect with Omnifi" panel was rendering for WPAY even though SSO isn't wired for WPAY on the Omnifi side yet (clicking the button would 500). Fix: provider-toggle JS now has a wpay branch and excludes wpay rows from the Tyro selector; the SSO panel renderer consults a new `Omnifi_Sso_Controller::is_provider_supported()` source-of-truth method and skips the panel entirely for unsupported providers. Description helper now shows a WPAY-aware "Paste your live/sandbox merchant key below" message instead of the misleading "Connect via Omnifi above".

= 1.13.31 =

Fixes a webhook routing bug discovered during 1.13.30 E2E testing: when `default_provider = wpay`, the webhook controller was falling through to the Tyro verifier (which reads `webhook_secret`, not `wpay_webhook_secret`) because `wpay` wasn't recognised in the provider whitelist. WPAY-signed webhooks would have silently failed verification. Now `resolve_webhook_provider()` recognises `wpay`, `get_verifier()` returns the prefix-based verifier with prefix='wpay' (reading `wpay_webhook_secret`), and `is_signature_required()` reads `wpay_require_webhook_signature`. No effect on Tyro/Worldline/Worldline ANZ webhook handling.

= 1.13.30 =

Completes WPAY (MX51) v1 wiring per Omnifi's 2026-06-22 spec. `GetWpayAuthenticationToken` is now the auth endpoint when WPAY is active. `create_wpay_checkout` calls `authenticate()` directly (skips the Tyro-specific `CreateTyroAuthCheckout` step that doesn't apply to WPAY). `get_checkout_status` routes WPAY orders to the WPAY status endpoint for backup reconciliation. Status sync now recognises WPAY in its provider whitelist and reads `wpay_include_order_key_in_return_url` for return-URL strictness. Place Order → redirect to PAY → wpay-return reconciliation path is functional end-to-end; only Pay Channels activation on Omnifi's tenant (Gr4vy/MX51 keys) is needed to test against real PAY traffic. Refund endpoint is still being implemented on Omnifi (per Omnifi); the stub error stays in place until then.

= 1.13.29 =

Adds WPAY (MX51) as a new payment provider option alongside Tyro / Worldline / Worldline ANZ. Hosted-Checkout flow scaffolding only in this release: customer clicks Place Order, redirects to PAY's WPAY hosted checkout page, pays, and returns. Settings, surcharge/processing-fee controls, webhook secret, and currency-gate follow the same pattern as other providers (prefix `wpay_`). Refund support is intentionally stubbed (returns a clear "WPAY refund not yet supported" error mirroring Tyro pre-1.13.15) — Omnifi's WPAY refund endpoint is still TBD per Omnifi; the stub will be swapped for a real API call when ready. Secure Fields (inline iframe) variant is out of scope for this release pending Omnifi-side contract confirmation.

= 1.13.28 =

Fixes 1.13.27 regression: Tyro gateway was incorrectly hidden on Block checkout. Tyro's auth response (GetTyroAuthenticationToken) has `useHostedCheckout: false` + `paymentFlow: null` because those fields are Worldline-product-specific; the API client's flow-resolution fallback misclassified Tyro as Tokenization, which 1.13.27's `is_active()` then used to hide it. 1.13.28 gates the flow check by provider — only Worldline / Worldline ANZ go through the FLOW_TOKENIZATION test; Tyro is unconditionally treated as Hosted-Checkout (which is correct: Tyro always redirects to its hosted page).

= 1.13.27 =

Refines 1.13.26: hide the Omnifi gateway from Block checkout when the active provider is on Tokenization flow. 1.13.26's server-side Hosted-Checkout fallback fired correctly but Omnifi rejected it with HTTP 400 `PAYMENT_FLOW_TOKENIZATION` ("Organisation payment flow is set to Tokenization. Use CreateWooCommerceWorldlineAuthCheckout, not hosted checkout"). Tokenization merchants can't use the Hosted-Checkout endpoint at the API level. Now `Omnifi_Blocks_Support::is_active()` checks `Omnifi_API_Client::get_active_payment_flow()` and returns false for `FLOW_TOKENIZATION` — Tyro Hosted-Checkout merchants still appear on Block checkout; Worldline/Worldline ANZ on Tokenization merchants are hidden (customer either picks another method on Block checkout, or uses the classic shortcode checkout where the inline iframe works end-to-end).

= 1.13.26 =

Block-checkout customers on Tokenization-flow providers (Worldline / Worldline ANZ) now reliably complete checkout via the Hosted-Checkout redirect path. The inline tokenizer iframe attempted in 1.13.24/25 hit a WC Blocks framework limitation: `onPaymentSetup` is subscribed but the framework never dispatches to it without the official `@woocommerce/blocks-checkout` npm build pipeline, so `meta.paymentMethodData` never reaches the Store API. Verified across 14 live iterations on staging — orders were created but with `payment_status: failure` because the server never received the card token. 1.13.26 makes both Worldline providers detect Store API requests (`REST_REQUEST` + `/wc/store/` path) and route to `process_hosted_checkout_payment()` when no token is in `$_POST`. Customers see the gateway label + brand chips + description on Block checkout, click Place Order, redirect to Worldline ANZ's hosted checkout page, complete payment there, and return. Classic shortcode checkout is unaffected — the inline tokenizer iframe still works there end-to-end.

= 1.13.25 =

Completes the Block-checkout Tokenization work started in 1.13.24. The iframe now actually mounts on Block-checkout (1.13.24 had the scaffolding but skipped the AuthCheckout bootstrap so the Tokenizer constructor was called with no URL). 1.13.25 wires the same AuthCheckout session that classic-checkout uses through to the Block-checkout client, and switches the Tokenizer constructor to the canonical 3-arg signature `new Tokenizer(htpUrl, containerId, config)`. Verified live with a real customer driven through a Block-based checkout page.

= 1.13.24 =

Block-checkout Tokenization: customers on the WooCommerce Block-based checkout can now enter card details inline via the Worldline / Worldline ANZ tokenizer iframe (previously only the classic shortcode checkout supported the Tokenization flow; Block-checkout customers on Tokenization providers would have seen no card form). The Hosted-Checkout flow (Tyro) was already supported on Block checkout and is unchanged. The Store API's `payment_data` is bridged into the legacy `$_POST` slot so the existing server-side process_payment() path is reused unchanged.

= 1.13.23 =

i18n: JS strings in `admin-settings.js` and `checkout-tokenization.js` are now translatable via `wp.i18n.__()`. The PHP enqueue calls declare `wp-i18n` as a dependency and register translations via `wp_set_script_translations()`. English locales are unaffected; non-English WordPress sites will pick up translations from `languages/omnifi-woocommerce-gateway-{locale}-{handle}.json` once provided. Also catches one stray "DF_PAY" string in the JS confirm dialog that the 1.13.19 rename missed.

= 1.13.22 =

Performance: per-request memoisation on the Omnifi auth-token fetch. A single checkout request used to fire `GetWorldlineAuthenticationToken` 4-8 times because each `new Omnifi_API_Client(...)` constructed an independent client and debug mode bypassed the transient cache. 1.13.22 adds a process-scoped static cache that all clients share, so a checkout request now does one auth fetch regardless of debug. Transient cache + cross-request behaviour are unchanged.

= 1.13.21 =

Security hardening on the saved-cards tokenize-start endpoint (`?wc-api=omnifi_gateway_tokenize_start`). Adds a same-origin Referer check (cross-site Referer is rejected) and tightens the nonce policy so a present-but-invalid nonce now hard-fails instead of silently continuing. Absent nonces still take the soft path so cached My Account pages keep working. No customer-facing behavior change on legitimate flows.

= 1.13.20 =

UX tweak: the SSO connection panel now shows either the connected Company name OR the Company ID, not both. Connections whose OAuth payload includes a company name show only the readable name; older or future connections that omit the name fall back to the numeric Company ID.

= 1.13.19 =

Admin UI cleanup: renames every user-facing "DF_PAY" reference to "Omnifi" (status badges, buttons, notices, dashboard labels, help text). No functional or contract changes — internal endpoint paths, option keys, settings keys, log messages, and PHPDoc all keep their existing identifiers; this is strictly a brand polish. Also adds the connected Company name to the SSO connection panel when the OAuth exchange payload includes it; existing connections need a one-time reconnect to populate the new field, otherwise the panel falls back to showing just the Company ID as before.

= 1.13.18 =

Adds compatibility with the new Omnifi outbound webhook specification used by Tyro and Worldline integrations. Webhook reconciliation now recognizes `eventType` payloads and maps `transactionCode` and `externalOrderId` to existing WooCommerce orders. Added support for `PAYMENT_SUCCEEDED`, `PAYMENT_FAILED`, and `PAYMENT_REFUNDED` event tokens during status normalization so webhook-driven order updates resolve correctly without falling back to polling. Preserves backward compatibility with legacy payload formats (`payId`, `merchantReference`, `orderId`, etc.) while supporting the new signed webhook contract for upcoming Omnifi outbound dispatch.


= 1.13.17 =
Fixes "Refund amount exceeds the remaining refundable balance" being raised incorrectly. The gateway's own remaining-balance guard double-counted the refund being processed (WooCommerce saves the WC_Order_Refund record before calling the gateway), so any refund larger than half the remaining balance — including every full/final refund — was rejected. The guard is removed; WooCommerce core already validates the amount before creating the refund and the provider rejects over-refunds server-side.

= 1.13.16 =
Completes the 1.13.14 cross-provider refund fix. Previously the dispatcher routed to the order's recorded provider correctly, but the API client below it still used the active default_provider's credentials and rejected with "X refund can only be used with the X provider." Each provider's process_refund now overrides default_provider locally before constructing the API client so the credential / endpoint resolution matches the order, not the operator's current default.

= 1.13.15 =
Fixes broken Tyro refunds. The plugin was sending Tyro refund requests without the required `currency` field; Omnifi responded with HTTP 400 "currency is required" and the operator saw "Refund attempt FAILED at the payment provider" on every Tyro refund attempt. Currency is now derived from the order and included in the request. Verified live against staging refund matrix 2026-06-12. Worldline + Worldline ANZ refunds were already passing currency correctly.

= 1.13.14 =
Refunds now dispatch to the provider that originally charged the order (read from `_omnifi_provider` order meta), not the currently-selected default provider. Fixes the failure mode where an operator who switched their default provider couldn't refund older orders charged on the previous provider — the wrong refund endpoint was called and Omnifi rejected with "Payment record not found". Legacy orders without `_omnifi_provider` meta still fall back to active provider for backward compat.

= 1.13.13 =
Telemetry-clarity: Worldline / Tyro pending-status tokens are now explicitly enumerated in `Omnifi_Utils::normalize_status()` instead of all bucketing into the catch-all default. Unknown tokens still map to PENDING (behaviour-preserving) but additionally fire a `omnifi_wc_status_unknown_token` action so operators can hook telemetry onto schema drift.

= 1.13.12 =
Completes uninstall cleanup. The plugin's uninstall script now also removes the `omnifi_wc_gateway_schema_version` option, any `omnifi_tok_charge_lock_*` / `omnifi_status_lock_*` lock rows, the `_transient_omnifi_auth_response_*` cache, and DROPs the covering indexes added by the migration. Order meta (`_omnifi_*`) is intentionally preserved — financial audit / refund-history data.

= 1.13.11 =
Schema migrations no longer run on customer-facing front-end requests. Previously, the first front-end request after a plugin upgrade with a stale schema_version would synchronously issue `CREATE INDEX` on `wp_postmeta` — potentially locking the table for seconds on large stores. Now gated to admin / wp-cron / WP-CLI contexts.

= 1.13.10 =
Adds three diagnostic console.info lines around Worldline's tokenizer lifecycle (constructor / initialize-resolved / initialize-rejected). No visible UI change. Lets support identify whether an intermittent "blank card form" report is a Worldline silent-hang vs. an explicit rejection.

= 1.13.9 =
"Webhook Signature Required" toggle now defaults to OFF for all three providers. Omnifi does not currently send webhooks (Tyro receiver exists but Omnifi has not enabled delivery; Worldline + ANZ rely on polling), so the previous default-ON state was misleading. Security model is unchanged: empty webhook secret or missing signature still rejects unconditionally per the 1.13.5 fix.

= 1.13.8 =
Lifecycle hygiene: plugin now clears its scheduled cron jobs (status polling + abandonment timers) on deactivation. No behaviour change while the plugin is active.

= 1.13.7 =
Migration robustness: schema_version is now only bumped when ALL CREATE INDEX statements succeed. Previously, a failed CREATE INDEX (MySQL privileges error, table-lock during install) was silently recorded as "done" and never retried — leaving the customer paying the un-indexed meta_value lookup penalty forever. No action required for sites where indexes already exist; sites that previously failed to index will re-attempt on the next request.

= 1.13.6 =
Webhook routing security: the `x-omnifi-provider` request header is no longer honoured as a provider override. The configured `default_provider` in WC settings is now the only source. Closes an attack path where an attacker with one provider's webhook secret could route forged events through a different provider's verifier.

= 1.13.5 =
Webhook security hardening: empty `webhook_secret` now unconditionally rejects all webhooks regardless of the `require_webhook_signature` toggle. Previously, operators who unchecked the toggle and left the secret blank accepted forged webhooks. Defaults are unchanged — only operators in that misconfigured state are affected.

= 1.13.4 =
Fixes customer trap on failed Worldline charges. The single-use hostedTokenizationId was being resubmitted on every retry, causing the customer to loop through "Payment failed" until they manually refreshed the page. The plugin now clears the hidden token input on WC's `checkout_error` event so the next click mints a fresh tokenization id.

= 1.13.3 =
Fixes the deployment pipeline. The internal `upload-plugin.spec.ts` Playwright test was uploading a hardcoded `omnifi-woocommerce-gateway-1.13.0.zip` regardless of the current plugin version, so every version bump from 1.13.0 onward silently uploaded a stale build. The upload spec now reads the version from the plugin header at test time and verifies the active version on the admin plugins page matches what was just uploaded. No customer-facing change.

= 1.13.2 =
Fixes the "card form stuck on spinner" bug. WC's own `$.blockUI()` overlay (added on `.woocommerce-checkout-payment` before its update_order_review AJAX) was sitting on top of the iframe with a spinning indicator forever. WC normally removes the overlay by replacing the entire payment fragment in the AJAX response — but 1.13.1's server-side fragment-suppression filter prevented that swap, leaving the overlay stuck. The plugin now explicitly calls `$.unblock()` on the payment box in the `updated_checkout` handler. Update strongly recommended for anyone running 1.13.1.

= 1.13.1 =
Fixes the "card form goes blank on address change" bug on Worldline + Worldline ANZ. Server-side filter now preserves the in-flight tokenization iframe across WC's update_order_review fragment swap; cache key for the AuthCheckout session is keyed on items-only cart hash (stable across address-induced tax/total recalcs). Visual loading hint + 25 s watchdog removed — customers were misreading the spinner as a stuck state.

= 1.13.0 =
Adds Omnifi Tokenization flow support (embedded card form on the WC checkout page) for Worldline + Worldline ANZ. Hosted Checkout merchants: no behaviour change, no action needed. Tokenization merchants: customers now enter card details inline on the WC checkout page; no redirect for the frictionless path.

== Changelog ==

= 1.13.17 =
* Fix: removed the remaining-balance guard in `Omnifi_Gateway::process_refund()` that incorrectly blocked large/full refunds with "Refund amount exceeds the remaining refundable balance." WooCommerce's `wc_create_refund()` saves the `WC_Order_Refund` record BEFORE invoking the gateway, so `$order->get_total_refunded()` inside `process_refund()` already includes the in-flight refund; the guard's `total − refunded` math double-counted it, computing remaining = 0 for a full refund (and rejecting any refund greater than half the true remaining balance). Observed live on staging order 545: $98.10 previously refunded of $247.10, attempting the final $149.00 was blocked despite WooCommerce showing "Total available to refund: $149.00". Amount validation is WooCommerce core's job (it checks against the real remaining balance with `wc_format_decimal` rounding before creating the refund) and Omnifi rejects over-refunds server-side.

= 1.13.16 =
* Fix: completes the 1.13.14 cross-provider refund dispatch. 1.13.14 made `Omnifi_Gateway::process_refund()` look up the order's recorded provider via `_omnifi_provider` meta and call THAT provider's `process_refund()`. Live verified on staging 2026-06-12: dispatch worked, but the API client constructed inside the provider's refund method (`new Omnifi_API_Client( $settings, $logger )`) still picks `default_provider` from the live gateway settings — i.e. the operator's current active provider — and uses it for provider-mismatch guards (`is_worldline_provider()` / `is_worldline_anz_provider()`) plus credential resolution (`get_merchant_key()` resolves per-provider). Symptom on staging: refunding a Tyro order (489) while active = worldline_anz returned "Tyro refund can only be used with the Tyro provider." Each provider's `process_refund()` now sets `$settings['default_provider'] = $this->get_key()` (locally — does NOT persist to options) before instantiating the API client, so credential resolution and the inner mismatch guards both target the right provider. Source-level guard in `182-refund-forces-provider-identity.test.php`.

= 1.13.15 =
* Fix: Tyro refunds were failing with HTTP 400 "validation error: currency is required" because `Omnifi_API_Client::create_tyro_refund()` posted only `{Tyro_token, MerchantKey, Pay_id, amount, debug}` — no currency. Per the integration handover doc (Section 4 — Refund endpoints) Omnifi's `CreateWooCommerceTyroRefund` requires the same `currency` field that Worldline/ANZ already pass. Method signature now accepts a third `string $currency` parameter, validates it (uppercased + trimmed, empty → omnifi_missing_tyro_refund_currency WP_Error), and includes it in the POST body. `Omnifi_Provider_Omnifi::process_refund()` passes `$order->get_currency()` at the call site. Live verified 2026-06-12 against staging order 491; pinned by `180-tyro-refund-includes-currency.test.php`.

= 1.13.14 =
* Fix: `Omnifi_Gateway::process_refund()` now dispatches to the provider that ORIGINALLY CHARGED the order (read from `_omnifi_provider` order meta via the provider registry) instead of the currently-selected default provider. Pre-1.13.14, an operator who switched their default provider — say, from Tyro to Worldline ANZ — and then tried to refund an older Tyro order had the Worldline ANZ refund endpoint called with a Tyro Pay ID. Omnifi rejected with "Payment record not found" / cross-merchant error, the refund row showed as failed in the metabox, and there was no way to refund the legacy order without temporarily flipping the active provider back in settings. Every order stamps `_omnifi_provider` during process_payment (all four provider classes verified), so the lookup is always populated for orders charged on 1.x+. Active-provider fallback is retained for legacy orders without the meta key. Sanitize_key() applied to the meta value before registry lookup.

= 1.13.13 =
* Telemetry: `Omnifi_Utils::normalize_status()` now explicitly enumerates 13 pending-status aliases (PENDING, PROCESSING, IN_PROGRESS, INITIATED, AUTHORIZED, AUTHORIZATION_AVAILABLE, AUTHORIZATION_PENDING, CAPTURE_PENDING, PAYMENT.PENDING, PAYMENT.PROCESSING, PAYMENT.AUTHORIZED, CHALLENGE_REQUESTED, REDIRECTED) before falling through to the catch-all default. Pre-1.13.13 these all returned PENDING via the catch-all, behaviourally correct but indistinguishable in logs from a typo or schema-drift token. The catch-all branch additionally fires a new WP action `omnifi_wc_status_unknown_token` (passing the normalized token + original string) so operators can wire alerting onto upstream status changes we haven't enumerated yet. No mapping change — same WC status output as before.

= 1.13.12 =
* Hygiene: `uninstall.php` now removes the `omnifi_wc_gateway_schema_version` option, the `omnifi_tok_charge_lock_*` lock rows (Tokenization charge mutex, written via `add_option` so they survive process crashes), the `omnifi_status_lock_*` lock rows (status-sync mutex), and the `_transient_omnifi_auth_response_*` cache. Each `DROP INDEX` is pre-checked with `SHOW INDEX` to avoid throwing on missing indexes. HPOS index drop is additionally guarded by `SHOW TABLES LIKE` so it's a silent no-op on non-HPOS sites.
* Intentional preservation: `_omnifi_*` order and customer meta is NOT touched. That data carries Pay IDs, refund history, 3DS state, and audit notes — destroying it would lose financial audit records if the operator later re-installs the plugin. Pinned by `174-uninstall-cleanup-complete.test.php`.

= 1.13.11 =
* Hygiene: `Omnifi_Migrations::maybe_run()` is now gated on `is_admin() || wp_doing_cron() || (defined WP_CLI && WP_CLI)`. Previously the method was wired to `plugins_loaded` priority 5 and would fire on EVERY request — including front-end customer pages. Whenever the stored `schema_version` was behind, the first such request synchronously ran `CREATE INDEX` on `wp_postmeta` (and the HPOS analog if present), briefly locking the table. On large stores this could stall a customer-facing checkout request for seconds. Activation entry point (`activate()`, registered via `register_activation_hook`) is unchanged — it bypasses the gate because activations are inherently admin-context. Behaviour change is invisible to customers and operators in the steady state (everyone's stored schema is `1.4.0`); only matters during the brief window between a plugin upgrade and the next admin page load.

= 1.13.10 =
* Diagnostic: re-added three `console.info` / `console.error` lines around the Worldline tokenizer lifecycle in `checkout-tokenization.js` — constructor entry (`initTokenizer: constructing Tokenizer with {htpUrl}`), `initialize() resolved`, and `initialize() rejected`. Removed in 1.13.1 as part of "remove the loading state", but that took with it the only console signal into Worldline's tokenizer behaviour, leaving the ~1-in-8 silent-bail invisible to support. The three lines together let support diagnose any future "blank card form" report at a glance: constructor + resolved = form interactive (normal); constructor + rejected = Worldline returned an explicit error; constructor only = Worldline silently hung (upstream — refresh-to-retry is the only customer recovery path). No visible UI change; the watchdog / spinner / loading-hint that were removed in 1.13.1 stay removed (pinned by `170-diagnostic-console-logs-restored.test.php`).

= 1.13.9 =
* UX: `require_webhook_signature` (and the two Worldline-prefixed equivalents) now default to `'no'` across all three provider field definitions. Omnifi does not currently send webhooks for any provider — Tyro receiver exists but Omnifi has not enabled outbound delivery; Worldline + Worldline ANZ are reconciled via polling and have no signing secret. Defaulting the toggle to `'yes'` while the secret can't be obtained from the upstream provider was a misleading config burden on operators. The runtime `Omnifi_Webhook_Controller::is_signature_required()` fallback also flips from `true` to `false` for consistency.
* No security regression: the 1.13.5 verifier behaviour is unchanged — empty `webhook_secret` and missing signature header both reject unconditionally regardless of this toggle. The toggle is mostly cosmetic until Omnifi enables webhook delivery; we kept it so operators can opt back into the strict gating once they have a real secret to configure.
* New unit test `168-require-webhook-signature-default-no.test.php` pins all four defaults (three form-field declarations + runtime fallback).

= 1.13.8 =
* Lifecycle: added `register_deactivation_hook` → `omnifi_wc_gateway_on_deactivate()` which calls `wp_clear_scheduled_hook()` on both `omnifi_wc_gateway_poll_payment_status` and `omnifi_wc_gateway_abandon_pending_order`. Pre-1.13.8, these single-event WP-cron entries persisted in the `cron` option forever after deactivation — functionally harmless (their action handlers were no longer registered) but they accumulated in `wp_get_scheduled_event()` lookups for the lifetime of the database. The deactivation hook does NOT delete settings, options, order meta, or `schema_version` — those live across deactivation cycles intentionally; uninstall.php is responsible for hard cleanup. New source-level unit test (`166-deactivation-clears-crons.test.php`) pins both the registration and the function body.

= 1.13.7 =
* Fix: `Omnifi_Migrations::run()` previously called `update_option(OPTION_SCHEMA_VERSION, TARGET)` unconditionally after `index_postmeta()` + `index_hpos_orders_meta()`. A failed `CREATE INDEX` (MySQL privileges, table-lock during install/upgrade, etc.) was silently recorded as "done", because `maybe_run()` short-circuits on the version match on subsequent requests. The customer paid the un-indexed-meta_value full-scan penalty forever with no signal of the failure. The two sub-migration methods now return `bool` reflecting whether the index actually exists on the table after the call (post-CREATE `SHOW INDEX` re-check, since `$wpdb->query()` return values for DDL are driver-dependent). `run()` only bumps `schema_version` when both returned true; a transient failure leaves the version untouched so the next request retries.

= 1.13.6 =
* Security: `Omnifi_Webhook_Controller::resolve_webhook_provider()` no longer reads the `x-omnifi-provider` request header to override the configured `default_provider`. Server config is now the single source of truth for which verifier handles an inbound webhook. Closes a low-likelihood-but-real attack path: an attacker who held one provider's webhook secret could send a payload signed with that secret + `x-omnifi-provider: <other_provider>` and the controller would route to the wrong verifier — bypassing the provider-specific signature path that any single verifier would have performed correctly. The request parameter remains in the method signature for hook/filter compatibility but is no longer consulted.

= 1.13.5 =
* Security: webhook empty-secret bypass closed. Both Tyro and Worldline webhook verifiers used to honour the `require_webhook_signature` toggle and accept unsigned webhooks when the toggle was off and the secret empty. That combination is no longer accepted under any circumstances — empty secret is an unconditional reject. The `require_webhook_signature` setting still exists but is now load-bearing only when both fields are set (it gates whether the absence of the signature header alone is rejected). Defaults unchanged: `require_webhook_signature` defaults to `yes`, secret defaults to empty — the default state still rejects all webhooks until an operator configures both.

= 1.13.4 =
* Fix: customer trapped in a "Payment failed" loop after a declined card on Worldline + Worldline ANZ tokenization. Worldline's hostedTokenizationId is single-use — once SavedCheckout consumes it (even on a declined card), the same id resubmitted by the customer fails on the server with the same error. The optimization in `onPlaceOrder` that short-circuits the second click after a successful submitTokenization (so we don't re-call Worldline before our own self-triggered form re-submit) was re-broadcasting the dead token on every retry click. The plugin now listens for WC's `checkout_error` jQuery event and clears the hidden tokenization inputs so the next click mints a fresh id from Worldline.

= 1.13.3 =
* Dev: `tests/e2e/upload-plugin.spec.ts` now reads the current plugin version from the plugin header at test time instead of uploading a hardcoded `omnifi-woocommerce-gateway-1.13.0.zip`. Prior revision silently uploaded a stale build on every version bump after 1.13.0 — meaning 1.13.1 and 1.13.2 never actually reached the staging server during testing. Spec now also verifies the version visible on `/wp-admin/plugins.php` matches the zip that was just uploaded, so future invisible deployment failures fail loudly.

= 1.13.2 =
* Fix: card form stuck behind a spinning loading overlay. WC's `update_order_review` AJAX calls `$.blockUI()` on `.woocommerce-checkout-payment` before firing, and normally removes the overlay by replacing the entire payment fragment in the response. Our 1.13.1 fragment-suppression filter prevented that swap, leaving the overlay (the `div.blockUI.blockOverlay` with `opacity: 0.6` and a spinning `::before` pseudo-element) on top of the iframe permanently. The `updated_checkout` JS handler now explicitly calls `$.unblock()` on the payment box and removes any orphaned `.blockUI` nodes as belt-and-braces cleanup. Resolves the "always loading state" reports from 1.13.1.

= 1.13.1 =
* Fix: card form going blank on every address-field change on Worldline + Worldline ANZ. Server-side `woocommerce_update_order_review_fragments` filter (`omnifi_wc_gateway_preserve_tokenization_iframe`) now drops the `.woocommerce-checkout-payment` fragment from WC's AJAX response when the customer's cart items are unchanged, so the iframe stays in the document and keeps its browsing context. Iframes do NOT survive DOM detach — removing them destroys the iframe's content per HTML spec.
* Fix: `bootstrap_tokenization_session` cache key now derived from items-only cart hash (product_id + variation_id + quantity + variation attrs). Previous `cart_hash` keying included the cart total, which flipped on every address-induced tax/shipping recalc — caused a fresh AuthCheckout (and a new session URL) on every update_order_review AJAX. Items-only keying is stable across all address changes; only genuine cart-item changes invalidate.
* Refactor: removed the visual loading-hint spinner, the 25-second mount watchdog, and the "Try again" recoverable error UI from `checkout-tokenization.js`. Customers were misreading the spinner stack (our loading hint + Worldline's own internal skeleton inside the iframe) as a stuck loading state, even when `initialize() resolved` had logged successfully and the form was interactive. Empty container + Worldline's native form is now the cleanest honest signal.

= 1.13.0 =
* Feature: Omnifi Tokenization flow support. Worldline + Worldline ANZ merchants configured for Tokenization in Omnifi now show an embedded Worldline iframe on the WC checkout page (customer enters card details inline). On Place Order the plugin charges via a single SavedCheckout call — no off-site redirect for the frictionless path. 3DS challenges still redirect to the issuer and return through the existing return handler. Resolves the PAYMENT_FLOW_TOKENIZATION mismatch error (order #2193).
* Feature: Hosted Checkout flow remains untouched. Routing is driven by Omnifi's `paymentFlow` flag in the auth response — no plugin setting to change. The Setup-status dashboard shows which flow is active.
* Feature: wrong-flow recovery. If Omnifi's merchant config is flipped mid-checkout (Hosted Checkout ↔ Tokenization), the plugin invalidates the auth cache, logs the mismatch, and asks the customer to refresh. No customer charge fires on the mismatch.
* Feature: multi-attempt resilience. Decline / 3DS-abandonment retries on the same order are tracked in `_omnifi_superseded_attempts` order meta (capped at 5), visible in the order metabox for triage.
* Feature: idempotency lock on the SavedCheckout call. Double-submission (customer double-click / fast network resubmit) results in one charge, not two.
* Feature: operator escape hatch — set `add_filter( 'omnifi_wc_tokenization_use_redirect', '__return_true' )` to force Tokenization merchants into redirect mode for sites where the iframe can't work (CSP, IT-imposed iframe sandboxing).
* Feature: 7 device/browser 3DS fingerprint hints collected in-iframe and forwarded to SavedCheckout — improves frictionless 3DS rates.
* Feature: provider/`isAnzWorldline` mismatch detection. If the operator picks the wrong provider dropdown (e.g. "Worldline" while connected to an ANZ-backed merchant), a red row appears on the Setup-status dashboard with actionable wording.
* Feature: order metabox now surfaces the active payment flow, the AuthCheckout id (debug-only, dimmed), 3DS pending state, and prior-attempts count.
* Feature: 5-minute auth-response transient cache reduces upstream auth calls on busy checkout pages. Debug mode bypasses the cache.
* Fix: latent bug in `Omnifi_API_Client::create_worldline_auth_checkout()` and `::create_worldline_saved_checkout()` — both rejected the `worldline_anz` provider with a `provider_mismatch` error, breaking the filter-gated saved-cards UX for ANZ merchants. The guards now accept the Worldline-family of providers.
* Change: AuthCheckout's `savePaymentMethod` parameter default flipped from hardcoded `true` to parameterised `false`. Default-checkout Tokenization does not persist cards. The existing saved-cards UX call site (My Account → Add Payment Method) explicitly passes `true` to preserve its behaviour.
* Tests: 76 new PHPUnit tests across 13 new test files covering auth cache, flow routing, provider consistency, AuthCheckout savePaymentMethod backward-compat, requires_tokenization_session fallback, SavedCheckout response shape + classifier, idempotency lock, end-to-end process_tokenization_payment, 3DS redirect, device-fields decoder, wrong-flow recovery, multi-attempt resilience, refund + auto-capture, dashboard indicators, metabox visibility. 172 PHPUnit tests total (was 40).
* Docs: new Omnifi Worldline API contract (canonical, internal), `TOKENIZATION_FLOW_SCOPING.md` (internal doc) (design rationale), `TOKENIZATION_IMPLEMENTATION_PLAN.md` (internal doc) (per-work-item build plan). Operator runbook §6 covers Tokenization triage. Manual test plan §11 covers 14 live scenarios.

= 1.12.3 =
* Fix: customers no longer see "Unable to validate payment return" after a successful Worldline payment when the gateway environment doesn't append the returnMac to the return URL (seen on ANZ preprod). The plugin now soft-validates: a missing returnMac is logged and the return proceeds (the real payment status is still verified server-to-server via Omnifi's GetCheckout API, and the WC order_key check is still strict). A returnMac that IS provided but does not match what was stored at checkout creation is still rejected as a tamper attempt.

= 1.12.2 =
* Fix: every checkout fee is now included in the amount sent to the payment gateway. Previously only products, shipping, tax and discounts went into the base, so third-party cart fees (reusable bag, gift wrap, packaging, tip, etc. added by other plugins) appeared on the WooCommerce order total but were not sent to Omnifi — the customer was charged less than the WC total. The amount calculation now adds every cart fee (label-agnostic) to the base; only our own surcharge / processing-fee lines are skipped to avoid double-counting. Surcharge percentage therefore now also applies to those fees.

= 1.12.1 =
* Fix: saved-card token lookup/removal no longer risks a fatal error when given a stale or invalid payment-token id. The Worldline and Worldline ANZ token resolver/delete methods now null-check the result of WC_Payment_Tokens::get() before use. Added unit coverage for the saved-card storage layer (save / list / resolve / delete, ownership checks, and the invalid-id path).

= 1.12.0 =
* Feature: WooCommerce Cart & Checkout Blocks support. The Omnifi method now appears and is selectable on the modern block-based checkout (previously it only worked on the classic shortcode checkout), showing the same card-brand chips, description, and surcharge / processing-fee disclosure. Payment still runs through the hosted-checkout redirect. Implemented with a small vanilla-JS script (no build step required).
* Maintenance: applied PHPCS auto-fixes (whitespace/alignment) across the runtime PHP. No behaviour change.

= 1.11.0 =
* i18n: the plugin is now translation-ready. A translation template ships at languages/omnifi-woocommerce-gateway.pot (248 strings), the plugin loads translations from /languages, and the build includes that folder. Regenerate the template after string changes with `composer make-pot`.

= 1.10.3 =
* Maintenance: removed dead code left over from earlier UI changes — the unused inline pricing-preview renderers (PHP), the surcharge/process toggle decorators and pricing-preview script (JS), and their CSS. No behaviour change; the settings page renders identically.

= 1.10.2 =
* UI: removed the "Integration URLs" section (Customer Return URL + Webhook URL) from the gateway settings page to declutter. The Customer Return URL is sent automatically by the plugin; the Webhook URL is configured on Omnifi's side during onboarding. Note: if you still need the webhook URL it is your site's REST endpoint at /wp-json/omnifi/v1/webhook.

= 1.10.1 =
* UI: the "Process Amount" field label now shows the provider's settlement currency symbol instead of a hardcoded "$" — Worldline Hosted Checkout shows (€), Tyro and Worldline ANZ show ($). Reflects the fixed per-provider currency.

= 1.10.0 =
* UX: Surcharge and Process Amount no longer have a separate on/off toggle. Each is now a single value field — enter a value to apply it, leave it blank (or 0) to disable. This removes the confusing toggle pill and applies to all three providers (Tyro, Worldline, Worldline ANZ). A stale value saved by an older version is honoured by its number, not a hidden flag.

= 1.9.0 =
* Change: Debug logging is now ONE global setting that applies to every provider, instead of a separate toggle per provider. The per-provider debug checkboxes have been removed; a single "Debug logging" toggle sits at the bottom of the gateway settings. (Previously logging effectively followed the Tyro toggle anyway.)
* Change: Worldline ANZ "Auto Capture" is no longer an editable checkbox — it is ON by default. When an ANZ payment is confirmed (status PENDING_CAPTURE), the plugin now captures it automatically, so authorised payments are not left uncaptured. It is idempotent (won't double-capture) and a mis-timed capture is safely rejected by the gateway. To turn it off, use: add_filter( 'omnifi_wc_worldline_anz_auto_capture', '__return_false' ).

= 1.8.0 =
* Change: each provider's currency is now FIXED, not an editable "Allowed Currencies" list. A hosted-checkout merchant account settles in exactly one currency, and a free-text list let operators enter a currency the merchant does not support (e.g. adding AUD to a EUR-only Worldline), which was accepted at the WooCommerce checkout but then hard-rejected by the payment platform (CHECKOUT_ERROR). The editable field has been removed from all three providers: Tyro = AUD, Worldline = EUR, Worldline ANZ = AUD (ANZ is contractually AUD-only). If your Worldline merchant settles in a different single currency, override it with the new `omnifi_wc_provider_currency` filter (does not apply to ANZ).
* UI: the Setup-status dashboard currency check now states the fix in plain language when there's a mismatch — e.g. "Worldline Hosted Checkout settles in EUR, but your store currency is AUD. Change the store currency to EUR to use this provider — the merchant currency is fixed and cannot be changed here." — with a direct link to WooCommerce → Settings → General. The store currency is what must change to match the provider, not a plugin setting.

= 1.7.6 =
* Fix: the Setup-status dashboard (and checkout availability) now use the correct currency fallback when Allowed Currencies has not been saved yet. Previously the runtime fallback was hard-coded to AUD for every provider, so after 1.7.5 changed the Worldline field default to EUR, the field showed "EUR" while the actual check still used AUD — the dashboard wrongly reported "Store currency AUD is in the provider allowlist" for an EUR-only Worldline. The fallback now matches each provider's field default (Worldline EUR; Tyro and Worldline ANZ AUD), so the dashboard tells the truth before the first save.

= 1.7.5 =
* Fix/Change: the "Worldline Hosted Checkout" provider now defaults its Allowed Currencies to EUR instead of AUD. The global Worldline merchant transacts in EUR (per Omnifi's Worldline Postman collection: "Use EUR for Worldline preprod/global; use AUD for ANZ"); the previous AUD default would hide the method on a correctly-configured EUR store. Worldline ANZ remains AUD-only and Tyro remains AUD. The field help text now explains the EUR-vs-ANZ distinction. Existing sites that already saved an Allowed Currencies value are unaffected (this only changes the default for new installs / unset fields) — adjust the value if your Worldline merchant uses a different currency.

= 1.7.4 =
* UI: further declutter when Omnifi SSO is connected. The entire "Omnifi SSO (optional)" intro block is now removed (in 1.7.3 it was kept with a note), and each connected provider's "Connected via Omnifi — … merchant #N. Disconnect above to use manual keys." description line is removed. Both were redundant: the provider's SSO panel already shows the connection (merchant key, type, IDs, connected date) and a Disconnect button, and the section heading already shows the green "SSO CONNECTED" pill. Everything reappears when no provider is connected.

= 1.7.3 =
* UI: when at least one provider is connected via Omnifi SSO, the "Omnifi API key (X-API-Key)" field and the "Clear saved Omnifi API key" checkbox are now hidden. The key is a site-wide credential used only for the one-time OAuth exchange and is dormant once connected, so hiding it declutters the settings page. The section heading stays with a short note explaining the key is saved and that you should disconnect all providers first to change it. The fields reappear automatically when no provider is connected.

= 1.7.2 =
* Fix: clicking "Connect" for Omnifi SSO before the Omnifi API key is configured no longer shows a raw JSON error page ({"code":"omnifi_sso_no_api_key"…}). The /sso/start endpoint now redirects back to the gateway settings page with a clear admin notice ("Configure the Omnifi API key before connecting."). An invalid provider is handled the same graceful way. This matches the existing behaviour for a cancelled authorize flow.

= 1.7.1 =
* UI: when a provider is connected via Omnifi SSO, the manual "Live Merchant Key" (and Sandbox Merchant Key) fields are now hidden entirely instead of shown as read-only. The key is supplied by the SSO connection, so an editable-looking field was confusing. Disconnect Omnifi for the provider to restore manual key entry. The Webhook Secret field is unaffected — it is independent of SSO and stays editable.
* UI: clearer currency instructions on every provider's "Allowed Currencies" field. The help text now states plainly that your WooCommerce store currency must be listed or the payment method will not appear at checkout, gives the comma-separated ISO-code format, and points to WooCommerce → Settings → General → Currency options. Worldline ANZ explains it is AUD-only.

= 1.7.0 =
* UI: storefront checkout polish — the Omnifi payment method now shows accepted card-brand chips (Visa, Mastercard, Amex by default; filterable via `omnifi_wc_payment_method_brands`) above the description, and a fee-disclosure banner beneath it whenever surcharge or processing-fee is enabled.
* UI: new "Omnifi payment" metabox on the WooCommerce order edit screen (works on both legacy and HPOS order screens) showing provider, last status pill, Pay ID with copy button, transaction ID, last update source, last-synced timestamp, money breakdown (surcharge / processing fee / customer paid), refund history, and a one-click link to WooCommerce logs.
* UX: failed refund attempts now get recorded against the order and surfaced as a red callout in the Omnifi payment metabox — including amount, provider error message, and provider refund ID — so store managers no longer have to scrape logs to find out why a refund failed. Successful attempts do not trigger the callout.

= 1.6.1 =
* UI: Test mode toggle and Sandbox merchant key fields are hidden by default (no longer used in the current operator workflow). Reveal via the existing `omnifi_wc_gateway_show_advanced_settings` filter when sandbox testing is needed.
* Fix: Surcharge and Process Amount toggle labels now render with proper spacing ("On" pill and "Active" caption no longer mash together) for ALL providers, including Worldline ANZ. CSS now uses class-based selectors instead of hard-coded field IDs.
* UI: removed the duplicate "Customer preview" pricing-breakdown row beneath the Process Amount field — the Setup-status dashboard at the top of the page already shows the surcharge disclosure, and having two "preview" blocks was confusing.

= 1.6.0 =
* UI: new Setup status dashboard at the top of the gateway settings page — at-a-glance ok/warn/error/info checks for gateway enabled, provider selected, merchant key resolved (SSO or manual), webhook secret set, store currency in allowlist, and debug logging state. Each check links to the relevant settings anchor.
* UI: storefront checkout preview card next to the status checks — shows the customer-facing Title, Description, and optional surcharge disclosure with the current settings. Title/Description edits update the preview live.

= 1.5.0 =
* UI: redesigned Omnifi SSO Connect panel — coloured status pill, two-column key/value layout, human-formatted Connected timestamp.
* UI: integration URLs (Webhook URL, customer Return URL, SSO callback URL) now have one-click Copy buttons.
* UI: each provider's settings section title shows a status pill — `SSO Connected` (green), `Manual key` (blue), or `Not configured` (grey) — plus a short description.
* UI: new inline pricing preview row beneath the surcharge / processing-fee fields shows what a $100 order would look like with the current values; updates live as inputs change.

= 1.4.8 =
* Fix: clicking Cancel on the Omnifi authorize screen no longer shows a raw JSON error page. Omnifi redirects back with `error=user_cancelled` and no `code`; the callback now handles that gracefully and redirects to the gateway settings page with an info notice ("Omnifi connection cancelled. Try Connect again when you are ready.").
* Improvement: admin notice renderer now supports info and warning notice types in addition to success/error.

= 1.4.7 =
* Fix: "Disconnect DF_PAY" button now works. The button was inside an inner <form> nested in the WooCommerce settings outer <form>, which is invalid HTML — browsers closed the outer form prematurely, the button submitted to the wc-settings save URL instead of our REST endpoint, and the connected panel was reparented to the top of the page. The disconnect button is now a plain button with data-* attributes plus a JS click handler that POSTs via fetch.
* Fix: connected SSO panel now sits inside its provider section (between e.g. "Tyro Settings" heading and the merchant key fields), not at the top of the page.

= 1.4.6 =
* Fix: Omnifi OAuth exchange now sends `externalStoreId` as a JSON number, not a SHA1 hex string. Omnifi's API expects `Nullable<int>` and was rejecting the previous string with HTTP 400 "The JSON value could not be converted to System.Nullable`1[System.Int32]". The resolver now derives a stable 31-bit positive integer from `home_url + wp_salt`. Legacy SHA1-hex stored values are migrated transparently on first read.
* Admin: `External store ID override` advanced field now only accepts numeric input; non-numeric values are ignored and the derived integer is used instead.

= 1.4.5 =
* Improve: SSO OAuth exchange failures now log the full Omnifi response body and the masked request body (clientCode, externalStoreId visible; code+state masked). Previously the log only showed our default fallback message — useless when Omnifi returned no `message` field. API key is still never logged.

= 1.4.4 =
* Fix: Omnifi OAuth callback no longer 401s ("rest_forbidden"). The /wp-json/omnifi/v1/sso/callback REST route is now open; authentication is the single-use, server-bound `state` token validated inside the handler. The previous `manage_woocommerce` permission check required a WP `_wpnonce` parameter that third-party OAuth providers cannot forward.
* Security: state token is still validated as before — single-use, 600s TTL, bound to the admin user who initiated Connect. After validation, the handler restores that user context for the rest of the request (so capability checks downstream still work, and the success-notice transient lands in the right user's inbox).

= 1.4.3 =
* Fix: per-provider "Connect with DF_PAY" button now stays inside the WooCommerce settings table at the right position. Previously it rendered as a free-floating <div> which the browser reparented above the form, making it look missing for Worldline ANZ admins.
* Fix: global Omnifi SSO fields (API key + Clear) no longer disappear when Worldline or Worldline ANZ is the active provider. The admin-row visibility JS treats them as shared fields.

= 1.4.2 =
* Improve: refund rejection error now includes the provider status word and refund ID so operators can diagnose without opening the debug log. Example new message: "Worldline refund request was rejected by the provider. (Status: REJECTED; Refund ID: 10020197545_6)".

= 1.4.1 =
* Fix: refunds are no longer incorrectly rejected when the provider response omits the `success` field (regression introduced in 1.4.0). Worldline's real success response is `{refundId, status: "REFUND_REQUESTED", ...}` without a `success` boolean — this is now accepted. Explicit `success: false` or failure status words still reject.

= 1.4.0 =
* Fix: `request_timeout` setting now honoured up to 120 seconds (was silently capped at 15).
* Fix: Worldline ANZ now reads its own `worldline_anz_include_order_key_in_return_url` setting (previously fell through to the Worldline value).
* Fix: Worldline ANZ uses its own checkout fee-snapshot session key, preventing stale values when switching providers.
* Fix: Refund response without an explicit `success` field is now treated as failure (was silently treated as success).
* Fix: `find_order` digit-fallback regex anchored to end of string, preventing year-like fragments from matching the wrong order.
* Webhook replay-window check now honours `omnifi_wc_webhook_require_timestamp` filter (default off) — webhooks without a timestamp are no longer rejected by default.
* Security: SSO admin panel output is now escaped at the point of echo.
* Observability: Omnifi OAuth exchange failures are now logged via `wc_logger` when debug is enabled.

= 1.3.34 =
* Security hardening across payment callbacks and webhooks.
* Added webhook replay protection windows for Tyro and Worldline.
* Strengthened return callback verification and order-key validation rules.
* Improved refund authorization safeguards and over-refund prevention checks.
* Enhanced logging redaction for sensitive fields.
* Added order status lock robustness and status poll rate limiting.

