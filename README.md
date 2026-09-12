# localgrid.proxy

A GitHub Pages (`github.io`) mirror of [localgrid.dev](https://github.com/AstreliteHQ/localgrid.dev).

localgrid.dev deploys behind its own custom domain. This repo exists so the
app is also reachable at `astrelitehq.github.io/localgrid.proxy/`, without
touching that custom-domain deployment. It doesn't contain any app source of
its own: it installs the [`@astrelitehq/localgrid`](https://github.com/AstreliteHQ/localgrid.dev/pkgs/npm/localgrid)
npm package (published to GitHub Packages on every localgrid.dev release) and
builds *that* package's own source, with the Vite base path overridden to
`/localgrid.proxy/` instead of localgrid.dev's default `/`. The base path is
baked into the built assets at build time, so this is the only difference
between the two deployments.

## How it stays up to date

localgrid.dev's release workflow sends a `repository_dispatch` event to this
repo after it publishes a new `@astrelitehq/localgrid` version, which
triggers [`pages.yml`](.github/workflows/pages.yml) here to reinstall
(picking up the new version) and redeploy. It can also be run manually from
the Actions tab (`workflow_dispatch`).

## One-time manual setup

A few things can't be done from code and need to happen once in each repo's
settings:

- **GitHub Pages source**: in this repo's Settings → Pages, set the source
  to "GitHub Actions".
- **`PACKAGES_READ_TOKEN` secret** (in this repo): a personal access token
  with `read:packages` scope (classic) or Packages: Read (fine-grained) on
  the AstreliteHQ org, so `npm install` can pull `@astrelitehq/localgrid`
  from GitHub Packages. Add it under Settings → Secrets and variables →
  Actions.
- **`PROXY_DISPATCH_TOKEN` secret** (in localgrid.dev, not here): a PAT with
  `repo` scope (classic) or Contents: Read + Actions: Write (fine-grained)
  on this repo, so localgrid.dev's release workflow can dispatch the deploy
  above. See the `notify-proxy` job in localgrid.dev's
  `.github/workflows/release-please.yml`.

A single fine-grained PAT scoped to both repos (Packages: Read on
localgrid.dev, Contents: Read + Actions: Write on localgrid.proxy) can back
both secrets if you'd rather manage one token than two.

## Why `vitest`, Testing Library, and `@types/spark-md5` are devDependencies here

This repo has no tests of its own, but `package.json` still lists `vitest`,
`@testing-library/react`, `@testing-library/user-event`,
`@testing-library/jest-dom`, and `@types/spark-md5` as devDependencies.
That's a deliberate, permanent accommodation this repo makes for how it
consumes `@astrelitehq/localgrid` — not something expected to be cleaned up
once some other repo changes.

`@astrelitehq/localgrid`'s own `build` script is `tsc -b && vite build`,
and its `tsconfig.app.json` does `"include": ["src"]` with no exclusion for
`*.test.ts(x)` files, so `tsc -b` type-checks the whole `src` tree —
including its test files and `spark-md5`-typed source — on every build.
But the npm package only ships its own `dependencies`, not the
`devDependencies` those files need type declarations for (`vitest`,
`@testing-library/*` for the tests; `@types/spark-md5` for
`HashGeneratorWidget.tsx` itself, which imports the untyped `spark-md5`
package directly). Since this repo builds `@astrelitehq/localgrid` by
running its `build` script straight out of `node_modules` (see above), that
build fails here with `TS2307: Cannot find module 'vitest'` and similar
errors unless those packages are available too. `@testing-library/dom` (a
*peer* dependency of `@testing-library/react`/`user-event`) isn't listed
explicitly — npm auto-installs declared peers by default, so it's pulled in
on its own.

Because npm installs into one flat, hoisted `node_modules`, adding the same
packages as devDependencies *here* puts them where `tsc -b` running inside
`node_modules/@astrelitehq/localgrid` can still resolve them, without
touching localgrid.dev. Their versions should track the `devDependencies`
versions in [localgrid.dev's `package.json`](https://github.com/AstreliteHQ/localgrid.dev/blob/main/package.json)
for whichever files a given `@astrelitehq/localgrid` release ships; bump
them here if a new release needs something this list doesn't cover yet (a
build failing with another `TS2307`/`TS7016` for a `devDependency`-only
package is the signal).

This is intentionally contained to this repo rather than "fixed" in
localgrid.dev (e.g. by excluding test files from its production `tsc`
project). That app has no reason to carry build-config complexity to serve
a consumer that rebuilds its raw source with a different Vite base path —
this repo is the one adapter-shaped place that already exists to hold
differences like that (see the base-path override above), so this is where
the accommodation belongs.

## Lockfile

`package-lock.json` isn't committed yet (see `.gitignore`): generating one
requires resolving `@astrelitehq/localgrid` from GitHub Packages, which
needs the same `read:packages`-scoped credentials as above. Once you have
those locally (`export NODE_AUTH_TOKEN=...` with a PAT that has
`read:packages`), run `npm install`, commit the generated
`package-lock.json`, remove it from `.gitignore`, and switch
[`pages.yml`](.github/workflows/pages.yml) from `npm install` back to
`npm ci` for reproducible installs.

## Local development

```bash
export NODE_AUTH_TOKEN=<a PAT with read:packages on AstreliteHQ>
npm install      # needs npm >= 12; older npm can crash resolving vitest's
                 # optional peer deps, see the "Upgrade npm" step in pages.yml
npm run dev      # dev server, base path defaults to /
npm run build    # production build at /localgrid.proxy/, output in ./dist
```
