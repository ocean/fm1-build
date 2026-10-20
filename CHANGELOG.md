# Changelog

## v2 (proposed)

- Three jobs, each with only the permissions it needs. **build** is read-only, so your build and tests can't sign
  or write. **attest** signs exactly the packages in `SHA256SUMS`. **release** checks them again, and only it can
  write.
- Everything downloaded is pinned and checked: JieLi's toolchain by its SHA-256, the AC79 SDK by its commit, and
  Pillow by its hash.
- Every action on Node 24 (GitHub is retiring Node 20): checkout v7.0.1, setup-python v7.0.0, upload-artifact
  v7.0.1, download-artifact v8.0.1, attest-build-provenance v4.2.2. All pinned to commit SHAs.
- The release is made with `gh` (no third-party release action). It uploads to a release that already exists, and
  never replaces a file.
- New input `zig`: Zig 0.17.0, pinned, for a WebAssembly build.
- `packages` can be several globs. Their names must differ, since a release holds one file of a name.
- The same inputs and the same release as v1, so a firmware only changes `@v1` to `@v2`.

## v1 (2026-10-18)

The first version: checkout, Python + Pillow, JieLi's Linux toolchain, the AC79 SDK at a fixed tag (optional), the
repository's build and test commands, `actions/attest-build-provenance`, the release on a tag. All actions pinned to
commit SHAs. Tested with Shine On (zvenson/shineon).
