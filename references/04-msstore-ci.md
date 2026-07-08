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
│ 9P26FDCWV1GC │ Pulsar Barycenter │ ReluxWorksLLC.PulsarBarycenter │
```

If it does, the Azure AD app is correctly linked as Manager and the API is
usable. See `assets/store-auth-check.yml` for the full workflow.

## Publish syntax (updates only — see 06 for the first-submission caveat)

```bash
MSIX=$(ls pkg/Pulsar-*-win.msix | head -1)
./msstore publish "$MSIX"     # matches the product by package identity
```

`--id` is **not** a valid flag (prints help). Passing a bare product ID as the
positional arg fails ("not a path or URL"). Pass the **.msix path**; msstore
matches the reserved product by the package's identity.
