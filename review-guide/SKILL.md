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
6. Render both artifacts directly in chat using the Output format below. Short sentences, bullets/lists, ASD-STE100 plain language. No paragraphs unless necessary. Do not create files unless the user explicitly requests file output.

## Output format

Findings are a numbered list, most severe first. Each finding ends with a **Sample comment**: a ready-to-paste PR comment in a blockquote.

```
N. <Severity> — <short title>
   • <path>:<line-or-range>
   • <What the code does now, and why it is wrong.>

   Sample comment:
   > <label> (<decoration>): <comment text>
```

Sample comment rules:

- Every finding gets one. No exceptions, including praise and questions.
- It is a blockquote (`>`), so the engineer can copy it without picking it out of the surrounding bullets.
- It starts with a Conventional Comments label: `praise:`, `nitpick:`, `suggestion:`, `issue:`, `question:`. Add `(blocking)`, `(non-blocking)`, or `(if-minor)` when merge impact is not obvious.
- Write it to the PR author in second person. One paragraph, 1-3 sentences.
- Say the fix, not just the problem. End with the concrete ask ("please map X separately and add tests").
- No markdown headers, no nested bullets, no line breaks inside it. It must survive a paste into a GitHub comment box.
- Do not repeat the file path inside it. The bullet above already has it.

Worked example:

```
1. Blocking — incorrect payment method mapping
   • apps/ratehawk/src/transform/toConfirmRateResponse.ts:104-107
   • The code maps every prepaid rate to CARD. ETG defines `deposit` as payment
     from the client's ETG deposit, which the gateway defines as BALANCE.
   • Required mapping: deposit → BALANCE, now → CARD, hotel → USER_CARD_FORWARDING

   Sample comment:
   > issue (blocking): Derive payment methods from `payment_options.payment_types`. A `deposit` rate uses the ETG supplier balance, so it must map to BALANCE, not CARD. Please map `deposit`, `now`, and `hotel` separately and add tests for each type.

2. Test gap — reprice acceptance path
   • apps/ratehawk/src/transform/__tests__/toConfirmRateResponse.test.ts
   • No test covers a prebook price that differs from the find-time price.

   Sample comment:
   > suggestion: Add explicit reprice coverage. Return a different `show_amount` from prebook and verify that the confirmed offer contains the new price and refreshed `book_hash`.
```
