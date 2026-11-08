---
description: Update CHANGELOG.md from recent changes, commit it, and create an annotated semver tag
argument-hint: [major | minor | patch | X.Y.Z]
model: haiku
allowed-tools: Read, Edit, Bash(git status:*), Bash(git log:*), Bash(git diff:*), Bash(git describe:*), Bash(git tag:*), Bash(git add CHANGELOG.md), Bash(git commit:*), Bash(date:*)
---

## Context

- Today: !`date +%F`
- Latest tag: !`git describe --tags --abbrev=0 2>/dev/null || echo "none"`
- Existing tags (recent): !`git tag --sort=-v:refname | head -5`
- Status: !`git status --short`
- Commits since latest tag: !`git log $(git describe --tags --abbrev=0 2>/dev/null || git rev-list --max-parents=0 HEAD)..HEAD --pretty=format:"%h %s%n%b" --no-merges`

## Task

Release request: `$ARGUMENTS` (empty = infer the bump).

Do everything below in one run, without asking questions.

### 1. Pick the version (semver)

- Explicit `X.Y.Z` in the arguments: use it. `major`/`minor`/`patch`: bump the latest tag accordingly.
- Otherwise infer from the commits: breaking change -> major (minor while the version is 0.x), `feat` -> minor, anything else -> patch.
- Match the existing tag style (keep or omit the `v` prefix as the current tags do; default to `v`).
- No commits since the latest tag: stop and say so.

### 2. Update `CHANGELOG.md`

- Read the file first. Insert the new section directly below the intro, above the newest existing release. Never touch older entries.
- Heading: `## [X.Y.Z] - <today>`.
- Use only the subsections that apply, in this order: Breaking changes, Added, Changed, Fixed, Internal.
- Follow the file's own language and wording. The reference template uses Spanish headings (`Cambios incompatibles`, `Agregado`, `Cambiado`, `Corregido`, `Interno`); if the file is in English, use the English names.
- One bullet per user-visible change, written from the consumer's point of view (not commit hashes). Keep endpoints, flags and identifiers in backticks and keep issue/PR references like `(#16)` when the commits have them.
- Group related commits into one bullet. Skip noise (merge commits, formatting, the previous changelog commit).
- If `CHANGELOG.md` does not exist, create it with a `# Changelog` title and a one-line intro, then the section.

### 3. Commit

- Stage only `CHANGELOG.md` (`git add CHANGELOG.md`); leave any other working-tree changes alone.
- Commit message: `chore(release): <tag>`. Conventional commit, no body needed, **no `Co-Authored-By` and no AI attribution lines**.

### 4. Tag that commit

- Create an annotated tag with the exact version chosen in step 1, on the commit just created.
- Tag message: first line `Release <tag>`, blank line, then a short summary of the new CHANGELOG section (the same subsections and bullets, condensed; plain text, in the changelog's language). Pass it via a heredoc:

```bash
git tag -a <tag> -F - <<'EOF'
Release <tag>

<summary>
EOF
```

### 5. Report

Reply in 2-3 lines: the tag, the commit hash, and that nothing was pushed (`git push origin <tag>` is up to the user).

## Rules

- Never push, never delete or move an existing tag, never use `--force`. If the tag already exists, stop and report it.
