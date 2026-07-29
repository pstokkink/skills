---
name: bump-version
description: 'Bump a Python project version and prepare the release metadata. Use when the user wants to bump a package version, cut a release candidate, convert an rc to a release, update pyproject.toml or setup.py, refresh uv.lock with uv sync --all-extras, or add a dated changelog entry summarizing branch changes.'
argument-hint: 'major | minor | patch | rc | release'
user-invocable: true
disable-model-invocation: false
---

# Bump Version

Bump the version of a Python project, update the package metadata at its source of truth, refresh `uv.lock` when present, and add a changelog entry that matches the repository's existing release-note style.

## Inputs

Accept one of these arguments:

- `major`
- `minor`
- `patch`
- `rc`
- `release`

If the argument is omitted or invalid, ask the user to choose one.

## Outcome

Complete only when all of these are true:

- The new version is computed correctly from the current version.
- The version is updated in the repository's real source of truth.
- `uv.lock` is refreshed with `uv sync --all-extras` if `uv.lock` exists.
- The changelog file used by the project has a new entry dated today.
- The changelog entry follows the same heading, date, and bullet style as the recent entries.
- The changelog summary reflects the current branch's changes, not a generic placeholder.
- A focused validation step has run after edits.

## Procedure

### 1. Find the version source of truth

Determine where the project defines its package version before editing anything.

Check in this order:

1. `pyproject.toml`
2. `setup.py`
3. `setup.cfg`
4. package metadata files such as `src/<package>/__init__.py` or a dedicated `_version.py`
5. release or build tooling referenced from `pyproject.toml` or project docs

Do not assume the first file that contains a version string is authoritative. Prefer the file that packaging or build tooling actually reads.

Use one cheap check to confirm the source of truth, such as:

- the packaging config explicitly contains the project version
- the build backend documents a dynamic version source
- the current version appears in one file and other files derive from it

If the project uses dynamic versioning from tags or a plugin and there is no checked-in version value to edit, stop and tell the user that this skill needs a project-specific extension for that versioning system.

### 2. Read the current version

Parse the current version into:

- `major`
- `minor`
- `patch`
- optional `rcN` suffix

Support versions shaped like `X.Y.Z` and `X.Y.Z-rcN` or `X.Y.ZrcN`, depending on repository convention. Preserve the repository's separator style when writing the new version. If the repository has no existing rc example to copy, default to `-rcN`.

### 3. Choose the next version

Apply exactly one branch.

#### `major`

- If the current version is `X.Y.Z` or `X.Y.Z-rcN`, set the new version to `(X + 1).0.0`.
- Drop any rc suffix.

#### `minor`

- If the current version is `X.Y.Z` or `X.Y.Z-rcN`, set the new version to `X.(Y + 1).0`.
- Drop any rc suffix.

#### `patch`

- If the current version is `X.Y.Z` or `X.Y.Z-rcN`, set the new version to `X.Y.(Z + 1)`.
- Drop any rc suffix.

#### `rc`

Use this decision tree:

1. If the current version is already an rc version, bump only the rc number: `X.Y.Z-rcN` -> `X.Y.Z-rc(N+1)`.
2. Otherwise, ask the user whether the release candidate should target the next `major` or next `minor` release.
3. Compute that base version first.
4. Write the new version as that base version plus `rc1`, preserving the repository's rc separator style.

Do not invent a patch-target rc unless the repository already has that convention and the user explicitly asks for it.

#### `release`

- This branch is only valid when the current version is an rc version.
- Remove the rc suffix and keep the base version unchanged.
- If the current version is not an rc version, stop and tell the user there is no rc suffix to release.

### 4. Update the version everywhere that must match

Update the source-of-truth file first.

Then update any checked-in mirrors only if the repository already keeps them in sync, for example:

- a CLI `--version` constant
- `__version__` mirrors
- docs or examples that intentionally track the released version

Do not broaden scope into unrelated cleanup.

### 5. Refresh `uv.lock` when present

If `uv.lock` exists, run:

```bash
uv sync --all-extras
```

Run it from the project root. This must happen after the version edit so the lockfile reflects the new package metadata.

If `uv.lock` does not exist, skip this step.

### 6. Add the changelog entry

Find the changelog file the repository actually uses. Common candidates include:

- `CHANGES.md`
- `CHANGELOG.md`
- `HISTORY.md`

Read the last 10 release entries and copy their conventions exactly:

- heading level
- version placement
- date format
- bullet style
- whether entries are newest-first or oldest-first

Use today's date.

Build the summary from committed changes on the current branch. First determine the base branch for the current branch, then diff or log from that branch point to `HEAD`. Do not include uncommitted working-tree changes in the changelog summary unless the user explicitly asks for that.

Determine the base branch in this order:

1. the current branch's configured upstream, if that reflects the branch the work is based on
2. the repository's remote HEAD branch
3. an existing `origin/main` or `origin/master`, whichever exists and matches the repository setup

Do not assume the base branch is named `main`.

Use repository evidence to summarize what changed:

- `git diff` or `git log` from the merge-base with the detected base branch to `HEAD`
- changed tests
- changed docs
- changed user-facing behavior

Write a concise summary of the changes already made on the branch. Keep it factual. Do not pad with vague bullets like "maintenance" or "miscellaneous fixes" unless the branch really is that broad.

If the branch contains no meaningful committed changes beyond the version bump itself, say so plainly in the changelog entry rather than inventing content.

### 7. Validate

Run the narrowest useful checks after editing:

1. a targeted test or package metadata check if the repository has one
2. otherwise a focused command that confirms the project still parses and the lock refresh succeeded
3. otherwise inspect the diff for the touched files only

Minimum validation:

- confirm the new version string appears in the expected source-of-truth file
- confirm `uv.lock` was updated when present
- confirm the changelog contains today's dated entry in the correct location and style

## Decision Rules

- If the user supplied an argument, do not ask again unless the `rc` branch needs the major-versus-minor choice.
- If multiple version files exist, update only the authoritative one plus established mirrors.
- If the repository convention uses `rc1` instead of `-rc1`, preserve that existing style. If no convention is visible yet, default to `-rc1`.
- If both changelog file name and formatting are unclear, inspect recent entries before editing.
- When deriving branch history for the changelog, detect the base branch first and do not hardcode `main`. Repositories may use `master` or another branch name.
- If no changelog exists, stop and ask the user whether to create one rather than inventing a new release-notes file.
- If `uv` is unavailable but `uv.lock` exists, stop and tell the user validation is blocked by missing tooling.

## Prompting

Ask concise follow-up questions only when the workflow cannot continue safely:

- missing bump type
- `rc` without a clear target release level
- ambiguous source-of-truth version file
- no existing changelog file
- dynamic versioning that is not file-based

## Example Invocations

- `/bump-version patch`
- `/bump-version rc`
- `Bump this package to the next minor release and update the changelog.`
- `Prepare a release from the current rc version.`
