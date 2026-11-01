# Astro action pnpm lockfile fixture

A minimal Astro site exercising [MindTooth/action's pnpm lockfile support](https://github.com/MindTooth/action/tree/feat/support_pnpm_lockfile).

`package.json` declares `devEngines.packageManager.version: ^12.0.0`, with no legacy `packageManager` pin. The generated `pnpm-lock.yaml` records pnpm **12.8.1**, an older version within that range. The action must install that locked version instead of resolving the newest matching release.

The first three workflows reuse the root site and lockfile. The legacy fixture has its own manifest and pnpm 10 lockfile in `fixtures/legacy-pnpm`:

| Workflow | Action input | Expected pnpm | Setup path |
| --- | --- | --- | --- |
| [Automatic detection](.github/workflows/fixture.yml) | Omitted | 12.8.1 | pnpm/setup |
| [Explicit manager](.github/workflows/pnpm.yml) | `package-manager: pnpm` | 12.8.1 | pnpm/setup |
| [Explicit version override](.github/workflows/pnpm-override.yml) | `package-manager: pnpm@12.8.2` | 12.8.2 | pnpm/action-setup |
| [Legacy packageManager](.github/workflows/pnpm-legacy.yml) | `path: fixtures/legacy-pnpm` | 10.30.0 | pnpm/action-setup |

Each workflow builds and uploads the site through the feature branch, then checks the installed pnpm version with project version switching disabled, the selected action path, Node 24, the rendered page, and that the manifest and lockfile remain unchanged. The override also checks `pnpm --version` inside the action's build step to catch project version switching back to 12.8.1.

The legacy fixture declares only `"packageManager": "pnpm@10.30.0"`, without `devEngines.packageManager` or a `package-manager` action input. It verifies automatic detection, the legacy setup path, the installed pnpm version, and that the legacy manifest and lockfile remain unchanged. It also checks the pnpm version inside the action's build step.

Separate workflows avoid conflicts from the action's fixed Pages artifact name. They upload artifacts without deploying a website. Run them from the Actions tab or push a change to repeat the checks. Their action references intentionally follow the feature branch.

## Run locally

With pnpm 12.8.1 and Node 24 or newer:

```sh
pnpm install --frozen-lockfile
pnpm run build
```
