# Bookaroo Security Patch — Deployment and Tests

## Status
Prepared from supplied source code and SQL metadata. **Not deployed or live-tested.** Stripe must remain in test mode until all checks pass.

## Important compatibility notes
- The SQL patch changes `bookaroo_release_checkout` from `(uuid)` to `(uuid,text)`. Deploy the matching webhook in the same maintenance window.
- Do not apply to a database with active checkout sessions unless you can reconcile them: an event for a session whose `stripe_session_id` was never stored will be rejected for manual reconciliation.
- Stripe test events for `async_payment_failed` must be checked against real test-mode session data. Some failed payment events may have differing amount/currency fields; adapt only after verifying event semantics.
- The existing `create-checkout` function still needs robust recovery if Stripe creates a session but saving its ID fails. This patch intentionally fails closed and logs mismatches rather than fabricating payment success.
- Confirm whether `seller_reviews` UPDATE column grants affect your existing client. This patch allows updating `rating` and `comment` only.
- The new demo trigger allows service-role imports but disallows users creating demo listings. Review existing demo workflows before deploying.
- Stripe Checkout expiry is 30 minutes while database lock is 35 minutes. A late webhook can still require manual reconciliation. Do not automatically fulfill/refund from this patch.

## Before deployment
1. Back up the database and save current definitions of all changed functions/policies.
2. Check for active orders: `select id,status,stripe_session_id,created_at from public.orders where status='pending';`
3. Verify Stripe webhook endpoint uses `stripe-webhook` and that `STRIPE_WEBHOOK_SECRET` matches the endpoint.
4. Schedule a quiet window with no active checkouts.
5. Deploy `security_patch.sql` and `stripe-webhook_index.ts` together; check Edge Function logs.

## Test in Stripe test mode
1. Confirm buyer cannot purchase own listing; unverified/non-TTU users rejected.
2. Confirm two buyers cannot purchase the same listing concurrently.
3. Complete a test payment; verify one order paid, listing sold, exactly one notification per party.
4. Replay the same webhook; ensure no duplicate sale or notifications.
5. Send an expired session event; verify only the matching pending order is released.
6. Send a mismatched session/order event; ensure no order or listing is changed and the error is logged.
7. Verify an authenticated user cannot invoke any of the three privileged RPCs directly.
8. Attempt seller-side status/checkout timestamp updates and demo flag changes; confirm rejection.
9. Verify a review cannot target a seller unrelated to its paid, received order.
10. Verify a buyer can update only their own review rating/comment, not buyer_id/seller_id/order_id.
11. Check payment amount, currency, destination account and webhook signature in Stripe test logs.
12. Reconcile any order left pending with a missing `stripe_session_id` before ending testing.

## Rollback
Restore the previous function definitions and review policies from your pre-deployment backup; redeploy the previous webhook *together*. Do not simply roll back SQL after a new payment has been processed: reconcile affected orders against Stripe first. The patch is transactional at SQL level, but Edge Function deployment is separate.
