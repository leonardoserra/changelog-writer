---
name: init
description: Creates a repo's first CHANGELOG.md. With no release tags it writes a single entry, 0.0.1 or the version given; with existing tags it writes one entry per tag from the git history. Use it whenever the user types /changelog-writer:init, or asks to create, start or bootstrap a changelog for a repo that has none.
argument-hint: "[X.Y.Z]"
---

# init

Create `CHANGELOG.md` at the repository root for a repo that has no changelog yet.

## 1. Arguments

- `/changelog-writer:init`: no version given. With no tags, the single entry is `0.0.1`.
- `/changelog-writer:init 1.2.3`: with no tags, the single entry is `1.2.3`. With tags, see section 4.
- Always ask, before reading the history: "Include changes that only matter to developers (dependency upgrades,
  refactors, tooling, CI, tests)?" The answer applies to every entry, as in section 5 of
  `../changelog-writer/SKILL.md`.

## 2. Check there is no changelog

1. Look at the repository root for `CHANGELOG*`, `CHANGES*`, `HISTORY*` and `RELEASES*`, matching case-insensitively.
2. If a real changelog exists (dated version headings, newest first), stop. Tell the user to use `/changelog-writer`
   for the next entry. Never overwrite or rewrite an existing changelog.
3. If a file has one of these names but is not a changelog, ask the user before creating `CHANGELOG.md` next to it.

## 3. Find the release tags

1. Run `git fetch --all --prune --tags` first.
2. List the release tags, oldest first, with their dates:
   `git for-each-ref --sort=v:refname --format='%(refname:short) %(creatordate:short)' refs/tags`.
   The date is the tagging date for annotated tags and the commit date for lightweight ones.
3. Keep only tags that name a version (`1.2.3` or `v1.2.3`). Tell the user which other tags you skipped.
4. With more than 20 release tags, ask whether to cover all of them or only the newest ones.

## 4. Write the entries

Start from `../changelog-writer/references/default-template.md` (relative to this skill's directory): copy its file
header and follow its conventions for headings, sections, bullets and width. Replace the example entry.

### No tags

Write one entry, `## <version> - <today>`, describing what the project does at `HEAD` of the production branch
(`git config gitflow.branch.master`, else `master` or `main`). Read the README, the commit history
(`git log --no-merges --reverse --format='%h %s%n%b'`) and the code. Use a single `- Changed` section, one line per
main capability a user or operator would notice, and keep it short: this is the first release, not a history.

### With tags

Write one entry per release tag, newest first. The version is the tag name without a `v` prefix. The date is the tag
date from section 3.

- The range of a tag is `<previous tag>..<tag>`. The oldest tag covers everything up to it (`<tag>` alone), and its
  entry may use a single `Changed` section listing its main capabilities, like the no-tag case above.
- For each range, read the commit bodies (`git log --no-merges --reverse --format='%h %s%n%b' <range>`), the merge
  commits (`git log --merges --oneline <range>`) and the diff stat (`git diff --stat <range>`). Read the code where a
  message is unclear or the diff touches user-facing behaviour.
- Decide what goes in and word each line following sections 5 and 6 of `../changelog-writer/SKILL.md`.
- If the user declined developer changes and a range has nothing a user would notice, its entry gets a single line,
  `- No user-facing changes.`, so every tag has an entry.
- If a version argument was given and `HEAD` has commits after the newest tag, ask whether to add a top entry with
  that version for `<newest tag>..HEAD`. Commits after the newest tag are otherwise left out.

## 5. Save, do not commit

- Write `CHANGELOG.md`. Do not commit, push or tag.
- Report to the user:
  - the entries written, with their version, date and number of commits read;
  - any skipped tags, and how many commits after the newest tag were left out;
  - lines that could not be verified from the code.
- If an entry has no tag yet (the no-tag case, or a version for commits after the newest tag), say that
  `/changelog-writer:tag` creates it once the changelog is committed and pushed.
