---
name: pr-merge-workflow
description: Merge pull requests following the required bugbot review process. Use when merging PRs to ensure CI passes and bugbot approval is obtained.
---

# PR Merge Workflow

## Overview

This skill describes the required process for merging pull requests. PRs must have both passing CI and passing bugbot review on the **latest commit** before merging.

## Prerequisites

- GitHub CLI (`gh`) available in PATH
- Repository configured with CI workflow
- Bugbot (cursor[bot]) enabled on the repository

## Critical Requirements

**NEVER merge a PR without:**
1. CI passing on the latest commit
2. Bugbot review returning **LGTM** on the latest commit

## Understanding Bugbot Reviews

### IMPORTANT: Address All Feedback

Bugbot may provide feedback in different ways:

1. **LGTM** - Review complete, no issues. OK to merge.
2. **Approval with suggestions** - Bugbot says "good to merge" or "approved" but includes minor feedback. **You must address this feedback** and request another review.
3. **Changes requested** - Explicit issues that must be fixed.

**A bugbot review is only considered complete when bugbot returns "LGTM" indicating there are no remaining issues.**

Even if bugbot indicates a PR is "good to merge" or "approved", any feedback or suggestions provided should be addressed. After addressing the feedback:
1. Commit and push the changes
2. Request a new review with `@cursor please review`
3. Wait for bugbot to return **LGTM**

### Why This Matters

Bugbot's minor suggestions often catch:
- Potential edge cases
- Documentation improvements
- Code clarity issues
- Security considerations

Addressing all feedback ensures higher code quality and prevents accumulating technical debt.

## Checking CI Status

```bash
# Get recent workflow runs - check the commit SHA matches your latest
gh api repos/OWNER/REPO/actions/runs \
  --jq '.workflow_runs[:5] | .[] | "\(.id) \(.status) \(.conclusion // "running") \(.name) \(.head_sha[:7])"'
```

**Verify the commit SHA** - The `head_sha` in the output must match your latest pushed commit.

## Checking Bugbot Review

```bash
# Get PR reviews and comments - look for cursor[bot]
gh pr view PR_NUMBER --repo OWNER/REPO --json reviews,comments

# Get detailed review comments
gh api repos/OWNER/REPO/pulls/PR_NUMBER/comments
```

### Passing Review

A passing bugbot review contains **LGTM** (case insensitive):
```
"body": "LGTM — changes look good..."
```

### Non-Passing Reviews (Must Address)

These responses require action before merging:

```
"body": "Approved — ... [suggestions follow]"
"body": "Good to merge. [feedback follows]"
"body": "Changes requested: ..."
```

If bugbot provides ANY feedback or suggestions, address them and request a new review.

### Triggering a Review

```bash
gh pr comment PR_NUMBER --repo OWNER/REPO --body "@cursor please review"
```

## Handling Bugbot Feedback

When bugbot provides feedback (even minor suggestions):

1. **Read carefully** - Understand what bugbot is suggesting
2. **Implement the fix** - Make the suggested changes
3. **Commit and push**:
   ```bash
   git add -A && git commit -m "Address bugbot review feedback"
   git push
   ```
4. **Request new review**:
   ```bash
   gh pr comment PR_NUMBER --repo OWNER/REPO --body "@cursor please review"
   ```
5. **Wait for LGTM** - Only proceed when bugbot returns LGTM

### When to Skip Feedback

Only skip bugbot feedback if:
- The suggestion doesn't apply to your use case (explain why in a comment)
- The feedback is about intentional behavior (document the decision)
- Implementing would require architectural changes beyond scope (discuss with user)

## Merge Process

Once bugbot returns **LGTM** on the latest commit:

```bash
# Final verification
gh pr view PR_NUMBER --repo OWNER/REPO --json reviews,comments

# Check latest commit SHA
git rev-parse HEAD

# Verify LGTM is on latest commit

# Merge
gh pr merge PR_NUMBER --repo OWNER/REPO --merge --delete-branch
```

## Verification Checklist

Before merging, confirm ALL of the following:

- [ ] CI passes on latest commit
- [ ] Bugbot review contains **LGTM** (not just "approved" or "good to merge")
- [ ] LGTM review is on the latest commit SHA
- [ ] All bugbot feedback has been addressed

## Common Issues

### Bugbot said "good to merge" but not LGTM
Read the full comment. If there's any feedback, address it and request another review.

### Bugbot reviewed old commit
Push your changes and add `@cursor please review` to trigger a new review.

### CI passed but on wrong commit
Wait for CI to run on the latest commit, or push again to trigger a new run.

## Example Session

```bash
# 1. Check current commit
git rev-parse HEAD  # abc1234

# 2. Check bugbot review
gh pr view 24 --repo owner/repo --json comments --jq '.comments[-1].body'
# "Approved — docs look good. Minor suggestion: add example for edge case."

# 3. Bugbot gave feedback - address it
# ... make changes ...
git add -A && git commit -m "Add edge case example per bugbot feedback"
git push

# 4. Request new review
gh pr comment 24 --repo owner/repo --body "@cursor please review"

# 5. Wait and check for LGTM
sleep 60
gh pr view 24 --repo owner/repo --json comments --jq '.comments[-1].body'
# "LGTM — changes look good."

# 6. Now OK to merge
gh pr merge 24 --repo owner/repo --merge --delete-branch
```
