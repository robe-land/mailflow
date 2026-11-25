---
name: feature-lifecycle
description: "Manage a Mailflow feature from synchronized develop through PR merge, cleanup, and the next feature branch."
---

# Mailflow feature lifecycle

Use this procedure when asked to start or carry a Mailflow feature through
development, pull-request review, merge, branch cleanup, or preparation of
the next feature. Respect any narrower task instructions; a request to
document or plan the lifecycle does not authorize performing its GitHub or
Git operations.

When the user authorizes completing the full feature lifecycle, that
authorization covers routine, task-scoped commits, publishing the feature
branch, creating the PR, monitoring required CI, merging once all required CI,
branch, creating the PR, monitoring required CI, merging after explicit owner
approval of the exact candidate and successful required CI, and normal
post-merge cleanup and synchronization. This is a single-developer policy;
it does not require an independent GitHub review. Stage and commit only files
in the authorized task scope. Do not request redundant confirmation for each
routine Git or GitHub operation. Stop for user intervention when an unresolved
review conversation, safety issue, or merge conflict remains, or when an
active GitHub protection prevents integration. Never change or bypass a
server-side protection to satisfy the local policy. An explicit instruction
to stop at a particular lifecycle boundary takes precedence over broader
lifecycle authorization.

## Begin from synchronized develop

1. Inspect the current branch and worktree before switching. Preserve
   unrelated untracked files. Do not automatically stash, reset, clean, or
   overwrite existing work. If unrelated tracked changes would interfere with
   the lifecycle, prefer a separate clean worktree where practical rather
   than carrying them across a switch. If no safe worktree is practical, stop
   and ask how to proceed.
2. Fetch `origin`, switch to local `develop` only when safe, then synchronize
   it with `origin/develop` using a fast-forward-only operation. If local
   `develop` cannot fast-forward, stop; do not reset it or merge divergent
   history automatically. The normal sequence is `git fetch origin`, `git
   switch develop`, then `git merge --ff-only origin/develop`.
3. Create a descriptively named feature branch from synchronized `develop`,
   using `feature/<short-kebab-case-scope>`. If the feature scope or name is
   genuinely unclear, ask rather than inventing one. Create it with
   `git switch -c feature/<short-kebab-case-scope>`.

## Develop and open a milestone PR

1. Make small, meaningful commits as work progresses when the user has
   authorized committing. Stage only task-related files; do not amend commits
   unless explicitly asked. Never force-push a protected branch.
2. Run `npm run verify` before milestone integration. It is the canonical
   non-mutating local check; wait for all required CI checks, including any
   required PostgreSQL CI, to finish and resolve failures before merging.
   Successful required PostgreSQL CI is sufficient by default. Local
   `npm run verify:postgres` runs disposable containers and requires separate
   explicit authorization; full-lifecycle authorization does not include
   that local execution.
3. When authorized to publish the work, push the feature branch and create a
   PR targeting `develop` (for example, `git push -u origin <feature-branch>`
   and `gh pr create --base develop --head <feature-branch>`). Confirm the PR
   base and head are correct.

## Review and merge

1. Wait for every required CI check to complete successfully (for example,
   `gh pr checks <number> --watch`). Review the PR diff and scope, merge
   eligibility, and all review conversations. Do not merge while a required
   conversation or safety concern is unresolved, while checks are failing, or
   while conflicts remain.
2. Obtain explicit owner approval for the exact candidate and verify all
   required CI checks. The local single-developer policy does not override
   GitHub branch protections or rulesets. If server-side policy requires an
   independent approving review, report the exact requirement and stop; do not
   request an independent reviewer, change the setting, or bypass it.
3. Merge only after confirming owner approval, successful required CI, and
   GitHub merge eligibility. Recent Mailflow PRs use merge commits; use
   `gh pr merge <number> --merge` unless the repository's established strategy
   changes. Merge into `develop`, never automatically into `master`.
4. Confirm the PR reports a successful merged state before starting cleanup.

## Clean up and prepare the next feature

1. Inspect whether GitHub's **Automatically delete head branches** setting is
   enabled, but do not change repository settings as part of this procedure.
   If disabled, report that and recommend enabling it. Never assume automatic
   deletion occurred. Verify that the remote feature branch is absent after
   merge (for example, with
   `git ls-remote --heads origin refs/heads/<feature-branch>`). If it remains,
   compare its tip with the merged PR's head SHA and confirm it contains no
   commits outside the PR. Only then delete it with
   `git push origin --delete <feature-branch>` and verify it is absent. Stop
   if it has unmerged work; never delete an unmerged branch without explicit
   approval.
2. With a safe worktree, fetch `origin`, switch to `develop`, and update it
   using fast-forward-only operations (for example,
   `git fetch origin`, `git switch develop`, then
   `git merge --ff-only origin/develop`). Stop if it cannot fast-forward.
3. Delete the merged local feature branch with Git's safe branch deletion,
   `git branch -d <feature-branch>`.
   If Git refuses because commits are not recognized as merged, stop and
   investigate; do not force-delete or use destructive Git cleanup commands.
4. Create the next descriptively named feature branch from synchronized
   `develop` only when a subsequent scope is already clear and authorized.
   If no next scope was provided, finish on synchronized `develop` without
   asking for one. Create an authorized next branch with
   `git switch -c feature/<next-short-kebab-case-scope>`. Verify the remote
   deletion and local synchronization, and verify the new branch base when
   creating one, before reporting completion.

## Safety stops

Never force-push protected branches, delete unmerged branches without explicit
approval, or use destructive Git cleanup commands. Preserve unrelated
untracked files. Stop rather than guessing when fast-forwarding fails, a
feature branch has unmerged commits, PR checks or safety concerns are
unresolved, or the next feature's scope is unclear.
