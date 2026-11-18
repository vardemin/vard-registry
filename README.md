# The public Vard package registry

The default registry of [`vard`](https://github.com/vardemin/vardui) (name `vard`,
`https://vardemin.github.io/vard-registry`). It is a set of static files under
[`public/`](public), served by GitHub Pages and signed with a key compiled into `vard`
itself — clients verify every archive against the signed, hash-chained log, so they never
have to trust the host. Protocol and formats: [Registries and hosting](https://vardemin.github.io/vardui/packages/registry.html).

## Using it

Nothing to configure: with vard 0.3+ this is the default registry.

```bash
vard search chart
vard info tailwind
vard add tailwind
```

Offline, the packages bundled with vard are available as `--registry builtin`
(or `VARD_OFFLINE=1`).

## Publishing a package

Publishing is a pull request. The same gate as `vard publish` runs on it, and the package is
published when a maintainer merges.

1. Check your package locally (compiles, capabilities, tests, semver, AGENT.md budget, name):
   ```bash
   vard publish path/to/mypkg --registry https://vardemin.github.io/vard-registry --dry-run
   ```
   How to write one: [Writing a package](https://vardemin.github.io/vardui/packages/authoring.html).
2. Fork this repository and copy the package directory to `submissions/<name>/` (the
   directory name must equal `[package] name`). One package per directory; several
   packages per PR are fine.
3. Open a pull request. The `check` job runs the gate with you as the owner and comments
   nothing else: green = publishable.
4. After the merge, CI publishes every submission (as the PR author), removes
   `submissions/<name>/`, commits the updated `public/` and redeploys the site.

Rules (enforced by the gate):

- **Names.** Lowercase identifiers, not reserved, not one edit away from another owner's
  package. The first publisher of a name owns it; new versions must come from the owner's
  pull requests.
- **Versions are immutable** and semver is computed from the API diff: removing or changing
  something needs a major version.
- **Capabilities** declared in `vard.toml` must equal what the compiler computes; sensitive
  ones (secure, permissions, push, native, network) are shown to consumers, who must approve
  them.
- **AGENT.md** ≤ 6 KB, an `example/` app with tests, no path dependencies, ≤ 8 MB.
- Pull requests may change only `submissions/`. Registry files are written by CI only.

## Yanking

Open an issue naming the package and version (as its owner); a maintainer runs
`vard yank NAME@VERSION --registry .` and pushes. Yanked versions keep working for existing
lockfiles; new resolutions skip them.

## Verifying

```bash
vard registry verify https://vardemin.github.io/vard-registry
```

## Layout

| Path | What |
|---|---|
| `public/` | the registry: `config.json`, `catalog.json`, `index/`, `blobs/`, `log.txt`, `log.head` |
| `submissions/` | packages waiting for review |
| `.github/workflows/check.yml` | the gate on pull requests |
| `.github/workflows/publish.yml` | publish on merge, deploy GitHub Pages |

The signing key lives only in the `REGISTRY_KEY` Actions secret and with the maintainer.
