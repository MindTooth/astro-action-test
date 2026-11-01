# Astro action pnpm lockfile fixture

A minimal Astro site exercising [MindTooth/action's pnpm lockfile support](https://github.com/MindTooth/action/tree/feat/support_pnpm_lockfile).

`package.json` declares `devEngines.packageManager.version: ^12.0.0`, with no legacy `packageManager` pin. The generated `pnpm-lock.yaml` records pnpm **12.8.1**, an older version within that range. The action must install that locked version instead of resolving the newest matching release.

The first three workflows reuse the root site and lockfile. The remaining fixtures have their own manifests and lockfiles under `fixtures/`:

| Workflow | Action input | Expected pnpm | Setup path |
| --- | --- | --- | --- |
| [Automatic detection](.github/workflows/fixture.yml) | Omitted | 12.8.1 | pnpm/setup |
| [Explicit manager](.github/workflows/pnpm.yml) | `package-manager: pnpm` | 12.8.1 | pnpm/setup |
| [Explicit version override](.github/workflows/pnpm-override.yml) | `package-manager: pnpm@12.8.2` | 12.8.2 | pnpm/action-setup |
| [Legacy packageManager](.github/workflows/pnpm-legacy.yml) | `path: fixtures/legacy-pnpm` | 10.30.0 | pnpm/action-setup |
| [No version declaration](.github/workflows/pnpm-fallback.yml) | `path: fixtures/no-version-pnpm` | npm `latest` tag | pnpm/action-setup |
| [Workspace member](.github/workflows/pnpm-workspace.yml) | `path: fixtures/workspace/apps/site`, `package-manager: pnpm` | 12.8.1 | pnpm/setup |
| [Per-project workspace lockfile](.github/workflows/pnpm-workspace-unshared.yml) | `path: fixtures/workspace-unshared/apps/site`, `package-manager: pnpm` | 12.8.2 | pnpm/setup |
| [Custom workspace lockfile directory](.github/workflows/pnpm-workspace-lockfile-dir.yml) | `path: fixtures/workspace-lockfile-dir/apps/site`, `package-manager: pnpm` | 12.8.1 | pnpm/setup |
| [Included lockfile directory](.github/workflows/pnpm-workspace-included-lockfile-dir.yml) | `path: fixtures/workspace-included-lockfile-dir/apps/site`, `package-manager: pnpm` | 12.15.0 | pnpm/setup |
| [Site version without root declaration](.github/workflows/pnpm-workspace-site-version.yml) | `path: fixtures/workspace-site-version/apps/site`, `package-manager: pnpm` | 12.8.2 | pnpm/setup |
| [Root declaration precedence](.github/workflows/pnpm-workspace-precedence.yml) | `path: fixtures/workspace-precedence/apps/site`, `package-manager: pnpm` | 12.8.1 | pnpm/setup |
| [Excluded workspace site](.github/workflows/pnpm-workspace-excluded.yml) | `path: fixtures/workspace-excluded/apps/site`, `package-manager: pnpm` | 12.8.2 | pnpm/setup |

Each workflow builds and uploads the site through the feature branch, then checks the installed pnpm version with project version switching disabled, the selected action path, Node 24, the rendered page, and that the manifest and lockfile remain unchanged. The override also checks `pnpm --version` inside the action's build step to catch project version switching back to 12.8.1.

The legacy fixture declares only `"packageManager": "pnpm@10.30.0"`, without `devEngines.packageManager` or a `package-manager` action input. It verifies automatic detection, the legacy setup path, the installed pnpm version, and that the legacy manifest and lockfile remain unchanged. It also checks the pnpm version inside the action's build step.

The no-version fixture declares neither `packageManager` nor `devEngines.packageManager`, and leaves the action's `package-manager` input unset. Its workflow checks out the feature branch once, runs the real lockfile-detection script with an isolated environment file, and asserts the missing-version warning and `VERSION=latest`. It then builds through that same action checkout and compares the installed pnpm version with npm's `latest` tag. The actual build also emits the warning as a GitHub annotation.

The workspace fixture puts the Astro site in `fixtures/workspace/apps/site`, with no member lockfile. The root `pnpm-workspace.yaml` includes `apps/*`, and the shared `pnpm-lock.yaml` records pnpm 12.8.1. Both root and site declare `devEngines.packageManager.version: ^12.0.0`; the site's declaration enables delegation to pnpm/setup. The workflow explicitly selects `package-manager: pnpm` because automatic detection does not search parent directories. It checks that the member lockfile is absent before and after building, pnpm 12.8.1 is used during the build, and the shared lockfile and manifests remain unchanged.

The per-project workspace fixture sets `sharedWorkspaceLockfile: false` in `fixtures/workspace-unshared/pnpm-workspace.yaml`. Only the site declares the pnpm range, and its lockfile records pnpm 12.8.2; the root has a separate dependency lockfile with no pnpm version declaration. This verifies that setup retains the member declaration and selects its locked version. The site lockfile was generated independently with pnpm 12.8.2. The workflow explicitly selects `package-manager: pnpm`, checks the version during the Astro build, and verifies that both lockfiles, manifests, and workspace configuration remain unchanged.

The custom-directory workspace fixture sets `lockfileDir: lockfiles` in `fixtures/workspace-lockfile-dir/pnpm-workspace.yaml`. Its generated shared lockfile is `lockfiles/pnpm-lock.yaml`, relative to the workspace configuration, and records pnpm 12.8.1. The nested site has no lockfile, and there is no default root lockfile. The workflow uses explicit `package-manager: pnpm`, verifies the selected version during the Astro build, checks the lockfile layout before and after, and requires the custom lockfile, manifests, and workspace configuration to remain unchanged.

The included-config workspace fixture delegates `lockfileDir` with `include: { './config/paths.yaml': [lockfileDir] }`. That local file sets `lockfileDir: lockfiles`, so the generated lockfile lives at `config/lockfiles/pnpm-lock.yaml`, relative to the included file rather than the workspace root. Both manifests declare `^12.15.0`, and the custom lockfile records pnpm 12.15.0. Local config includes require pnpm 12.15.0 or newer; see [pnpm's config includes documentation](https://pnpm.io/config-includes). The workflow uses explicit `package-manager: pnpm`, checks the selected version during the Astro build, rejects lockfiles at the root, site, and root-relative `lockfiles/` paths, and verifies that both config files, manifests, and the custom lockfile remain unchanged.

The excluded-site fixture has workspace patterns `apps/*` and `!apps/site`, so the Astro site is explicitly excluded. Both manifests declare `^12.0.0`, but the root lockfile records pnpm 12.8.1 and the site's own lockfile records 12.8.2. The site reuses the independently generated lockfile from the per-project fixture, which has the same dependencies and pnpm declaration. The workflow uses explicit `package-manager: pnpm` and checks pnpm 12.8.2 during the build, so accidentally treating the site as a workspace member fails. It also verifies that neither manifest, lockfile, nor workspace configuration changes.

The root-precedence workspace fixture gives the root `^12.0.0` with pnpm 12.8.1 locked, while the Astro site declares `^12.8.2`. The workflow explicitly sets `package-manager: pnpm` and checks that pnpm 12.8.1 is used during the build and remains installed. This version falls outside the site's range, demonstrating that the root declaration takes precedence.

The site-version workspace fixture has no root pnpm version declaration; the nested Astro site's `devEngines.packageManager` declares pnpm 12.8.2. The workflow uses explicit `package-manager: pnpm` and verifies that version during setup and build, along with the shared-lockfile layout and unchanged manifests.

Separate workflows avoid conflicts from the action's fixed Pages artifact name. They upload artifacts without deploying a website. Run them from the Actions tab or push a change to repeat the checks. Their action references intentionally follow the feature branch.

## Run locally

With pnpm 12.8.1 and Node 24 or newer:

```sh
pnpm install --frozen-lockfile
pnpm run build
```
