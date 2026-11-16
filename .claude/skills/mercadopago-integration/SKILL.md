---
name: mercadopago-integration
description: Mercado Pago integration for Turnos App - OAuth connection of the professional's account, Checkout Pro preferences, webhook validation and processing, payment reconciliation, refunds and the professional's subscription. Use when designing, implementing, testing or reviewing any code that talks to Mercado Pago.
when_to_use: Use when the task mentions "Mercado Pago", "MP", "pago", "cobro", "checkout", "preferencia", "webhook", "x-signature", "OAuth", "reembolso", "suscripción", "preapproval", "marketplace_fee", or a card labelled Pagos in TAREAS.md.
---

# Mercado Pago Integration

**Verify before you code.** Mercado Pago changes its API, its documentation URLs and its fees. Before implementing or changing an integration, read the current official documentation for Argentina (mercadopago.com.ar/developers) and the release notes of the `mercadopago-sdk` version pinned in the project. Treat the details below as the design this project chose, and confirm field names and limits against the documentation.

## Design decisions

- **The money never passes through the platform.** Each professional connects their own Mercado Pago account with OAuth, and payment preferences are created with that professional's access token. The payment is credited directly to them.
- **Never ask a professional to paste an access token.** The only way in is the OAuth authorization code flow.
- **The platform fee** is the `marketplace_fee` of the preference. It is configurable by the admin and is 0 by default.
- **Mercado Pago is behind a port.** `Application` defines the payment gateway interface in the project's own terms; the SDK and its types stay in `Infrastructure`.
- **A webhook is a hint, not the truth.** After a notification, fetch the payment from the API and act on what the API returns.

## OAuth (connect the professional's account)

1. "Connect Mercado Pago" redirects to the authorization URL with our application id, the redirect URI and a `state` value.
2. `state` is random, single-use, bound to the signed-in professional and checked on return. It prevents another account from being linked by a forged callback.
3. The callback exchanges the code for an access token and a refresh token.
4. Store both tokens **encrypted**, with their expiry and the Mercado Pago user id. Never log them or send them to the browser.
5. Refresh before expiry with a scheduled job. The refresh token lasts about six months: if a refresh fails, mark the connection as broken, notify the professional and stop offering Mercado Pago for their offerings.
6. Disconnecting deletes the stored tokens.

## Checkout Pro (pay a booking)

- Create the preference with the professional's token when the client chooses Mercado Pago.
- `external_reference` is the booking id. It is how a payment is matched to a booking.
- Set `back_urls` for success, pending and failure, and the notification URL of our webhook.
- The preference expires when the payment deadline of the booking does.
- Send an idempotency key when creating a preference or a refund, so a retry does not create a duplicate.
- The pages the client returns to only display status. They never confirm a booking: the return URL can be opened by anyone.

## Webhooks

The endpoint is public, so it trusts nothing until the signature is valid.

1. **Validate the signature.** The `x-signature` header carries `ts` and `v1`. Build the manifest `id:<data.id>;request-id:<x-request-id>;ts:<ts>;` from the `data.id` query parameter, the `x-request-id` header and `ts`, omitting a pair whose value is absent. Compute HMAC-SHA256 of the manifest with the application's webhook secret, hex-encode it, and compare it with `v1` in constant time. Answer 401 when it does not match. Confirm the exact manifest rules in the current documentation.
2. **Answer fast.** Mercado Pago expects a 200 or 201 within about 22 seconds and retries otherwise. Store the notification, enqueue a job and return.
3. **Process in the job.** Fetch the payment by id from the API with the token of the professional it belongs to, find the booking by `external_reference`, check that the amount and currency match what the booking expects, and apply the transition.
4. **Be idempotent.** The same notification can arrive several times and out of order. Record the processed payment id and status; applying the same one again changes nothing.
5. **Handle every status**: approved, pending or in process, rejected, cancelled, refunded and charged back. Each maps to a transition of the payment and the booking, or to no change.
6. **Late payments.** A payment approved after the booking expired or was cancelled must not silently confirm it. The state machine decides: re-confirm if the slot is still free, otherwise flag it for a refund and notify both parties.

Never log the full notification body, the signature or a token.

## Reconciliation

A scheduled job reviews bookings with a pending Mercado Pago payment and queries the API for each, in case a webhook was lost. It runs the same processing code as the webhook job.

## Refunds

- Total or partial refund through the API, with the professional's token and an idempotency key.
- Record who requested it, why, the amount and the result. The refund status arrives by webhook like any other change.
- A transfer payment is refunded outside the platform; the system only records it.

## Subscription of the professional

- This is a different flow: the professional pays the platform, so it uses the **platform's** credentials.
- One subscription plan in Mercado Pago (`preapproval_plan`) per plan defined by the admin, with its price, period and free trial.
- The webhook topics `subscription_preapproval` and `subscription_authorized_payment` drive the subscription state: active, payment failed, cancelled.
- Access rules (hidden profile, no new bookings, grace period) are decided by the Subscriptions domain from that state, not inside the webhook handler.

## Testing

- Use only Mercado Pago **test users** (a buyer and a seller) and **test cards**. Never a real account or a real card, in any environment before production.
- Test credentials come from secrets or environment variables. Production credentials are entered by the user and are never visible to tests.
- Unit-test the signature validator with fixed inputs, including a wrong secret, a missing header and a tampered id.
- Test the processing job against a fake payment gateway for each status, for duplicates, for out-of-order delivery and for an amount that does not match.
- Cover approved, rejected and pending payments end to end in the sandbox (card E7-08).

## Review checklist

- [ ] No access token, refresh token or secret in code, logs, URLs or pages.
- [ ] The webhook rejects an invalid signature before reading anything else.
- [ ] The booking is confirmed only from the payment fetched from the API, with amount and currency checked.
- [ ] Processing is idempotent and tolerates out-of-order notifications.
- [ ] No call to Mercado Pago inside a database transaction.
- [ ] SDK types do not leave `Infrastructure`.
- [ ] Every status and the late-payment case have a test.
