---
name: skill-microsoft-store-publishing
description: >-
  Publish a Windows app (MSIX) to the Microsoft Store from CI, including the
  full first-time setup: Partner Center account, the Microsoft Entra tenant
  association gotcha, the Azure AD app + Manager role for the submission API,
  the msstore CLI on Linux runners (libsecret/dbus/keyring), MSIX identity and
  version rules, and why the FIRST submission must be manual while updates
  automate. Load this when setting up or debugging Microsoft Store / Partner
  Center / Entra / msstore-cli / Windows code-signing publishing.
metadata:
  type: reference
  source: relux-works (hard-won during the Pulsar Barycenter launch, 2026-07)
---

# Microsoft Store publishing from CI

Everything you need to take a Windows MSIX from a GitHub repo to the
Microsoft Store — and the traps that cost a full evening the first time.
Read the reference that matches your step; each is short and battle-tested.

## The 30-second map

1. **Partner Center account** — sign up, verify. Individual is fine; the
   publisher display name can still be your company. → `references/01-partner-center-account.md`
2. **Associate an Entra tenant** — Partner Center starts with NONE. This is
   the #1 time sink. Associate with the tenant's **`.onmicrosoft.com` global
   admin UPN**, not your custom-domain email. → `references/02-entra-tenant-association.md`
3. **Azure AD app for the API** — register it, make a secret, link it in
   Partner Center as **Manager(Windows)**, grab the Seller ID. → `references/03-azure-ad-app.md`
4. **msstore CLI in CI** — self-contained binary (not a NuGet tool); on Linux
   it needs libsecret + a dbus/gnome-keyring secret service. → `references/04-msstore-ci.md`
5. **MSIX identity & version** — must match the Store-assigned identity
   exactly; version revision must be 0. → `references/05-msix-identity-versioning.md`
6. **First submission vs updates** — the first submission MUST be manual (a
   CLI bootstrap chicken-egg); updates automate. → `references/06-first-submission-vs-updates.md`
7. **Windows GUI build (Go)** — `-H windowsgui`, file logging, Segoe UI. →
   `references/07-windows-gui-build.md`
8. **Signing options** — Store signs for free; Azure Trusted Signing excludes
   many countries; EV certs are expensive. → `references/08-signing-options.md`

## The five traps that will actually get you

1. **No Entra tenant is associated by default.** Account settings → Tenants is
   empty. Without it, "Sign in with Microsoft Entra ID" in User management
   loops forever and you cannot add the Azure AD app. Associate the tenant
   first (§02).

2. **The association login is NOT your email.** Entra wants the tenant's
   global-admin UPN on `*.onmicrosoft.com` (e.g. `admin@contoso.onmicrosoft.com`),
   not `admin@contoso.com`. "contoso.com isn't in our system" = your custom
   domain isn't verified in Entra; use the `.onmicrosoft.com` UPN (find it in
   entra.microsoft.com → Users) (§02).

3. **msstore is not a NuGet dotnet tool.** `dotnet tool install MSStore.CLI`
   fails ("not found in NuGet feeds"). Download the self-contained binary from
   github.com/microsoft/msstore-cli releases. On Linux CI it then needs
   `libsecret-1-0` + `gnome-keyring` + `dbus-x11`, run inside `dbus-run-session`
   with an unlocked keyring (§04).

4. **MSIX Version revision must be 0.** `0.3.0.15` is rejected
   ("revision number other than zero"); use `0.3.15.0`. And Name/Publisher
   must match the Store-assigned identity byte-for-byte (§05).

5. **`msstore publish` needs `--appId <ProductId>` in CI.** Without it the
   CLI looks for a LOCAL "initialized project" (`msstore init` state) and
   dies with "Failed to find the AppId" — it does NOT resolve the app from
   the MSIX package identity, and the alias is `-id` (single dash; `--id`
   does not exist). The FIRST submission still belongs in the Partner Center
   UI — certification requires listing + ≥1 screenshot + age ratings +
   pricing, which publish does not create (§06). Every update after is fully
   automated; a pending submission is auto-superseded once the app has a
   published version (§06). Bonus: cert runs your app on a real Windows VM —
   a free smoke test for a blind-built Windows app.

6. **Partner Center API 504s are routine.** The DevCenter v1.0 API times out
   through Azure Front Door (~80 s waits) on GETs, submission creation and
   the post-commit status polls. Retry the run; and treat "publish exited
   non-zero AFTER 'Submission Committed'" as a status-poll casualty, not a
   failed submission — verify with `msstore submission status <ProductId>`
   (§04, and the hardened `assets/store-submit.yml`).

## Assets

- `assets/store-auth-check.yml` — verify API credentials (`msstore apps list`).
- `assets/store-submit.yml` — push an MSIX update to the Store from CI.
- `assets/AppxManifest.xml.in` — MSIX manifest template with the identity/
  version placeholders and the appContainer trust level.

Every command in the references is one that actually ran and worked.
