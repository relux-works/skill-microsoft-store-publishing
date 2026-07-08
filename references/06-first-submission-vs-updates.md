# First submission vs updates

**The first submission cannot be automated by the msstore CLI.** Do it once in
the Partner Center UI; automate every update after.

## Why the CLI can't do submission #1

`msstore publish path/to.msix` fails with:

```
Unhandled exception: MSStore.API.MSStoreException: Failed to find the AppId.
```

msstore resolves the target app by the **package identity** (e.g.
`ReluxWorksLLC.PulsarBarycenter`). That identity is only **registered on the
Store after the first submission commits**. So the very first publish is a
bootstrap chicken-and-egg: the CLI can't find the app by an identity the Store
doesn't know yet. (`msstore apps list` shows the app — but publish's identity
lookup still fails pre-first-submission.)

## The first submission (manual, ~10 min)

In Partner Center → your product → **Start submission**:

1. **Packages** — upload the MSIX. Delete any earlier rejected packages.
2. **Store listings** (per language) — description + **at least one
   screenshot** (min 1366×768). Screenshots are required; the app doesn't have
   to be running to provide them — a faithful high-res mockup of the actual UI
   is acceptable (render your real strings/layout to a PNG).
3. **Properties** — category, etc.
4. **Age ratings** — the questionnaire.
5. **Pricing and availability** — Free, markets.
6. **Submit for certification.**

### Bonus: cert is a free Windows smoke test

Certification **runs your app on a real Windows VM**. For a blind-built
Windows app (no hardware to test on), submitting IS the test:

- Passes → published; the Store signs it, users install with one click, no
  SmartScreen.
- Fails → Microsoft's cert report tells you exactly what broke — your first
  real Windows diagnostic.

Cert takes a few hours, up to 3 business days.

## Updates (automated)

Once submission #1 is committed, the package identity is registered and the
CLI works. Use a **manual-trigger** workflow (never auto on every tag — you
don't want betas publishing publicly):

```bash
MSIX=$(ls pkg/Pulsar-*-win.msix | head -1)
./msstore publish "$MSIX"     # inside dbus-run-session, see 04
```

See `assets/store-submit.yml`.
