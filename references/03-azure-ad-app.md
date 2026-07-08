# Azure AD app for the submission API

The msstore CLI authenticates as an **Azure AD application** (service
principal) that you register in Entra and then **link into Partner Center**
with a role. Prerequisite: the Entra tenant is associated (see `02`).

## 1. Register the app (Entra)

[entra.microsoft.com](https://entra.microsoft.com) → **App registrations** →
**New registration**:
- Name: e.g. `pulsar-store-ci`.
- Supported account types: **Accounts in this organizational directory only**
  (single tenant).
- Redirect URI: leave blank. → Register.

From the app's Overview, copy (not secret — safe to share):
- **Application (client) ID**
- **Directory (tenant) ID**

## 2. Client secret

App → **Certificates & secrets** → **New client secret** → 24 months → copy
the **Value** (shown once). This **is** a secret:

```bash
gh secret set MSSTORE_CLIENT_SECRET --repo <owner/repo>   # paste the value
```

Never paste a client secret into chat, logs, or the repo. Tenant ID and
client ID are not secrets but are conveniently kept as CI secrets too:

```bash
gh secret set MSSTORE_TENANT_ID --repo <owner/repo> --body "<tenant-guid>"
gh secret set MSSTORE_CLIENT_ID --repo <owner/repo> --body "<client-guid>"
gh variable set STORE_SELLER_ID --repo <owner/repo> --body "<seller-id>"
```

## 3. Link the app in Partner Center (the step that grants API rights)

Partner Center → Account settings → **User management** →
**Microsoft Entra applications** tab → **Add Microsoft Entra application** →
**Add** (select existing, not Create) → pick `pulsar-store-ci` → role
**Manager(Windows)** → Save.

- Role: **Manager(Windows)** guarantees full submission access without hitting
  a permissions gap. `Developer` also covers "upload packages and submit apps"
  and is the least-privilege choice; pick Manager if you'd rather not debug a
  403 blind. Do **not** pick Manager(**Desktop**) — that's the separate
  desktop-app analytics program, not Store submission.
- Both the app and its **Service Principal** rows should show the role.

## 4. Verify

Run `assets/store-auth-check.yml` (msstore reconfigure + `apps list`). Seeing
your reserved product printed = the link and role are correct.
