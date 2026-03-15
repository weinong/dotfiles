# Global Agent Instructions

## Branch Hygiene

Before making any code changes, always start from a clean state:

1. Determine the default branch (`main` or `master`) by inspecting the repo (e.g., `git symbolic-ref refs/remotes/origin/HEAD` or checking which branch exists).
2. Fetch the latest from the remote: `git fetch origin`
3. Check out the default branch: `git checkout <default-branch>`
4. Reset to the latest remote state: `git reset --hard origin/<default-branch>`
5. Create a new feature branch from there: `git checkout -b <descriptive-branch-name>`

Never make changes on a stale or dirty branch. Always ensure your working tree is clean and your branch is up to date with the remote before writing any code.

## Pre-Commit Code Review

Before creating any git commit, always run a code review on the pending changes:

1. Generate the diff of all staged and unstaged changes that will be committed:
   `git diff HEAD`
2. Invoke the `code-reviewer` sub-agent via the Task tool to review the diff.
3. If the review identifies any **Critical** or **High** severity issues, fix them before committing.
4. Once all critical/high issues are resolved (or if none were found), proceed with the commit.

Never skip the review step. Even small changes can introduce subtle bugs or style regressions.
