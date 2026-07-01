# apt-tap

Shared apt (Debian/Ubuntu) package repository for jamiesun tools, published via
GitHub Pages. One suite, many packages — the same shared-repository model as
[homebrew-tap](https://github.com/jamiesun/homebrew-tap) (which already hosts
formulae for several unrelated tools in one repo), just for `.deb` packages
instead of Homebrew formulae.

## Install

```sh
curl -fsSL https://jamiesun.github.io/apt-tap/pubkey.gpg | sudo gpg --dearmor -o /usr/share/keyrings/jamiesun-apt-tap.gpg
echo "deb [signed-by=/usr/share/keyrings/jamiesun-apt-tap.gpg] https://jamiesun.github.io/apt-tap stable main" | sudo tee /etc/apt/sources.list.d/jamiesun-apt-tap.list
sudo apt update
```

Then install any package published here, e.g.:

```sh
sudo apt install scoot-edge
```

## Packages

| Package | Source project |
| --- | --- |
| `scoot-edge` | [jamiesun/scoot](https://github.com/jamiesun/scoot) — optional fleet-agent companion for the scoot AI agent daemon |

## How this repository works

- `pool/` holds the actual `.deb` files, pushed here by each source project's
  own release workflow (standard apt `pool/<component>/<letter>/<source>/`
  layout, e.g. `pool/main/s/scoot-edge/`).
- `dists/` (the apt index: `Release`, `InRelease`, `Packages`, `Packages.gz`)
  is **never committed to git**. It is fully regenerated from `pool/` on every
  push by [`.github/workflows/publish.yml`](.github/workflows/publish.yml) via
  [`scripts/build-repo.sh`](scripts/build-repo.sh), and published straight to
  GitHub Pages as a build artifact. This keeps git history free of
  generated-index merge conflicts even when multiple unrelated projects push
  new packages concurrently.
- A single suite (`stable`) and a single component (`main`) are shared by every
  package here, exactly like `homebrew-tap` shares one `Formula/` directory
  across multiple unrelated tools.
- Releases are signed with a repository-owned GPG key
  (`APT_TAP_GPG_PRIVATE_KEY`, stored only as a secret on this repository — no
  source project's own workflow ever touches it). The public key is published
  at [`pubkey.gpg`](https://jamiesun.github.io/apt-tap/pubkey.gpg) once a
  signing key has been configured.

## Adding a new package from another project

1. Build your `.deb` in your own project's release workflow.
2. Push it into `pool/main/<first-letter-of-source-name>/<source-name>/` on
   the `main` branch of this repository. A PAT with push access to this repo,
   stored as a secret in your own project (e.g. `scoot` uses
   `APT_TAP_TOKEN`), is all that's needed — see the `apt` job in
   [`scoot`'s `release.yml`](https://github.com/jamiesun/scoot/blob/main/.github/workflows/release.yml)
   for a working example.
3. This repository's own workflow rebuilds and republishes `dists/`
   automatically on the next push to `pool/**`. Your project's workflow never
   needs the signing key.
