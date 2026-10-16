# changelog-writer

A Claude Code plugin that writes the next release entry of a repo's CHANGELOG from what `develop` holds that
`master` does not (git-flow). It reads every commit message, the merge commits, the diff and the code, and says
plainly what changed for users, one effect per line, in the style the changelog already uses.

## Install

Add this repository as a marketplace and install the plugin:

```
/plugin marketplace add leonardoserra/changelog-writer
/plugin install changelog-writer@changelog-writer
```

## Skills

### `/changelog-writer [version X.Y.Z | X.Y.Z]`

Writes the entry for the release `develop` is about to ship.

- Version: the one given, or the latest released version with the patch bumped (`4.0.8` becomes `4.0.9`).
- Asks first whether to include changes that only matter to developers (dependency upgrades, refactors, tooling, CI,
  tests).
- Finds the changelog (`CHANGELOG*`, `CHANGES*`, `HISTORY*`, `RELEASES*`) and checks its newest heading against the
  newest release tag. If there is none and you confirm this is the first, creates `CHANGELOG.md` from a default
  template.
- Fetches, resolves the git-flow branch names and reads `origin/<master>..origin/<develop>`, backing each line with
  the code rather than commit subjects alone. Temporary CI commits, bare merges, AI docs and formatting refactors are
  left out.
- Copies the existing file's heading, sections, indentation and line width exactly.
- Saves the file without committing, pushing or tagging, and reports the range, commit count and lines to confirm.

### `/changelog-writer:init [X.Y.Z]`

Creates the first `CHANGELOG.md` for a repo that has none.

- With no release tags: a single entry, `0.0.1` or the version given.
- With tags: one entry per tag, built from the history between consecutive tags with the same rules as the main skill.
- Never overwrites an existing changelog and does not commit.

### `/changelog-writer:tag [X.Y.Z]`

Creates and pushes the release tag once the entry is committed.

- Name: the newest changelog version with the existing tags' prefix (e.g. `v4.0.9`), or exactly the version given.
- Stops and asks if the tag already exists, the version is not higher than the latest tag, the changelog is
  uncommitted, or `HEAD` is unpushed or not on the production branch (gitflow's master setting, else `master` or
  `main`).
- Never moves or force-pushes a tag.

## Typical release flow

1. `/changelog-writer` on `develop` (or the release branch), review and commit the entry.
2. Finish the git-flow release so the commit lands on `master`, and push.
3. `/changelog-writer:tag` to tag and push the release.

## Layout

- `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`: plugin and marketplace manifests.
- `skills/changelog-writer/SKILL.md`: the main skill; `references/default-template.md` is the style for new
  changelogs.
- `skills/init/SKILL.md`: the bootstrap skill.
- `skills/tag/SKILL.md`: the tagging skill.

See [CHANGELOG.md](CHANGELOG.md) for release history.
