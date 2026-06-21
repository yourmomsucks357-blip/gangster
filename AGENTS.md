# repo-agent

A small Node.js CLI ("repo-agent") that clones a GitHub repository (or creates a new
one), analyzes it for missing scaffolding (README, `.gitignore`, dependency manifest),
fixes those issues, adds a GitHub Actions CI workflow, runs install/build, then commits
and pushes the result. The entry point is the `Main` file (CommonJS, no `.js` extension;
`package.json` `main`/`start` already point at it).

## Cursor Cloud specific instructions

- Runtime: Node.js (v22.x). The script is CommonJS and `require()`s `execa` using its
  named export (`const { execa } = require("execa")`). This only works because Node 22
  supports `require()` of ESM modules — do not downgrade `execa` to v5 (its default-only
  CJS export makes `{ execa }` `undefined`). Keep `execa` at v9+.
- Run the app with `npm start` or `node Main <repoUrl> [newName]`.
  - If you pass a repo URL, the script clones that URL and pushes the fixes back to it.
  - If you omit the URL, it calls the GitHub API to create a new repo, which requires a
    valid `GITHUB_TOKEN` in `.env` (the repo's `Token` file is a placeholder template for
    that `.env`). Without a real token, only the URL-based path works.
- The script clones into `./workspace/` (gitignored) and wipes it on each run.
- There is no lint or automated test setup in this repo (no ESLint config, no `test`
  script); "verification" means running the app end-to-end.
- Offline end-to-end check (no GitHub token needed): create a local bare repo to act as
  the remote, then run `node Main file:///path/to/bare.git`. The script will clone,
  add README/`.gitignore`/CI workflow, run `npm install`/`npm run build`, commit, and
  push back to the bare repo. This is the recommended smoke test.
