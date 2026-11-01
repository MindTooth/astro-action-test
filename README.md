# Astro action pnpm lockfile fixture

A minimal Astro site exercising [MindTooth/action's pnpm lockfile support](https://github.com/MindTooth/action/tree/feat/support_pnpm_lockfile).

`package.json` declares `devEngines.packageManager.version: ^12.0.0`, with no legacy `packageManager` pin. The generated `pnpm-lock.yaml` records pnpm **12.8.1**, an older version within that range. The action must install that locked version instead of resolving the newest matching release.

The [fixture workflow](.github/workflows/fixture.yml) uses automatic lockfile detection. It builds and uploads the site through the feature branch, then checks the installed pnpm version with project version switching disabled, the selected action path, Node 24, the rendered page, and that the manifest and lockfile remain unchanged.

The workflow uploads a Pages artifact; it does not deploy a website. Run it from the Actions tab or push a change to repeat the checks. Its action reference intentionally follows the feature branch.

## Run locally

With pnpm 12.8.1 and Node 24 or newer:

```sh
pnpm install --frozen-lockfile
pnpm run build
```
