---
description: 'Commit all changes and push to GitHub'
mode: 'agent'
---

# Commit and Push

You are an experienced engineer. Stage all current changes, write a clean **Conventional Commits** message, and push to the GitHub remote.

## Inputs

- **Branch (optional):** Provided by the user. If not supplied, default to `main`.
- **Commit message hint (optional):** A short description from the user. If not supplied, infer the message from the staged diff.
- **Project conventions:** Follow [agents.md](../../agents.md). Treat the workspace root as the git working directory.

## Steps

1. **Verify a git repository exists.** Run `git rev-parse --is-inside-work-tree`. If it fails, stop and report — do not initialize a repo silently.
2. **Inspect state.** Run `git status --short` and `git diff --stat HEAD` (or `git diff --cached --stat` if already staged) to understand what will be committed.
3. **Stop and report if there is nothing to commit.** Do not create empty commits.
4. **Ensure unwanted files are gitignored.** Before staging, scan the working tree and `git status --short` output for paths that should never be committed:
   - **Dependencies:** `node_modules/`, `bower_components/`, `vendor/`, `.pnp/`, `.pnp.js`
   - **Build output:** `dist/`, `build/`, `out/`, `.next/`, `.nuxt/`, `.svelte-kit/`, `coverage/`, `*.tsbuildinfo`
   - **Environment & secrets:** `.env`, `.env.*` (except `.env.example`), `*.pem`, `*.key`, `secrets.json`
   - **Editor & OS cruft:** `.DS_Store`, `Thumbs.db`, `.idea/`, `.vscode/` (unless intentionally tracked), `*.swp`, `*.swo`
   - **Logs & caches:** `*.log`, `npm-debug.log*`, `yarn-debug.log*`, `.cache/`, `.parcel-cache/`, `.turbo/`, `.eslintcache`
   - For each match, ensure the pattern is present in the nearest `.gitignore` (create one at the repo root if none exists, or update the appropriate sub-package `.gitignore`). Append missing patterns with a brief comment grouping (e.g., `# dependencies`, `# build output`).
   - If any of these paths are **already tracked**, run `git rm -r --cached <path>` to untrack them, then ensure they are in `.gitignore`. Report each untracked path back to the user.
   - Never commit secrets — if a `.env` or key file appears tracked, STOP and warn the user before proceeding.
5. **Stage everything tracked + untracked.** Run `git add -A`.
6. **Write the commit message** using the rules in *Commit Message Rules* below. Infer the type, scope, and body from the staged diff and the user's hint (if any).
7. **Show the proposed message to the user before committing** when the change touches more than 10 files OR more than 500 lines. For smaller changes, proceed directly.
8. **Create the commit** using `git commit -m "<subject>" -m "<body line>" -m "<body line>" ...` (one `-m` per logical bullet) so PowerShell quoting stays simple. Never use `--no-verify`.
9. **Determine the target branch.**
   - If the user supplied a branch, use it.
   - Otherwise default to `main`.
   - If the current branch differs from the target, **stop and ask** before switching — do not silently change branches.
10. **Push.** Run `git push -u origin <branch>`. If the remote rejects (non-fast-forward), STOP and report — do not run `git push --force` or `--force-with-lease` without explicit user confirmation.
11. **Report** the commit SHA, the message subject, the file count + insertion/deletion stats, the remote URL, the branch, and any `.gitignore` updates or `git rm --cached` actions performed in step 4.

## Commit Message Rules

Follow [Conventional Commits 1.0.0](https://www.conventionalcommits.org/):

- **Subject line:** `<type>(<scope>): <short summary>`
  - `type` ∈ `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
  - `scope` is optional but encouraged (e.g., `specs`, `storage`, `board`, `dnd`, `ci`).
  - Subject ≤ **72 characters**, imperative mood, no trailing period.
- **Body:** One blank line after the subject, then 2–6 bullets describing what changed and why. Each bullet ≤ 100 characters.
- **Breaking changes:** Add `BREAKING CHANGE: <description>` in the footer if applicable. Mark the subject with `!` (e.g., `feat(api)!: …`).
- **No co-author trailers, sign-offs, or AI-generated boilerplate** unless the user requests them.
- **Specs-only changes** use `docs(specs): …`. **Prompt/template changes** use `chore(prompts): …` or `docs(templates): …`.
- **Reject** subjects like "update files", "wip", "misc", "fix stuff" — they fail this prompt's quality bar; rewrite the subject.

## Quality Checklist

Before pushing, verify ALL items pass:

- [ ] Working tree is a git repo and a remote named `origin` exists.
- [ ] Something is actually staged (no empty commit).
- [ ] Subject line uses Conventional Commits format and is ≤ 72 characters.
- [ ] Body contains 2–6 informative bullets; no "wip"/"misc"/"update files".
- [ ] No secrets, credentials, `.env` files, or build artifacts (`dist/`, `node_modules/`) are being committed. If detected, STOP and warn the user.
- [ ] `.gitignore` covers `node_modules/`, build output, `.env*`, logs, and OS/editor cruft. Any newly-discovered unwanted paths were added (and untracked via `git rm --cached` if needed).
- [ ] Branch matches the user's input (or `main` by default); no silent branch switch.
- [ ] No `--force` / `--no-verify` flags used.
- [ ] After push, the commit SHA is reported back to the user.

## Output

After completing, respond with:

1. ✅ Branch: `<branch>`
2. ✅ Commit: `<short-sha> — <subject>`
3. ✅ Stats: `<N> files, +<I>/-<D>`
4. ✅ Remote: `<origin url>`
5. Any warnings (skipped files, large diff, push protection alerts).

## Failure / Recovery

- **Nothing to commit** → report cleanly, do nothing.
- **Pre-commit hook fails** → report the hook output verbatim and stop. Do not bypass with `--no-verify`.
- **Push rejected (non-fast-forward)** → report and ask the user whether to `git pull --rebase` or force-push. Do not act unilaterally.
- **Detached HEAD** → stop and ask the user how to proceed.
- **Uncommitted changes that look like build output or secrets** → list them and ask before staging.
