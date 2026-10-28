# rh: this fork's plugin builds

`rh` is upstream `main` plus the upstream pull requests listed below, so the
herdr plugin can run with them before upstream publishes them. `plugin-rh` is
its distribution branch, built with upstream's own `make plugin`
([ADR 0026](docs/adr/0026-plugin-distribution-branch.md)).

```sh
herdr plugin install randybias/Heeler --ref plugin-rh --yes
```

Switch back to `herdr plugin install ZingerLittleBee/Heeler --ref plugin --yes`
once every carried pull request is merged upstream and its plugin version is on
upstream's `plugin` branch.

## Carries

Each line is an upstream pull request whose change `rh` carries and that is not
yet on upstream's `plugin` branch.

- https://github.com/ZingerLittleBee/Heeler/pull/521 (merged; published from 0.7.1)

## Fork-only commits

These sit on upstream `main` before any carried pull request:

- `chore(rh): publish a fork's pre-release plugin from its own branches`:
  `SOURCE_BRANCH`, `PLUGIN_BRANCH` and SemVer pre-release versions in
  `scripts/publish-plugin.sh`.
- `chore(rh): carry upstream pull requests on rh`: this file.
- `chore(rh): version the plugin <X.Y.Z>-rh.<n>`: the plugin version
  `<upstream version>-rh.<n>`. The app reads it as `<upstream version>`, so it
  neither flags the fork as outdated nor offers to replace it with that release.

## Rebuilding rh

When upstream moves, or a carried pull request changes:

```sh
git fetch upstream && git fetch origin rh
git checkout -B rh upstream/main
git cherry-pick $(git log --reverse --format=%h --grep='^chore(rh)' upstream/main..origin/rh)
git fetch upstream pull/<n>/head:upstream-pr-<n>   # each open carried PR
git merge --no-ff upstream-pr-<n>
(cd plugin && npm test)
git push --force-with-lease origin rh
```

Leave out a carried pull request once upstream `main` has it. When upstream
bumps the plugin version, the version commit conflicts; resolve it to the new
`<upstream version>-rh.1`. Otherwise, any change in `plugin/` gets the next
`-rh.<n>`.

## Publishing plugin-rh

```sh
SOURCE_BRANCH=rh PLUGIN_BRANCH=plugin-rh DRY_RUN=1 make plugin
SOURCE_BRANCH=rh PLUGIN_BRANCH=plugin-rh YES=1 make plugin
```

`plugin-rh` is append-only like upstream's `plugin`: never force-push it.
