---
name: tag
description: Creates and pushes the git tag for a release, named after the newest version in the repo's CHANGELOG or after a version the user gives. Use it whenever the user types /changelog-writer:tag, or asks to tag the release, push the release tag, or "tag the version from the changelog".
argument-hint: "[X.Y.Z]"
---

# tag

Create the release tag and push it to `origin`. The tag name comes from the changelog, unless the user gives one.

## 1. Arguments

- `/changelog-writer:tag`: no version given. Use the version of the newest entry in the changelog.
- `/changelog-writer:tag 1.2.3`: use exactly `1.2.3` as the tag name, prefix included or not, as typed.

## 2. Find the version

1. Run `git fetch --all --prune --tags` first, so the checks below see the remote state.
2. Find the changelog at the repository root: `CHANGELOG*`, `CHANGES*`, `HISTORY*` or `RELEASES*`, matching
   case-insensitively. If there is none or more than one, ask the user which file to read.
3. Read the newest version heading, for example `## 4.0.9 - 2026-10-16` gives `4.0.9`.
4. With no argument, name the tag after that version, copying the existing tags' prefix: if they look like `v4.0.8`,
   the tag is `v4.0.9`. Read them with `git tag --sort=-v:refname | head -5`.
5. With an argument that differs from the newest changelog version, say so and ask before going on. The changelog may
   be missing the entry.

## 3. Check before tagging

Stop and ask the user when any of these holds. Never move, overwrite or delete a tag.

- The tag already exists, locally (`git tag -l <name>`) or on the remote (`git ls-remote --tags origin <name>`).
- The version is not higher than the latest existing tag.
- The changelog has uncommitted changes (`git status --porcelain -- <file>`), so the tagged commit would not hold the
  entry.
- `HEAD` is not pushed: `git branch -r --contains HEAD` lists no remote branch.
- `HEAD` is not on the production branch: `git merge-base --is-ancestor HEAD origin/<master>` fails. `<master>` is
  `git config gitflow.branch.master` when set; otherwise it is `master` or `main`, whichever exists on `origin`
  (`git ls-remote --heads origin master main`). If both exist, `HEAD` on either one passes. If neither exists, ask
  the user for the branch name. A tag points at a commit, not a branch. A commit made on
  a release or feature branch never reaches master if that branch is squash- or rebase-merged, and then
  `git describe --tags origin/<master>` no longer finds the tag.

## 4. Create and push

1. Copy the kind of the existing tags: `git cat-file -t <latest tag>` prints `tag` for annotated tags, which are made
   with `git tag -a <name> -m "<name>"`, and `commit` for lightweight ones, made with `git tag <name>`. With no tags
   yet, make an annotated tag.
2. Tag `HEAD`.
3. Push only that tag: `git push origin <name>`. Do not use `--tags`.
4. If the push fails, report the error and leave the local tag in place. Do not retry with `--force`.

Report the tag name, the commit it points at (short SHA and subject) and the branch it was made from. If the repo uses
git-flow, remind the user that `git flow release finish` creates its own tag, and needs `-n` to skip it now that this
tag exists.
