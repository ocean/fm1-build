# fm1-build

One shared, reusable GitHub Actions workflow that builds a firmware for the **M-VAVE FM-1** (and SMK-37) from its own
repository and signs a **build attestation** for the packages. The [FM-1 Firmware Hub](https://thefirmwarehub.com)
shows a firmware as **"Built from source"** only when its package was attested by this workflow, at a tag the hub has
reviewed (currently `v1`). The badge links the exact commit and the public build log.

**What the badge means:** the file you install is exactly the one GitHub built from that commit, and anyone can check
it. **What it does not mean:** that the firmware is safe. The code is reviewed only by those who read it, but now it
can be: the binary is tied to a public snapshot of the source.

## What the workflow does (and nothing else)

It runs as three jobs, and each gets only the permissions it needs:

1. **build** (read-only): checks out the calling repository at the commit of the run, then installs Python with
   Pillow (pinned, hash-checked) and JieLi's Linux toolchain (its SHA-256 checked). It can also add the JieLi AC79 SDK
   (its commit checked) and Zig (pinned). Then it runs **your repository's** build command, and your tests if you give
   them, and lists the packages with their SHA-256 in `SHA256SUMS`.
2. **attest** (sign only): attests exactly the packages in `SHA256SUMS` with
   [`actions/attest-build-provenance`](https://github.com/actions/attest-build-provenance).
3. **release** (on a tag; write): checks the packages against `SHA256SUMS` again, then publishes them as the release
   of that tag.

Your build command runs where it can't sign anything or write to your repository. The jobs that can do those
things run none of your code and nothing downloaded.

Every action is pinned to a commit SHA. The file is short on purpose: review it in
[`.github/workflows/build.yml`](.github/workflows/build.yml).

## Use it

In your firmware repository, add `.github/workflows/release.yml`:

```yaml
name: release
on:
  push:
    tags: ['v*']
jobs:
  build:
    uses: zvenson/fm1-build/.github/workflows/build.yml@v1
    permissions:
      contents: write
      id-token: write
      attestations: write
    with:
      build: ./build.sh --release "${GITHUB_REF_NAME#v}"
      packages: build/*.fwsc
      # test: sh tests/run_tests.sh
```

Push a tag (e.g. `v1.2`): GitHub builds, attests and releases the packages. The hub picks the release up and shows
the badge on your firmware's card. Verify a package yourself with the GitHub CLI:

```sh
gh attestation verify your-firmware.fwsc --repo <owner>/<repo> \
  --signer-workflow zvenson/fm1-build/.github/workflows/build.yml
```

## Inputs

| input | required | default | what |
|---|---|---|---|
| `build` | yes | | your build command, run in the repository root (bash) |
| `packages` | yes | | the packages to attest and release (a glob) |
| `test` | no | `""` | your test command |
| `tests-block` | no | `true` | whether a failing test stops the release |
| `sdk` | no | `true` | clone the JieLi AC79 SDK (`AC79NN_SDK_V1.2.1_2023-12-13`) to `~/fw-AC79_AIoT_SDK` (`$AC79_SDK`) |
| `zig` | no | `false` | install Zig 0.17.0 (`$ZIG`, and on the `PATH`): `zig cc` for a WebAssembly build of the firmware |

`$JIELI_TOOLCHAIN` points at the toolchain during your build.

## Versions

Changes come as new tags (`v2`, …), never by moving a tag. See [CHANGELOG.md](CHANGELOG.md). Feedback and review are
welcome: open an issue, or find us on the FM-1 Discord.

Made for the FM-1 community by [the FM-1 Firmware Hub](https://thefirmwarehub.com) (zvenson), with the idea and review
of Ignis32 and keremimo.
