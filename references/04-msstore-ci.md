# msstore CLI in CI (Linux)

The Microsoft Store submission API is driven from CI by the **msstore CLI**.
Two non-obvious things: it is **not a NuGet dotnet tool**, and on Linux it
needs a **secret service** running for credential storage.

## Install: self-contained binary, not a dotnet tool

`dotnet tool install --global MSStore.CLI` fails:

```
msstore.cli is not found in NuGet feeds https://api.nuget.org/v3/index.json.
```

The official CLI is shipped as **self-contained binaries** on GitHub releases
(`microsoft/msstore-cli`). On a Linux runner:

```bash
curl -sL https://github.com/microsoft/msstore-cli/releases/latest/download/MSStoreCLI-linux-x64.tar.gz | tar xz
chmod +x msstore
./msstore --version
```

The tarball extracts a `msstore` binary plus bundled `.so` files
(libSkiaSharp, libmsalruntime) — no .NET runtime needed. (The macOS builds are
**not** self-contained and do need .NET.)

## The secret-service requirement

`msstore reconfigure` stores credentials via **libsecret**, which needs a
running **freedesktop secret service**. On a bare runner you get:

```
DllNotFoundException: Unable to load shared library 'libsecret-1.so.0'
```
then, after installing libsecret:
```
Failed to write credential: 'The name org.freedesktop.secrets was not
provided by any .service files'
```

Fix: install libsecret **and** gnome-keyring, and run msstore inside a
`dbus-run-session` with an **unlocked keyring**:

```bash
sudo apt-get update
sudo apt-get install -y libsecret-1-0 gnome-keyring dbus-x11

dbus-run-session -- bash -euxc '
  echo "" | gnome-keyring-daemon --unlock --components=secrets
  ./msstore reconfigure --tenantId "$TENANT" --clientId "$CLIENT" \
      --clientSecret "$SECRET" --sellerId "$SELLER"
  ./msstore apps list
'
```

Both the reconfigure and the command that follows must run **inside the same
`dbus-run-session`** so they share the unlocked keyring.

## Credentials

- `--tenantId` — Directory (tenant) ID of the associated Entra tenant.
- `--clientId` — Application (client) ID of the Azure AD app.
- `--clientSecret` — a client secret (Certificates & secrets). **Store in CI
  secrets, never in the repo/logs.**
- `--sellerId` — Partner Center Seller ID (Account settings → Legal info).

## Verify auth

`msstore apps list` should print your reserved app:

```
│ ProductId    │ Display Name      │ PackageId                      │
│ 9XXXXXXXXXXX │ Your App          │ CompanyLLC.YourApp             │
```

If it does, the Azure AD app is correctly linked as Manager and the API is
usable. See `assets/store-auth-check.yml` for the full workflow.

## Publish syntax (updates only — see 06 for the first-submission caveat)

```bash
MSIX=$(ls pkg/*.msix | head -1)
./msstore publish "$MSIX" --appId <ProductId>
```

**`--appId <ProductId>` is required in CI.** Without it the CLI looks for a
LOCAL "initialized project" (state written by `msstore init` next to a
project) and dies with `Failed to find the AppId` — even for a published
app. It does **not** resolve the product from the MSIX package identity
(a day was lost to that wrong theory). Notes:

- The short alias is `-id` (SINGLE dash). `--id` does not exist and prints
  help — a second red herring.
- The positional arg is the **.msix path**; a bare product ID fails
  ("not a path or URL").
- `./msstore submission status <ProductId>` is a cheap read-only probe for
  "where is my certification" — handy in an auth-check workflow.
  Statuses you will actually see over a submission's life: `CommitStarted`
  (upload accepted, Partner Center ingesting) -> `PreProcessing` ->
  `Certification` (Microsoft testing the app on a real Windows VM) ->
  `Release`/`Publishing` -> `Published`. `CommitFailed` means the package or
  listing was rejected before certification — read the submission in Partner
  Center for the reason.

## Partner Center API flakiness (504s are routine)

The DevCenter v1.0 API regularly times out behind Azure Front Door: GETs and
submission creation can hang ~80 s and come back **504**. Two consequences:

1. **Retry the run.** A 504 on `GET /my/applications/<id>` or on submission
   creation is transient — the next dispatch usually goes through.
2. **A red run does not mean a failed submission.** After `Submission
   Committed` the CLI keeps polling status; if a POLL dies on a 504 the
   process exits non-zero while the submission is already committed
   server-side and proceeds to certification regardless. Verify with
   `msstore submission status` instead of resubmitting. The workflow in
   `assets/store-submit.yml` automates exactly this: on a non-zero publish
   exit it checks the submission status and greens the run when the status is
  CommitStarted/PreProcessing/Certification/....
