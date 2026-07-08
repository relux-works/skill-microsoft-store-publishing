# Windows code-signing options

The goal: a Windows app users can download and run **without a SmartScreen
"unrecognized app" warning**. Three routes, cheapest/best first.

## 1. Microsoft Store — free, best UX (preferred)

Submit an MSIX; **Microsoft signs the package itself** with the Store
certificate. Users install with one click, no SmartScreen, and get automatic
updates from the Store. **No code-signing certificate needed at all.** Broad
country support (see `01`). This is the recommended path for almost everyone.

## 2. Azure Trusted / Artifact Signing — cheap but region-locked

Cloud code signing at ~$10/mo, integrates with CI. **But** it is available
**only to organizations in the USA, Canada, EU & UK** (individual signing:
US/Canada only). The Azure portal states this on the create-account blade. An
org outside those regions (e.g. **Armenia**) is **ineligible** — don't waste
the identity-validation attempt. If your org qualifies and you need a
direct-download channel, this is the cheapest signed route.

## 3. EV code-signing certificate — expensive, works anywhere

For a signed direct download when the Store isn't an option and Azure signing
excludes your region:

- **EV (Extended Validation)**, not OV. EV gives **instant SmartScreen trust**
  from the first download; OV shows a warning until reputation accrues.
- Since 2023, EV keys must live on a **FIPS HSM** — for CI use a **cloud
  signing** service (no physical token in the runner):
  - **SSL.com eSigner**: EV cert ~$349/yr **plus** an eSigner cloud
    subscription (~$100/mo, i.e. ~$1200/yr) — the cloud subscription is the
    sting. Has a ready GitHub Action.
  - **Certum SimplySign EV**: ~$300–400/yr all-in (cloud signing included) —
    much cheaper; CI integration is rougher.
- The MSIX/exe `Publisher` (or signtool subject) must match the cert's exact
  subject DN.

## Decision

Try the **Store** first (free, best UX, broad availability). Only if the Store
truly can't serve your case do you pay for EV — and then Certum beats SSL.com
on price for CI signing.
