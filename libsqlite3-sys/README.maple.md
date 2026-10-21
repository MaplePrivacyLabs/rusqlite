# Maple SQLCipher provider patch

This branch carries one focused patch above rusqlite commit
`e88f112bef7899234a497baed5cc3c3d553deeb8` (rusqlite 0.40.2,
libsqlite3-sys 0.38.2). Maple selects this crate through an exact Git revision;
the rest of the upstream repository is unchanged.

## Selected sources and provider

- Backport the source selection from
  [rusqlite#1917](https://github.com/rusqlite/rusqlite/pull/1917), merged as
  `661107f6eb5d7477c61be1221744de3ad96c53e7`. `bundled-sqlcipher` selects
  `sqlcipher-src =420.0.0`: SQLCipher 4.20.0 / SQLite 3.54.1. Buildtime bindgen
  uses that same source's public header.
- Add `bundled-sqlcipher-openssl4`, selecting `openssl-src =400.0.2`
  (OpenSSL 4.0.3). This feature builds OpenSSL from source, explicitly selects
  SQLCipher's OpenSSL provider, and links only the resulting static crypto
  archive after the SQLCipher archive. It does not use an ambient OpenSSL
  installation. Platform library names and dependencies come from the build
  artifacts and target requirements.
- Reject competing provider/linkage features and a nonzero
  `LIBSQLITE3_SYS_USE_PKG_CONFIG` override on that route. Existing upstream
  provider routes remain available when this feature is absent.

No SQLCipher or OpenSSL C source is modified. Their exact registry archives and
source verification inputs are recorded by the consuming Maple workspace.
The old SQLCipher C file is retained here as upstream history but is unused
when `bundled-sqlcipher` selects the new source dependency.

## Maintenance and validation

This is a dependency build patch, not a change to rusqlite's public Rust API.
Keep it as one commit when rebasing onto a reviewed upstream release. Remove
the source-selection backport once a suitable released crate contains it;
remove the OpenSSL branch once a released compatible feature provides the
same explicitly pinned static provider.

Upstream's regular test matrix does not exercise this added OpenSSL feature.
The consuming application must verify selected dependency versions, runtime
SQLCipher/SQLite/OpenSSL versions, static linking, and native behavior on
each shipping target. A source-only review or a successful macOS build is not
Linux or Windows validation. The retained bindings expose an older compatible
ABI subset; query runtime versions instead of treating binding constants as
the selected engine version.
