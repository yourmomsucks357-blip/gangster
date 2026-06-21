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
- Run the app with `npm start` or `node Main <repoUrl> [newName] [task]`.
  - If you pass a repo URL, the script clones that URL and pushes the fixes back to it.
  - If you omit the URL, it calls the GitHub API to create a new repo, which requires a
    valid `GITHUB_TOKEN` in `.env` (the repo's `Token` file is a placeholder template for
    that `.env`). Without a real token, only the URL-based path works.
- The script clones into `./workspace/` (gitignored) and wipes it on each run.
- AI code generation: if a `task` (3rd CLI arg or `CODE_TASK` env) is provided, the
  agent asks an LLM to write real code files into the cloned repo before committing.
  - Provider defaults to Hugging Face's Inference Providers router
    (`https://router.huggingface.co/v1`) with an open-weights model
    (`Qwen/Qwen2.5-Coder-32B-Instruct`). It speaks the OpenAI-compatible wire format
    but does NOT call OpenAI/Anthropic. Override with `LLM_BASE_URL` / `LLM_MODEL`
    (e.g. a local Ollama server) for other open/self-hosted models.
  - Auth: set `HF_TOKEN` (or `HUGGINGFACE_API_KEY`); the HF token needs the
    "Inference Providers" permission. Without a token, codegen is skipped gracefully
    (the rest of the pipeline still runs).
  - To test codegen without a real token, point `LLM_BASE_URL` at a local
    OpenAI-compatible mock server that returns `{"files":[{"path","content"}]}`.
- `push` failures are non-fatal: the run returns `{ pushed, pushError }` instead of
  throwing, so generated/scaffolded work isn't lost (relevant in this VM, where git is
  forced through the `cursor[bot]` identity and pushes to user repos get a 403).
- There is no lint or automated test setup in this repo (no ESLint config, no `test`
  script); "verification" means running the app end-to-end.
- Offline end-to-end check (no GitHub token needed): create a local bare repo to act as
  the remote, then run `node Main file:///path/to/bare.git`. The script will clone,
  add README/`.gitignore`/CI workflow, run `npm install`/`npm run build`, commit, and
  push back to the bare repo. This is the recommended smoke test.
