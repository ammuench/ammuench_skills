---
name: review-guide
description: Review a pull request or branch diff for bugs, risks, and gaps, and provide a concise walkthrough guide for engineer triage. Use when user asks for code review, review PR, review branch, or triage changes.
---

# review-guide

## When to use

Use when the user asks for a code review of a remote pull request link or local branch changes versus the default branch.

## Instructions

1. Get the diff:
   - If PR link provided: fetch remote diff + existing PR comments. If fetch fails (auth, network, bad link), STOP and ask user to unblock.
   - If no link: diff current branch against default branch (usually `main`, sometimes `develop` or `master`). Ask if unclear.
2. If the PR references a ticket (Jira, Linear, GitHub issue), fetch it for context. If none, proceed.
3. Review only changed lines plus direct impacts. For each finding, give `filepath:line_number`. Use Conventional Comments format (see https://conventionalcomments.org/): labels `praise:`, `nitpick:`, `suggestion:`, `issue:`, `question:` with decorations `(blocking)`, `(non-blocking)`, `(if-minor)` to signal merge impact. Cover bugs, logic, architecture, testing gaps, AC mismatch, security, and so on.
4. Verdict: `Approve` = non-blocking comments only. `Comment` = needs clarification but don't block for churn. `Request Changes` = blocking bug or harmful pattern. When in doubt, favor `Request Changes`.
5. Provide a guide: group related files in logical read order (feature before tests), explain what each group does and what to focus on.
6. Render both artifacts directly in chat concisely: short sentences, bullets/lists, ASD-STE100 plain language. No paragraphs unless necessary. Do not create files unless the user explicitly requests file output.
