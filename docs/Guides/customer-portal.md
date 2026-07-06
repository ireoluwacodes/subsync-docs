---
title: Customer Portal
deprecated: false
hidden: false
metadata:
  robots: index
---
# Customer Portal

Tokenized self-service links let customers manage billing without logging into your app or the SubSync dashboard. The portal is **hosted HTML** served by SubSync at `{[base_url]}/portal/{token}`.

## Issue a portal token (merchant API)

```
POST {[base_url]}/api/v1/portal/token
Authorization: Bearer ssk_... or JWT
```

```json
{
  "subscription_id": "<uuid>",
  "expires_in_hours": 72
}
```

**Response:** portal URL or token.

```
{[base_url]}/portal/{token}
```

Post-checkout emails include this link automatically. Customers can optionally set up **direct debit** as a renewal fallback or **add a card** for automatic billing.

## Public portal pages

No SubSync auth — the token is the credential.

| Method | Path | Action |
|--------|------|--------|
| `GET` | `{[base_url]}/portal/:token` | HTML billing home — plan, status, actions |
| `GET` | `{[base_url]}/portal/:token/add-card` | Redirect to Nomba card checkout (₦100 verification) |
| `GET` | `{[base_url]}/portal/:token/direct-debit` | Direct debit setup form |
| `POST` | `{[base_url]}/portal/:token/direct-debit` | Submit bank details → create mandate |
| `GET` | `{[base_url]}/portal/:token/direct-debit/pending` | NIBSS validation instructions (auto-refresh) |
| `POST` | `{[base_url]}/portal/:token/cancel` | Cancel subscription |
| `POST` | `{[base_url]}/portal/:token/update-payment-method` | JSON API — returns Nomba checkout URL |

Send `Accept: application/json` on `GET /portal/:token` for the JSON summary (merchant tooling).

## Dual payment method model

| Method | Stored on | Used for |
|--------|-----------|----------|
| Card | `subscription.payment_method_id` | Primary renewal charges |
| Direct debit mandate | `subscription.fallback_payment_method_id` | Dunning fallback + renewal when no card |

Card and mandate coexist. Adding a card does not remove a mandate fallback.

Direct debit mandates become chargeable only after Nomba reports `Active` + `Advice sent` (customer completes NIBSS validation).

## When customers use the portal

- **Transfer signup** — add card or set up direct debit before renewal
- **Optional fallback** — card subscribers can add direct debit as backup
- **Past due** — update card or check mandate status
- **Cancel** — self-service cancellation

## Dashboard pattern

On subscription detail page:

1. **Send portal link** → `POST {[base_url]}/api/v1/portal/token` → email or copy link
2. Customer opens link in browser — no separate frontend required

## Security notes

- Tokens expire (`expires_in_hours`)
- Treat portal URLs like password reset links — time-limited, HTTPS only

## Related

- [Card capture](/docs/card-capture)
- [Invoices & dunning](/docs/invoices)
- [Subscriptions](/docs/subscriptions)
