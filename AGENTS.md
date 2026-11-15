# AGENTS.md

Short link site for **<https://atdr.eu>**. A static site built with
[Suri](https://github.com/surishortlink/suri) (`@surishortlink/suri`, a
devDependency) and deployed to GitHub Pages automatically on every push to
`main`. Each entry in `src/links.json` becomes a page at `https://atdr.eu/<key>`
that redirects to its URL via an instant `<meta>` refresh.

## Git workflow: commit straight to `main`

This repo deliberately does **not** use pull requests. Link changes are one-line
edits to a data file, fully validated by the `pre-commit` hook and again by
`deploy.yml` before anything reaches the live site, so a branch and a PR would
add ceremony without adding safety. Commit to `main` and push.

This is an intentional exception to the GitHub Flow convention used elsewhere.
Don't open PRs here, and don't "fix" this by introducing branches. The main
ruleset lets the repository admin push straight to `main` because the repo
commits `.github/admin-bypass`, which atdr/github-boilerplate's `configure.sh`
reads. Everyone else, Dependabot included, merges through a PR with the gates
required.

Larger structural work (build tooling, workflows, `scripts/`) is the one case
where a branch is worth it, since those changes can break the deploy in ways the
link validator won't catch.

## Adding a link

All links live in `src/links.json` as a flat `"key": "URL"` map.

1. Pick a key:
   - Memorable links get a named key, e.g. `"github"`, `"cvpdf"`.
   - Ad-hoc links get a random 5 character alphanumeric key (mixed case), e.g.
     `"ZlTbO"`. Generate one yourself; any `[A-Za-z0-9]{5}` string is fine.
2. Confirm the key doesn't already exist in the file (keys are case sensitive,
   but avoid keys differing only by case).
3. Insert the entry in alphabetical order (case insensitive). The first two
   entries, `"/"` and `"new-key"`, are special and stay at the top.
4. Verify: `npm run check` (lint + validate + build), then check
   `build/<key>/index.html` contains the right URL.
5. Commit as `feat: add <key> link` (one commit per logical change) and push to
   `main`. Deployment is automatic; confirm with
   `curl -s https://atdr.eu/<key>/` after the
   [Deploy workflow](.github/workflows/deploy.yml) finishes.

## Removing or changing a link

Edit or delete the entry in `src/links.json`, then the same verify, commit
(`fix:`/`chore:`), and push flow. Note that short links may be printed or
bookmarked; prefer updating a key's URL over deleting the key.

## CI

The workflows come from atdr/github-boilerplate (pinned by tag, bumped by
Dependabot):

| Workflow           | What it runs                                                                                            |
| ------------------ | ------------------------------------------------------------------------------------------------------- |
| `ci.yml`           | `lint`, `validate` and `build` on every Node LTS line, then `ci gate`. On PRs, and called by the deploy |
| `deploy.yml`       | On push to `main`: `ci.yml` first, then the Suri build and the GitHub Pages deploy only if it passes    |
| `checks.yml`       | Markdown (mdformat, markdownlint) and the boilerplate conventions, then `checks gate`                   |
| `pull-request.yml` | PR title and commit messages are Conventional Commits, then `pull-request gate`                         |

A commit pushed straight to `main` gets its CI from the deploy, so nothing
reaches gh-pages without passing. Run the same checks locally with
`npm run check`. A `pre-commit` hook (`.githooks/pre-commit`, enabled
automatically after `npm install`) runs lint + validate before each commit;
bypass it with `git commit --no-verify`.

## Repo rules

- Never edit `build/` (generated) or `package-lock.json` by hand.
- `suri.config.json` controls redirect behavior; don't change it for link work.
- Formatting is enforced by Prettier (`npm run lint:fix` to fix), except
  Markdown, which mdformat owns (`mdformat .`), and the workflow files, which
  follow the boilerplate's layout. Both are listed in `.prettierignore`.
- Workflows: each file's `name:` is its filename stem, and every job sets
  `timeout-minutes`. See atdr/github-boilerplate's STANDARDS.md.
- `src/links.json` is validated by `scripts/validate-links.mjs`
  (`npm run validate`): valid http(s) URLs, no case-insensitive key collisions,
  `/` and `new-key` pinned at the top, and the rest in natural case-insensitive
  order.
