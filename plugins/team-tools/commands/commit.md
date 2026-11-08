---
description: Commit recent changes with a Conventional Commit and a descriptive body, linking the issue if one exists
argument-hint: "[issue number] [extra context]"
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git branch:*), Bash(git add:*), Bash(git commit:*), Bash(git remote:*)
---

## Context

- Branch: !`git branch --show-current`
- Remote: !`git remote -v`
- Status: !`git status --short`
- Staged diff: !`git diff --cached --stat`
- Unstaged diff: !`git diff --stat`
- Recent commits (for style): !`git log --oneline -8`

User arguments: $ARGUMENTS

## Task

Create ONE commit for the recent changes. Write everything in English.

1. **Scope the change.** Read the full diff (`git diff --cached`, or `git diff` if nothing is staged). If nothing is staged, stage only the files that belong to this change — never `git add -A` blindly, and never stage secrets, `.env` files, or build output. If the diff mixes unrelated concerns, stop and propose a split instead of committing.

2. **Find the issue number**, in this order:
   - A number in `$ARGUMENTS`.
   - The branch name (`123-fix-login`, `feature/123-x`, `fix/#123`, `issue-123`).
   - Earlier commit messages on the branch (`Refs #N`, `Closes #N`).
   - If the GitLab MCP tools are available and the remote is GitLab, search the project's open issues for a clear match.
   - If none is found, omit the footer. Do NOT invent a number and do NOT ask — just mention in the final answer that no issue was linked.

3. **Write the message** in Conventional Commits format:

   ```
   <type>(<optional scope>): <imperative summary, lowercase, max 72 chars, no period>

   <body>

   <footer>
   ```

   - Types: `feat`, `fix`, `refactor`, `perf`, `docs`, `test`, `build`, `ci`, `chore`, `style`, `revert`. Add `!` and a `BREAKING CHANGE:` footer for breaking changes.
   - Body (required, wrapped at ~72 cols): explain WHAT changed and WHY, not a file-by-file list. Cover the motivation or problem, the approach taken, notable side effects, and anything a reviewer should watch. Use short paragraphs or `-` bullets for distinct changes.
   - Footer: `Closes #N` when the change fully resolves the issue, otherwise `Refs #N`. For an issue in another GitLab project use `group/project#N`.
   - No `Co-Authored-By` lines and no AI attribution.

4. **Commit** with a heredoc so the multi-line message is preserved:

   ```
   git commit -F - <<'EOF'
   ...message...
   EOF
   ```

   If a hook fails, fix the cause and create a new commit; never use `--no-verify`.

5. **Do not push.** Finish by showing the commit hash, the subject line, and the linked issue (or "no issue linked").
