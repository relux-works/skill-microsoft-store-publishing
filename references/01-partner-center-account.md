# Partner Center (Microsoft Store) account

The Microsoft Store developer account lives in **Partner Center**
(partner.microsoft.com), under the **Windows** program.

## Sign up

Account settings → Programs → **Other programs** → **Windows** card
("Register as an app developer to submit apps and games") → Get started.

- **Account type: Individual vs Company.** Individual is enough to publish
  MSIX apps (Microsoft signs them, no SmartScreen). The **publisher display
  name** shown to users can still be your company (e.g. "Relux Works, LLC")
  even on an Individual account. You **cannot** cleanly switch Individual →
  Company later — it's effectively a re-registration — so pick deliberately.
- Fee: ~$99 one-time for Company, less/waived for some account types.

## Country availability — much broader than Azure signing

The Store developer account is available in **far more countries** than Azure
Trusted/Artifact Signing. Armenia, for example, is accepted for the Store
account (verified live) but is **excluded** from Azure Artifact Signing
(US/Canada/EU/UK only). If your org is outside the Azure-signing regions, the
Store is your free-signing path (see `08-signing-options.md`).

## Verification gate

Even after onboarding shows "ready", a separate **publisher verification**
(email + business + employment) must complete before you can submit. A red
banner — "You can't submit to the Store until your account is verified" —
blocks submission meanwhile. It can take **up to 5 business days**. Nothing
speeds it up; start it early.

## Key identifiers you'll need later

Account settings → **Legal info**:
- **Seller ID** (an 8-digit number) — needed by the msstore CLI.
- Publisher display name, DUNS, Windows publisher ID.
