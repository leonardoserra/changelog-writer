# Changelog

All notable changes to this project are documented in this file.

## 1.3.1 - 2026-10-20

- Changed
  - `/changelog-writer` and `/changelog-writer:init` now leave out changes to `AGENTS.md` and to `.claude/` skills and
    commands, like `CLAUDE.md`. Before, edits to agent configuration could show up as release lines.

## 1.3.0 - 2026-10-16

- Changed
  - `/changelog-writer [version X.Y.Z | X.Y.Z] from <commit>` builds the entry from `<commit>` (included) onwards on
    `develop`, instead of from where `develop` leaves `master`. It asks for another commit if `<commit>` does not
    exist or is not on `develop`, and warns when the range includes commits already on `master`.
  - `/changelog-writer:init [X.Y.Z] from <commit>` ignores the history before `<commit>`: tags that do not contain it
    are skipped, and the oldest remaining entry starts at `<commit>`.
  - New `README.md` describing how to install the plugin, its three skills and their arguments, and the release flow.

## 1.2.0 - 2026-10-16

- Changed
  - New `/changelog-writer:init [X.Y.Z]` creates a repo's first `CHANGELOG.md`: one entry (`0.0.1` or the version
    given) when the repo has no tags, otherwise one entry per release tag built from the history between tags.
  - `/changelog-writer` now asks first whether to include changes that only matter to developers (dependency
    upgrades, refactors, tooling, CI, tests).

## 1.1.0 - 2026-10-16

- Changed
  - New `/changelog-writer:tag [X.Y.Z]` creates and pushes the release tag, named after the newest changelog version
    with the existing tags' prefix (e.g. `v4.0.9`), or exactly the version given.
  - `/changelog-writer:tag` stops and asks when the tag already exists, the version is not higher than the latest tag,
    the changelog is uncommitted, or `HEAD` is unpushed or not on `master`/`main`. It never moves or force-pushes a
    tag.

## 1.0.0 - 2026-10-16

- Changed
  - `/changelog-writer [version X.Y.Z | X.Y.Z]` writes the next release entry of the repo's CHANGELOG from what
    `develop` holds that `master` does not, bumping the patch version when none is given.
  - Each line is backed by the commit bodies, merge commits, diff and code, not by commit subjects alone.
  - The entry copies the existing file's heading, sections, bullet indentation and line width.
  - Creates `CHANGELOG.md` from a default template when the repo has none and the user confirms.
  - Saves the entry without committing, pushing or tagging, and reports the commit range and the lines to confirm.
