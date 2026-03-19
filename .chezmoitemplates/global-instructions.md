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
3. Evaluate each **Critical** or **High** severity issue per the **Review Feedback Evaluation** guidelines below. Fix verified issues before committing.
4. Once all critical/high issues are resolved (or if none were found), proceed with the commit.

Never skip the review step. Even small changes can introduce subtle bugs or style regressions.

## Review Feedback Evaluation

When processing review feedback — whether from a code-reviewer sub-agent, a human PR reviewer, or an automated bot — never blindly trust or apply suggestions. Independently verify every piece of feedback before acting on it.

### Verification Process

1. **Read the relevant code** yourself to understand the full context around the flagged area.
2. **Assess technical correctness** — is the reviewer's claim actually true? Research further if needed: consult language/framework documentation, check API references, or look up specification behavior to confirm or refute the claim.
3. **Check project conventions** — does the suggestion align with patterns already established in this codebase, or does it impose a foreign style?
4. **Consider missing context** — the reviewer may not be aware of intentional design decisions, broader system constraints, or upstream requirements.

### Decision Framework

- **Apply** the fix if independently verified as correct and beneficial.
- **Disregard** if the suggestion is wrong, inapplicable, or based on a misunderstanding of the code.
- **Explain your reasoning** when disagreeing: respond in PR comments for remote reviews, or state your rationale in your response to the user for sub-agent reviews.
- **When verification is inconclusive** — if you cannot determine whether the feedback is correct (e.g., subtle runtime behavior, ambiguous specs), err on the side of caution: apply the fix if it is low-risk, or flag it for human review if the change is significant.
