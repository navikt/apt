# navikt/apt

Debian and Ubuntu packages for Nav's CLI tools, served over GitHub Pages.

This repo is to `apt` what [navikt/homebrew-tap](https://github.com/navikt/homebrew-tap)
is to `brew`: it holds no source, only packages built from the tools' own releases.

## Install

```bash
curl -fsSL https://navikt.github.io/apt/keyring/navikt-archive-keyring.gpg \
  | sudo tee /usr/share/keyrings/navikt-archive-keyring.gpg >/dev/null

echo "deb [signed-by=/usr/share/keyrings/navikt-archive-keyring.gpg] https://navikt.github.io/apt stable main" \
  | sudo tee /etc/apt/sources.list.d/navikt.list

sudo apt update
sudo apt install nav-pilot cplt
```

Until the archive is signed, the commands above fail: `apt` refuses an unsigned
archive, and adding `[trusted=yes]` to work around that turns off verification
entirely. Install from the release asset instead:

```bash
gh release download --repo navikt/copilot --pattern '*_amd64.deb'
sudo apt install ./nav-pilot_*_amd64.deb
```

## What publishes it

`.github/workflows/publish.yaml` runs hourly and on demand. It downloads the
`.deb` assets from the latest release of each tool, adds them to `pool/`,
rebuilds the indices under `dists/stable/`, signs `InRelease` and `Release.gpg`,
publishes the public key to `keyring/`, and commits if anything changed.

Reading public releases needs no token beyond the workflow's own, so nothing has
to be dispatched from the tool repos.

Old versions stay in the pool, so a pinned install keeps working.

## Status

The archive is not signed yet, so the install block above does not work. Two
issues track what remains:

- [#1](https://github.com/navikt/apt/issues/1) generate the signing key and add the secrets
- [#2](https://github.com/navikt/apt/issues/2) verify the archive once the tools ship their first `.deb`

## Signing

Two secrets:

| Secret | What it holds |
|---|---|
| `APT_SIGNING_KEY` | ASCII-armoured private key for the archive |
| `APT_SIGNING_KEY_ID` | that key's fingerprint |

The key signs an archive, nothing else. Give it no other use, and rotate it by
replacing both secrets and re-running the workflow: the next run republishes
`keyring/navikt-archive-keyring.gpg`, and clients pick the new key up on their
next `apt update`.

Without `APT_SIGNING_KEY` the workflow still builds the pool and indices, warns,
and removes any stale signature.

## Architectures

`amd64` and `arm64`. Both tools build for both.
