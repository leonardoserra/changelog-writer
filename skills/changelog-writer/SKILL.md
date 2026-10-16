---
name: changelog-writer
description: Writes the next release entry of a repo's CHANGELOG from what develop holds that master does not. Reads every commit message, the diff and the code to say plainly what changed for users. Use it whenever the user types /changelog-writer, asks for a changelog or release-notes entry, wants "the changelog for the release", or is preparing a git-flow release branch, even without saying "changelog".
argument-hint: "[version X.Y.Z | X.Y.Z] [from <commit>]"
---

# changelog-writer

Write the changelog entry for the release that `develop` is about to ship. The entry tells a reader what is different
in this version: what users and operators will notice, explained in one short line each. A list of commit subjects is
not an entry.

## 1. Arguments

- `/changelog-writer`: no version given. Bump the **patch** of the latest released version (`4.0.8` becomes `4.0.9`).
- `/changelog-writer 1.2.3` or `/changelog-writer version 1.2.3`: use exactly `1.2.3`.
- If the explicit version is not higher than the latest released one, say so and ask before writing.
- `/changelog-writer from <commit>` (alone or after a version, e.g. `/changelog-writer 1.2.3 from a1b2c3d`): build
  the entry from `<commit>` onwards instead of from where `develop` leaves `master`. See section 4.
- Always ask, before reading the history: "Include changes that only matter to developers (dependency upgrades,
  refactors, tooling, CI, tests)?" The answer decides section 5.

## 2. Find the changelog file

1. Look at the repository root for `CHANGELOG*`, `CHANGES*`, `HISTORY*` and `RELEASES*`, matching case-insensitively.
2. A real changelog has dated version headings, newest first, each followed by a list of changes. A file with a
   different purpose that happens to have this name is not one. Read the top of the file to decide.
3. If none is found, or the candidate is not a real changelog, ask the user for the filename. If the user says this is
   the first changelog, create `CHANGELOG.md` from `references/default-template.md` and write the entry in that style.

## 3. Find the latest version

- Read the newest version heading in the changelog.
- Read the newest release tag: `git tag --sort=-v:refname | head -5`, and `git describe --tags --abbrev=0 origin/master`.
- If the two disagree, for example a tag exists with no changelog entry, tell the user which one you used and why.
- A missing tag entry is not invented. Only the requested version is written.

## 4. Find what is new

1. Run `git fetch --all --prune` first, so the comparison is against the remote state.
2. Find the branch names. Read `git config gitflow.branch.master` and `git config gitflow.branch.develop`. If they are
   unset, use `master`/`main` and `develop`.
3. Compare `origin/<master>..origin/<develop>`.
   - If local `develop` differs from `origin/develop`, say so and ask which one to use.
   - The first commit of the release is the oldest one in that range. `git merge-base origin/<master> origin/<develop>`
     shows where it starts.
   - With `from <commit>`, the range is `<commit>^..origin/<develop>` instead, so `<commit>` itself is included (use
     `origin/<develop>` alone if `<commit>` is the root commit). First check it exists
     (`git rev-parse --verify <commit>^{commit}`) and is on `develop`
     (`git merge-base --is-ancestor <commit> origin/<develop>`); if not, say so and ask for another commit. If it is
     older than the merge base, tell the user the range includes commits already on `master`.
4. List the commits with their full bodies: `git log --no-merges --reverse --format='%h %s%n%b' <range>`.
   - The bodies hold the reasons, the before and after, and the ticket keys. Read every one.
   - Read the merge commits too (`git log --merges <range>`). They show which feature branches and tickets the release
     contains.
5. Read the diff, `git diff --stat <range>`, then the code under the app, lib and config directories.
   - Commit messages can be wrong, stale or overstated. Back each line with the code.
   - Check the user-facing effect: which endpoint, screen, report or job changes.
6. Find the behaviour that changes without a commit saying so: enum values, new response fields, status codes, renamed
   labels. These are the lines a release reader most needs to see.

## 5. Decide what goes in

Include everything a user, an API client or an operator would notice. Leave out:

- Temporary CI commits (for example, adding a branch to a job's `only:` and removing it again).
- Merge commits on their own.
- Agent and AI documentation (`CLAUDE.md`, local notes).
- Formatting-only refactors.
- Mention dead code removal only when it removes something reachable by name: a route, an endpoint, a public method, a
  gem.

If the user said yes to developer changes (section 1), also include what changed for developers: dependency
upgrades, refactors, tooling, CI and tests. The exclusions above still apply.

## 6. Write the entry in the file's own style

Read the newest 3-5 entries and copy their conventions exactly:

- the heading format, for example `## 4.0.8 - 2026-10-02`, with today's date;
- the section names and their order, for example `Changed`, `Fix`, `Removed`, `Tests`;
- the bullet indentation, the line width (wrap continuation lines under the bullet text), and how backticks and quotes
  are used.

If an older heading breaks the file's own format, for example a missing `## `, point it out and let the user decide.
Do not fix it silently.

Write each line so it explains the change on its own. Patterns that read well:

- **"X no longer Y"** for fixes: "Radio campaign creation no longer answers 500 ("501 Invalid RCPT TO address
  provided") on a malformed spot provider e-mail such as `<name@domain.it`."
- **"X now Z" or "A → B"** for changes: "Rails 5.2.8.1 → 8.1.3.1 with `load_defaults 8.1`."
- **The exact error string, in quotes,** when a fix stops an error people saw, so they can match it to their reports.
- **Before and after** when behaviour changes: "...shows one block per broadcaster and day, with its name. Before, all
  broadcasters were summed into one block with an empty "Emittente"."
- **One concrete example** when a rule is abstract: "(e.g. `RADIO 24` vs `RADIO24`)".
- The screen or endpoint as the user knows it ("Radio campaign page", "Public API `me/dates`"), not the class name.
  Use a class or file name only when operators act on it (a rake task, a cron, an env var).
- No ticket keys, commit SHAs or `file:line` in the entry, unless the existing file uses them.
- One line per effect. Merge commits that describe the same fix into one line. Split a commit that ships two
  unrelated effects into two lines.
- Put visible behaviour changes under `Changed`, even when they come from a fix commit. Readers scan that section for
  "what will my users see differently".

## 7. Save, do not commit

- Insert the entry above the newest one. Keep the file header and every older entry byte-identical.
- Do not commit, push or tag. The user copies the file into the release branch.
- Report to the user:
  - the version, the commit range (`<first sha>..<last sha>`) and the number of commits read, and the `from` commit
    when one was given;
  - the lines that describe a visible change, so the user can confirm each one before release;
  - anything you left out on purpose, and anything that could not be verified from the code.
