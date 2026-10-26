# Elastishot visual check

A GitHub Action that compares a pull request's pages with its base branch and names the DOM elements that changed: `[data-testid="cta"]`, `#faq`, not just red pixels.

It runs [`elastishot pr`](https://github.com/Osman19702/elastishot#checking-a-pull-request-without-baselines). The base commit and the pull request are both built and served in the job, captured with the same browser, and compared:
- there are no baselines to commit or update;
- fonts and browsers on other machines can't cause differences;
- the pages are aligned before they are compared, so a banner that pushes everything down reads as one added banner, not a page of red.

## Usage

Add an `elastishot.config.mjs` with the pages to check (the [configuration](https://github.com/Osman19702/elastishot#checking-a-pull-request-without-baselines) explains the `pr` block for sites with a build step):

```js
export default {
  viewports: [{ name: 'desktop', width: 1280, height: 800 }, { name: 'mobile', width: 390, height: 844 }],
  capture: { fullPage: true },
  targets: [
    { name: 'home', url: '/' },
    { name: 'pricing', url: '/pricing' },
  ],
}
```

Then a workflow:

```yaml
name: Visual check
on: pull_request
permissions:
  contents: read
jobs:
  visual:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
        with:
          fetch-depth: 2
      - uses: actions/setup-node@v7
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - uses: Osman19702/elastishot-action@v0.3
```

The job summary gets a table of the pages that changed and the elements behind each change, with a link to the full report: a slider, flip, blink and overlay viewer for every page. By default a visual change does not fail the job. Set `fail-on-diff: true` to make it a required check.

## Inputs

| Input | Default | What it does |
|---|---|---|
| `base` | the pull request's base | Commit, branch or tag to compare against. |
| `config` | the nearest `elastishot.config.*` | Path to the config, relative to `working-directory`. |
| `working-directory` | `.` | Where to run; a package folder in a monorepo. |
| `args` | | More `elastishot pr` arguments, for example `home --threshold 0.99`. |
| `version` | this release's | The elastishot used when the project does not install it. |
| `fail-on-diff` | `false` | Fail the job when the UI changed. A check that could not finish always fails. |
| `artifact-name` | `elastishot-report` | Name of the uploaded report. |
| `install-browsers` | `true` | Install Playwright's Chromium and its system packages. |

## Outputs

| Output | What it holds |
|---|---|
| `exit-code` | `0` no visual change, `1` the UI changed or a page is new, `2` the check could not finish |
| `report-dir` | The run folder: `index.html`, `report.json`, `junit.xml`, `summary.md` |
| `artifact-url` | Where the uploaded report can be downloaded |

## Security

Use the `pull_request` trigger. `pull_request_target` would build code from a fork with access to your repository's secrets, and the check runs your install and build commands.

## Versions

`v0.3.0` installs elastishot 0.3.0 unless the project has its own. `v0.3` follows the latest 0.3 release. Dependabot's `github-actions` updates keep a pinned tag current.

## Licence

MIT
