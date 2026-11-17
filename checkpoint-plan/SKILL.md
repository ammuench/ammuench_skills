---
name: checkpoint-plan
description: "Turn an implementation plan into an ordered set of checkpoints: groups of related code changes, each one reviewable, separately committable, and leaving the repo green. Then work through them one checkpoint at a time, stopping for review after each one. Use whenever the user wants to plan or carry out multi-part work (features, refactors, dependency or SDK upgrades, migrations) and mentions checkpoints, chunks, commits, steps to review, \"break this up\", or \"split the plan\". Also use when a plan has only broad phases and needs to become committable units, even if the user doesn't say \"checkpoint\"."
---

# checkpoint-plan

The user reviews and commits every logical change before more code goes on top of it. A plan made of broad phases ("Phase 1: update deps, Phase 2: fix things") fails because a phase can't be reviewed as one diff or committed as one unit. This skill turns any plan into **checkpoints**: small sets of related changes, put in order, each of which works on its own.

## Part 1: Write the checkpointed plan

### What makes a good checkpoint

- **Related.** It has one purpose that fits in one commit subject line. If you need "and" to describe it, think about splitting it.
- **Green.** After the checkpoint, the project builds and its lint, type checks, and tests pass. Nothing is half-wired. If a change can't be green yet, put a scaffold, shim, or feature flag in the checkpoint and say so.
- **Ordered by dependency.** Foundations come first (types, schema, config, shared helpers), then the code that uses them, then UI, then cleanup. A reviewer should never need a later checkpoint to understand an earlier one.
- **Reviewable.** Aim for a diff a person can read in about 10 minutes. Split big mechanical changes (renames, codemods, lockfile bumps, reformatting) into their own checkpoint so they don't hide the logic changes.
- **Tested in place.** A checkpoint's tests go in that same checkpoint. Don't save all the tests for a final checkpoint.

### Steps

1. Read the plan or request. Look at the code it touches so the file paths are real, not guessed.
2. Find the project's checks (build, lint, typecheck, test commands) from package.json, Makefile, CI config, etc. Every checkpoint uses these.
3. Write the checkpoints using the template below. Usually 3 to 8. If you have more than 10, group the related ones together. If you have only 1, say the work doesn't need checkpointing.
4. Save the plan where the user or the harness expects it (the plan file in plan mode, otherwise show it in chat). Then stop and ask the user to approve it before you write any code.

### Template

```markdown
# <Plan title>

<One or two sentences: the goal and why it is split this way.>

## Checkpoint 1: <short name>
**Goal:** <what works after this checkpoint, in user or developer terms>
**Files:**
- `path/to/file.ts`: <what changes>
**Tests:** <new or updated tests, plus the commands to run>
**Green check:** `<exact commands>`
**Commit message:** `<type(scope): summary>`

## Checkpoint 2: ...

## Risks / open questions
- <anything that might force a re-order or a split>
```

## Part 2: Do the checkpoints

Work on **one checkpoint per turn**:

1. Say which checkpoint you are starting ("Checkpoint 2 of 5: …").
2. Change only what that checkpoint lists. If you find you need something outside it, stop and tell the user. Don't let it slip in.
3. Run the checkpoint's green check. Fix any failures before you report.
4. Report:
   - What now works, in concrete terms.
   - A short diff summary (files and what changed). If an edit mostly re-indents a block, list the real logic changes and point to `git diff -w`. A re-indented block otherwise looks like a full rewrite in the approval diff.
   - Check results (pass/fail, plus the first failing line if any).
   - The suggested commit message.
   - What the next checkpoint is.
5. **Stop.** Don't commit, and don't start the next checkpoint. The user reviews and commits, then says "proceed" (or something like it).

If review changes the plan, update the remaining checkpoints before you go on, and show what changed.

Don't commit, push, or open PRs unless the user tells you to. The user does those steps.
