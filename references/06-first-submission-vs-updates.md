# First submission vs updates

**Do the first submission once in the Partner Center UI; automate every
update after.**

## Why submission #1 is manual (corrected 2026-07-09)

An earlier version of this document blamed `Failed to find the AppId` on the
package identity "not being registered until the first submission commits".
**That theory was wrong.** The error simply means `--appId` was not passed:
`msstore publish` looks for a LOCAL "initialized project" (`msstore init`
state), never for the MSIX package identity — it fails identically for a
published app (verified live, update #1 of a published product).

```bash
./msstore publish path/to.msix --appId <ProductId>   # works pre- and post-#1
```

The real reason to do submission #1 in the UI is **certification content**:
the commit is validated against listing (description + ≥1 screenshot), age
ratings, properties and pricing — none of which `publish` creates. Filling
those once in Partner Center takes ~10 min; after that the CLI owns packages.

One more first-submission trap: if a DRAFT submission was created in the
Partner Center UI and left pending, the CLI cannot upload into it — it errors
"can't upload the packages for submissions created in Partner Center. Please,
delete it and try again." Delete the UI draft, let the CLI create its own.

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

Use a **manual-trigger** workflow (never auto on every tag — you don't want
betas publishing publicly):

```bash
MSIX=$(ls pkg/*.msix | head -1)
./msstore publish "$MSIX" --appId <ProductId>   # inside dbus-run-session, see 04
```

**Supersede semantics (verified live):** if the app already has a published
version AND a pending submission exists (e.g. a previous beta still in
certification), `publish` **deletes the pending submission and creates a
fresh one** — shipping a newer build over an in-cert one is a single command,
no Partner Center visit. Only a pending FIRST submission (nothing published
yet) is reused instead of deleted — and only if the CLI created it.

**Release cadence vs the certification clock (learned over four live
submissions).** A supersede does not "update" the in-flight submission — it
DELETES it and starts a brand-new certification from zero. Superseding a
submission that is already deep in `Certification` works flawlessly, but the
hours it had accumulated are gone. Consequences for a fast beta cadence:

- If you publish every beta, the Store channel is perpetually ~one
  certification behind and may never catch up while you keep shipping.
- Let the channels you control (GitHub releases, an in-app updater) run ahead
  freely; batch Store submits per meaningful milestone, or supersede
  consciously knowing you reset the clock.
- Observed certification times for a small utility app: from a few hours to
  1+ day. Plan around "up to 3 business days" as Microsoft states.
- The lifecycle you want to see in the publish log:
  `Deleting existing Submission` -> `Creating new Submission` ->
  `Submission Committed` -> `CommitStarted` -> `PreProcessing` ->
  `Certification`. After certification the STORE signs the package with
  Microsoft's certificate (see 08) — an unsigned CI-packed MSIX is fine for
  this channel.

**Expect Partner Center 504s** during upload/commit/status-poll — see §04 for
why a red run can still be a committed submission and how
`assets/store-submit.yml` verifies via `msstore submission status` before
declaring failure.

See `assets/store-submit.yml`.
