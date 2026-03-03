# git-pickdir

Extract one or more subdirectories from the current Git repository into a new repository, with contents flattened to the root and history rewritten so only those paths remain.

## Requirements

- **Bash** (with `set -euo pipefail` support)
- **git** ≥ 2.x
- **git-filter-repo** ([install](https://github.com/newren/git-filter-repo#installation)) — must be on your `PATH`

## Usage

```bash
git-pickdir [OPTIONS] REPODIR SUBDIRS...
```

- **REPODIR** — Target directory for the new repository (must not already exist).
- **SUBDIRS...** — One or more repository-relative subdirectory paths to extract.

### Example

```bash
git-pickdir ../newrepo src/lib include/common
```

Creates `../newrepo` containing only the contents of `src/lib` and `include/common`, at the root of the new repo, with history preserved for those paths.

## Install

Put `git-pickdir` somewhere on your `PATH`, e.g.:

```bash
cp git-pickdir ~/bin/
# or
sudo cp git-pickdir /usr/local/bin/
```

## Errors (exit non-zero, message to stderr)

The tool exits with an error if:

- Not run inside a Git repository
- Repository has no commits
- `REPODIR` already exists
- No `SUBDIRS` provided
- Any `SUBDIR` does not exist at `HEAD`
- Any `SUBDIR` is an absolute path or contains `..`
- One subdir is a prefix of another (e.g. `src` and `src/lib`)
- Flattening would produce a filename collision (same name from different subdirs)
- `git-filter-repo` is not installed or not in `PATH`
- Any Git or filter-repo command fails
- No files remain after filtering

## Behavior

- **Repo root** is detected with `git rev-parse --show-toplevel`; all logic is based on that root.
- **Copy vs clone**: If the repo is “fresh” (no loose objects, no remotes), the tool uses `cp -a` for speed; otherwise it uses `git clone --no-local` to avoid hardlinks.
- **History** is rewritten with `git filter-repo` so only the chosen paths exist and are moved to the repository root.
- The new repository is independent: no remotes, no hardlinks to the original.
