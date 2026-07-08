# MSIX identity & versioning

Two rules the Store enforces at upload time. Get them wrong and the package is
rejected before certification even starts.

## Identity must match the Store-assigned values exactly

When you reserve the app name, the Store assigns an identity. Find it at
Partner Center → your product → **Product management → Product Identity**:

```
Package/Identity/Name                     ReluxWorksLLC.PulsarBarycenter
Package/Identity/Publisher                CN=60105954-A0D9-4E89-B32D-18AF2F423ABE
Package/Properties/PublisherDisplayName   Relux Works, LLC
```

Your `AppxManifest.xml` `<Identity Name=... Publisher=...>` and
`<PublisherDisplayName>` must match these **byte-for-byte**, or you get:

```
Package acceptance validation error: ... the package Publisher ... does not match
```

The `Publisher` is the **Store's GUID CN** (`CN=<guid>`), not your friendly
company name. (Manifest note: a CN value containing a comma must be quoted —
`Publisher="CN=&quot;Relux Works, LLC&quot;"` — but for the Store you use the
GUID CN, which has no comma.)

## Version: revision MUST be zero

MSIX `Version` is a quad `Major.Minor.Build.Revision`. The Store **rejects any
non-zero revision**:

```
Apps are not allowed to have a Version with a revision number other than zero
specified in the app manifest. The package specifies 0.3.0.15.
```

Map your build/beta number into **Build** and pin **Revision = 0**. Example
mapping from a git tag `v0.3.0-beta.15`:

```bash
MAJOR_MINOR=$(echo "$TAG" | sed -E 's/^v//; s/^([0-9]+\.[0-9]+).*/\1/')   # 0.3
BUILD=$(echo "$TAG" | sed -E 's/.*[.-]([0-9]+)$/\1/; s/[^0-9]//g')         # 15
VERSION="${MAJOR_MINOR}.${BUILD}.0"                                        # 0.3.15.0
```

Versions must also **increase monotonically** across submissions.

## Trust level

For a packaged Win32/desktop app that shouldn't request full-trust, use
`uap10:TrustLevel="appContainer"` (no `runFullTrust`) with the LAN + internet
capabilities your app needs. The Store **overwrites** the manifest's Identity
Name/Publisher and PublisherDisplayName with the reserved values on
submission, so local/sideload placeholders are fine — but matching them avoids
sideload-testing confusion.

See `assets/AppxManifest.xml.in`.
