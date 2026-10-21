# Security audit — evidence and outstanding checks

**Audit status: PARTIAL / NOT CERTIFIED**  
**Date:** 2026-10-21  
**Reviewed:** Local frontend source and selected SQL migrations shared in the project.  
**Not reviewed:** Live Supabase configuration, complete schema/RLS, deployed Edge Function code, Stripe dashboard/webhooks, logs, or real payment execution.

## Evidence available

- Frontend uses a Supabase **publishable** key and Auth bearer token for Edge Function calls.
- Frontend calls `create-checkout` with `bookId` and `stripe-connect` with an empty payload.
- Frontend checks that returned redirect URLs start with the expected Stripe-hosted domains.
- Frontend includes demo-listing and availability checks, but browser checks **are not** server-side controls.
- Supplied messaging SQL includes participant-only RLS and a server-side conversation-creation RPC. This is source review, **not confirmation of deployment**.
- Supplied admin SQL uses RPC-based moderation and role checks. Schema compatibility and production permissions remain unverified.

## Findings and risks

| ID | Severity | Finding / gap | Required action | Status |
| --- | --- | --- | --- | --- |
| SEC-01 | Critical | Payment session creation function not supplied | Review JWT validation, trusted database price, listing availability, currency, seller Stripe account, metadata | Open |
| SEC-02 | Critical | Stripe webhook implementation not supplied | Verify signature with raw body; map Stripe event to correct order; idempotent transaction | Open |
| SEC-03 | Critical | Order-table RLS and RPC permissions not audited | Ensure browser users cannot set `status='paid'`, change amount or payout destination | Open |
| SEC-04 | High | Concurrent purchase behavior untested | Test double checkout and reservation/fulfillment races | Open |
| SEC-05 | High | Refund/dispute and payment-failure behavior untested | Define reconciliation and safe rollback/retention rules | Open |
| SEC-06 | High | Admin role and report permissions not tested against live database | Test owner, moderator, ordinary user and unauthenticated callers | Open |
| SEC-07 | High | Full schema and storage policies unavailable | Audit each table, bucket, RPC, SECURITY DEFINER search_path and grants | Open |
| SEC-08 | Medium | Messaging privacy/rate limits unverified live | Verify participant RLS, anti-spam and blocked-user strategy | Open |
| SEC-09 | Medium | Browser-only checks are bypassable | Confirm matching backend validation for email verification, demo and sold books | Open |
| SEC-10 | Medium | External scripts/API responses require review | Pin dependencies where practical and ensure untrusted text is rendered safely | Open |

**No security issue is marked 'passed' without a recorded test result.**

## Test environment requirements

- Supabase staging project or a safe isolated test dataset.
- Stripe **test mode**, test Connect accounts, and Stripe CLI or Dashboard test webhooks.
- Test users: verified buyer, verified seller, ordinary third-party user, moderator, owner, and unverified user.
- Browser devtools, network inspection, Supabase SQL Editor and Edge Function logs.
- Do not use real cards, actual payouts, production credentials, or real student data.

## Authentication and authorization test matrix

| Test | Procedure | Pass criteria | Result |
| --- | --- | --- | --- |
| AUTH-01 | Sign up and verify a test `@ttu.edu` address | Account session works after verification | Not run |
| AUTH-02 | Try restricted actions with unverified/non-TTU account | Rejected by **database/backend**, not merely hidden in UI | Not run |
| AUTH-03 | Sign out; replay a stale session/token | Protected endpoints reject invalid/expired token | Not run |
| RLS-01 | As user A, select/update user B's profile or private data | Unauthorized read/write rejected | Not run |
| RLS-02 | As third party, query a private conversation and its messages | No rows disclosed and inserts rejected | Not run |
| RLS-03 | As ordinary user, call admin RPCs directly | Permission denied | Not run |
| RLS-04 | As moderator, attempt owner-only admin grant | Permission denied | Not run |
| RLS-05 | As buyer, attempt to update `orders.status`, `amount_cents`, `seller_id` | Permission denied | Not run |
| RLS-06 | Attempt to upload/read another user's private storage files | Rejected according to bucket policy | Not run |

## Stripe test matrix

| Test | Procedure | Pass criteria | Result |
| --- | --- | --- | --- |
| PAY-01 | Purchase available test listing with Stripe test card `4242 4242 4242 4242` | Correct amount/currency, buyer, seller and order | Not run |
| PAY-02 | Modify price or seller ID in browser request | Server ignores forged values; Stripe price remains trusted | Not run |
| PAY-03 | Attempt purchase of demo/sold/own listing | Backend rejects before session creation | Not run |
| PAY-04 | Decline card using Stripe's documented test card | No paid order; listing state recovers safely | Not run |
| PAY-05 | Abandon checkout | No paid order; reservation eventually releases | Not run |
| PAY-06 | Send forged webhook with invalid signature | HTTP rejection; no order update | Not run |
| PAY-07 | Replay a legitimate webhook twice | One fulfillment/order transition; no duplicate side effects | Not run |
| PAY-08 | Attempt two simultaneous checkouts for same book | At most one completed sale; no double fulfillment | Not run |
| PAY-09 | Use wrong seller Connect destination | Backend refuses mismatch | Not run |
| PAY-10 | Refund/dispute in Stripe test mode | Database state and user-facing status reconcile | Not run |
| PAY-11 | Compare Stripe Dashboard session, payment and Supabase order | Amounts, IDs, statuses and timestamps agree | Not run |

## Review checklist for Edge Function code

1. Reject missing/invalid auth tokens and verify the user using trusted Supabase Auth context.
2. Read `bookId` only as a lookup identifier; query listing price/status/seller server-side.
3. Validate amount in integer cents and currency explicitly; prevent negative/zero/unexpected prices.
4. Verify Stripe Connect account belongs to the seller and is eligible for charges/transfers.
5. Reserve or lock the listing atomically before issuing a checkout session; define expiry behavior.
6. Associate checkout session and payment intent with a unique order ID in trusted metadata.
7. Verify webhook signature against raw request bytes using Stripe's SDK.
8. Process only appropriate successful events, with amount, currency and order-ID checks.
9. Make webhook processing idempotent with unique event/payment identifiers.
10. Reject client-originated attempts to mark orders paid or change payout destinations.
11. Protect secrets in environment variables; never log card details or full auth tokens.
12. Implement test-mode refund, dispute, failure and cancellation reconciliation.

## Recording results

For each test, save: date, environment, tester, test account role, expected result, actual result, relevant redacted request/response, and a Stripe/Supabase record identifier **without secrets**. Change a row from `Not run` to `Pass` only after reproducing the expected result.

**Launch gate:** Do not switch to live Stripe keys until critical/high findings are resolved and all payment tests pass.
