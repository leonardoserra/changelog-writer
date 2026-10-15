# AGENTS.md

Guidance for agents working in this repository.

## What this repository is

A Claude Code plugin that ships one skill, `changelog-writer`. There is no application code, build, lint or test
suite: the product is the skill's Markdown instructions plus the plugin manifests.

The skill writes the next release entry of a repo's CHANGELOG from what `develop` holds that `master` does not
(git-flow). When invoked (`/changelog-writer [version X.Y.Z | X.Y.Z]`) it:

1. Picks the version: the explicit one, or the latest released version with the patch bumped.
2. Finds the changelog file (`CHANGELOG*`, `CHANGES*`, `HISTORY*`, `RELEASES*`) and checks it is a real one; if there
   is none and the user confirms it is the first, creates `CHANGELOG.md` from `references/default-template.md`.
3. Reconciles the newest changelog heading with the newest release tag.
4. Fetches, resolves the git-flow branch names, and reads every commit body, the merge commits, the diff and the code
   in `origin/<master>..origin/<develop>`, backing each line with the code.
5. Filters out noise (temporary CI commits, merges alone, AI docs, formatting refactors).
6. Writes the entry copying the existing file's style exactly (heading, sections, indentation, width), one
   user-visible effect per line.
7. Saves the file without committing, pushing or tagging, and reports the range, commit count and lines to confirm.

A second skill, `tag` (`/changelog-writer:tag [X.Y.Z]`), creates and pushes the release tag once the entry is
committed. The name is the newest changelog version, using the existing tags' prefix, or exactly the version given. It
stops and asks if the tag exists, the version is not higher than the latest tag, the changelog is uncommitted, or
`HEAD` is unpushed or not on the production branch (gitflow's master setting, else `master` or `main`). It never moves or force-pushes a tag.

## Layout

- `.claude-plugin/plugin.json`: plugin manifest (name, version, description, repository).
- `.claude-plugin/marketplace.json`: single-plugin marketplace pointing at `./`, so the repo can be added directly as
  a marketplace.
- `skills/changelog-writer/SKILL.md`: the skill itself. Frontmatter `description` is what triggers it, so keep it
  specific about when to use the skill; `argument-hint` documents the arguments.
- `skills/changelog-writer/references/default-template.md`: the style used only when a repo has no changelog yet.
  `SKILL.md` refers to it by relative path.
- `skills/tag/SKILL.md`: the tagging skill. Plugin skills are namespaced `<plugin>:<skill>`, which is why its
  directory name `tag` yields `/changelog-writer:tag`.

## Editing notes

- The one-line description is duplicated in `plugin.json`, `marketplace.json` and the `SKILL.md` frontmatter (the
  latter is longer). Keep them consistent when changing what the skill does.
- Bump `version` in `plugin.json` when releasing a change to the skill.
- Markdown files wrap at 120 characters, matching the style the skill itself prescribes.
- To try the skill locally, load this directory as a plugin in Claude Code (e.g. `/plugin marketplace add <path>`,
  then install `changelog-writer`) and run `/changelog-writer` in a git-flow repo with a `develop` branch ahead of
  `master`.
