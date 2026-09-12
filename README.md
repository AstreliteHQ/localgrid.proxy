# localgrid.proxy

Mirrors [localgrid.dev](https://github.com/AstreliteHQ/localgrid.dev) to
`astrelitehq.github.io/localgrid.proxy/`, alongside its custom-domain
deployment. No app source lives here: it installs the
[`@astrelitehq/localgrid`](https://github.com/AstreliteHQ/localgrid.dev/pkgs/npm/localgrid)
npm package and builds *that* package's source with a different Vite base
path baked in.

## Staying up to date

localgrid.dev's release workflow dispatches a rebuild here on every new
package version. Also rebuilds on push to `main`, or manually via
`workflow_dispatch`. `ci.yml` builds (not deploys) on every push, any
branch, to catch a broken build before it reaches `main`.

## Extra devDependencies

`vitest`, Testing Library, and `@types/spark-md5` are devDependencies here
even though this repo has no tests. `@astrelitehq/localgrid`'s own build
(`tsc -b`) type-checks its whole source tree, tests included, but the
published package doesn't ship its devDependencies. Listing the same ones
here lets npm's hoisted `node_modules` satisfy that type-check. Keep their
versions in sync with localgrid.dev's own `package.json`.

## Lockfile

Not committed yet (see `.gitignore`); generating one needs
`read:packages` credentials to resolve `@astrelitehq/localgrid`. Once you
have one, run `npm install`, commit `package-lock.json`, and switch
`pages.yml` to `npm ci`.

## Local development

```bash
export NODE_AUTH_TOKEN=<a PAT with read:packages on AstreliteHQ>
npm install
npm run dev      # base path defaults to /
npm run build    # production build at /localgrid.proxy/
```
