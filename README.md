# skill-microsoft-store-publishing

An agent **skill** (and human runbook) for publishing a Windows **MSIX** app
to the **Microsoft Store from CI** — including the first-time setup that isn't
obvious anywhere: the Partner Center account, the Microsoft **Entra tenant
association** trap, the Azure AD app + Manager role for the submission API, the
**msstore CLI** on Linux runners, MSIX identity/version rules, and why the
**first submission must be manual** while updates automate.

Distilled from the real, painful first launch of *Pulsar Barycenter*
([relux-works/barycenter](https://github.com/relux-works/barycenter)). Every
command here actually ran.

## Start here

Read [`SKILL.md`](SKILL.md) — it has the 30-second map and the five traps that
will actually get you. Then open the reference for your current step:

| # | Reference | What it covers |
|---|-----------|----------------|
| 01 | [Partner Center account](references/01-partner-center-account.md) | Sign-up, Individual vs Company, verification, country availability |
| 02 | [Entra tenant association](references/02-entra-tenant-association.md) | **The #1 time sink** — associate the tenant, and which login to use |
| 03 | [Azure AD app](references/03-azure-ad-app.md) | Register app, secret, link as Manager, Seller ID |
| 04 | [msstore CLI in CI](references/04-msstore-ci.md) | Self-contained binary, libsecret/dbus/keyring, auth |
| 05 | [MSIX identity & versioning](references/05-msix-identity-versioning.md) | Match Store identity; revision must be 0 |
| 06 | [First submission vs updates](references/06-first-submission-vs-updates.md) | First is manual; updates automate; cert = free Win test |
| 07 | [Windows GUI build (Go)](references/07-windows-gui-build.md) | `-H windowsgui`, file logging, Segoe UI, go-winres |
| 08 | [Signing options](references/08-signing-options.md) | Store (free) vs Azure Trusted Signing (region-locked) vs EV |

## Assets

Working, ready-to-adapt files in [`assets/`](assets/):

- `store-auth-check.yml` — verify API credentials from CI (`msstore apps list`).
- `store-submit.yml` — push an MSIX **update** to the Store from CI.
- `AppxManifest.xml.in` — MSIX manifest template (identity/version placeholders,
  appContainer trust level).

## Install as a skill

This repo follows the `SKILL.md` convention (Claude Code / Codex skills).
Install it into an agent runtime by symlinking:

```bash
git clone https://github.com/relux-works/skill-microsoft-store-publishing
ln -s "$PWD/skill-microsoft-store-publishing" ~/.claude/skills/skill-microsoft-store-publishing
ln -s "$PWD/skill-microsoft-store-publishing" ~/.codex/skills/skill-microsoft-store-publishing
```

## License

MIT.
