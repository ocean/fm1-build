# fm1-build

One shared, reusable GitHub Actions workflow that builds a firmware for the **M-VAVE FM-1** (and SMK-37) from its own
repository and signs a **build attestation** for the packages. The [FM-1 Firmware Hub](https://thefirmwarehub.com)
shows a firmware as **"Built from source"** only when its package was attested by this workflow, at a tag the hub has
reviewed (currently `v1`). The badge links the exact commit and the public build log.

**What the badge means:** the file you install is exactly the one GitHub built from that commit, and anyone can check
it. **What it does not mean:** that the firmware is safe. The code is reviewed only by those who read it, but now it
can be: the binary is tied to a public snapshot of the source.

## What the workflow does (and nothing else)

1. checks out the calling repository at the commit of the run
2. installs Python + Pillow and JieLi's Linux toolchain from JieLi's own server (`pkgman.jieliapp.com`)
3. optionally clones the JieLi AC79 SDK at a fixed tag from JieLi's gitee repository
4. runs **your repository's** build command (and, if given, your tests)
5. attests the packages with [`actions/attest-build-provenance`](https://github.com/actions/attest-build-provenance)
   and, on a tag, publishes them as the release of that tag

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

`$JIELI_TOOLCHAIN` points at the toolchain during your build.

## Versions

Changes come as new tags (`v2`, …), never by moving a tag. See [CHANGELOG.md](CHANGELOG.md). Feedback and review are
welcome: open an issue, or find us on the FM-1 Discord.

Made for the FM-1 community by [the FM-1 Firmware Hub](https://thefirmwarehub.com) (zvenson), with the idea and review
of Ignis32 and keremimo.
