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
npm install
npm run dev      # dev server, base path defaults to /
npm run build    # production build at /localgrid.proxy/, output in ./dist
```
