# Entra tenant association — the #1 time sink

To use the Microsoft Store **submission API** (msstore CLI, StoreBroker) you
must add an **Azure AD application** to your Partner Center account with a
role. That option only appears once an **Entra (Azure AD) tenant is associated**
with the Partner Center account — and by default **none is**.

## Symptom

Account settings → User management shows:

> You are currently signed in as `you@yourdomain`. To manage users, sign in
> with your associated Microsoft Entra ID credentials. **[Sign in with
> Microsoft Entra ID]**

You click it and **nothing happens** — it loops back. You cannot reach the
"Azure AD applications" tab. This is not a bug; there is simply **no tenant to
sign into**.

## Diagnosis

Account settings → **Tenants** (Organization profile → Tenants). The
"Current tenant associations" table is **empty**.

## Fix

1. Account settings → Tenants → **Associate Microsoft Entra ID** (NOT
   "Create" — your tenant already exists if you have an Azure subscription or
   M365).
2. It prompts: "Sign in with the global admin credentials of the other tenant
   you wish to associate."
3. **Sign in with the tenant's global-admin UPN** — see the critical gotcha
   below.
4. The tenant appears in the table. Now User management → **Azure AD
   applications** works.

## The critical gotcha: which login

Your Partner Center **sign-up email** (e.g. `admin@relux.works`) is a
**Microsoft Account (MSA)**. The **Entra tenant** is a separate directory
(e.g. `adminrelux.onmicrosoft.com`) whose users have UPNs on the
`.onmicrosoft.com` domain.

- Typing `admin@relux.works` gives **"relux.works isn't in our system. Make
  sure you typed it correctly."** — because `relux.works` is not a *verified
  domain* in the Entra tenant; it's only your MSA email.
- Use the tenant's global-admin UPN, e.g. **`ivan@adminrelux.onmicrosoft.com`**.

Find the exact UPN: [entra.microsoft.com](https://entra.microsoft.com) →
**Users → All users** → open the admin user → **User principal name**. Reset
its password there if you don't know it.

## Browser hygiene

The Entra sign-in opens a popup/redirect. If you have **multiple Microsoft
accounts** logged into the same browser, or an aggressive ad-blocker
(Brave Shields), the flow loops or picks the wrong account. Do it in a
**private/incognito window with only the correct account**, shields down for
`partner.microsoft.com`.

## Watch for two tenants

It's common to end up with a stray Entra tenant from a first, mistaken
sign-up (wrong region, etc.). Make sure the tenant you associate is the one
where your Azure AD *app registration* lives — match the **tenant ID**
(Directory ID) on both sides. Delete the stray tenant later to avoid confusion.
