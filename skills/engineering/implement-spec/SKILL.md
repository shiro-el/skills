---
name: implement-spec
description: "Implement one or more issues end to end, each as its own lane: built, verified, reviewed, fixed, and opened as a PR."
disable-model-invocation: true
---

<!-- fork -->

You have been given one or more issues: a spec with tickets, a single ticket, or a large issue that has no tickets yet.

The issue tracker should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.

Each issue becomes a **lane**: its own **integration branch**, built, verified, reviewed and fixed, ending in its own PR. The user typed one command and is not watching. The run is done when every lane has a reviewed PR open (or is reported as blocked), every review finding is fixed or listed in its PR body, and nothing the run started (worktree, container, local database) is left running.

A lane's tickets are not a list of steps. They are a **task graph** with blocking relationships between them, so each lane always has a **frontier** of tickets ready to be grabbed. Work every lane's frontier at once.

Communication to and from subagents should be sparse. Communicate primarily through **context pointers**: to the issues, tickets, research notes, and previous commits. Don't duplicate information already available via pointers.

**Implementer subagents** should be run in the background where possible for maximum concurrency.

## Steps

1. **Preflight.** Run the repo's cleanup command if its AGENTS.md names one, so leftovers from earlier runs are gone. Fetch every issue and give each one a lane:
   - an issue with sub-issues: its tickets and their blocking edges are the lane's task graph;
   - an issue one implementer can land in a single context window: a graph of one ticket, the issue itself;
   - a larger issue with no sub-issues: split it into tracer-bullet vertical slices, create them as sub-issues with blocking edges, and comment `Assumption: split into <tickets>, because <why>` on the issue.

   Cut each lane's integration branch from the base branch: the one PRs target according to the repo's AGENTS.md or tracker doc, else the default branch.

2. (optional) Use an **exploration subagent** to conduct any exploration required by the tickets - relevant codebase files or external documentation. It saves its markdown notes in a directory outside the repo, accessible by all future subagents, so **implementer subagents** focus on implementation rather than exploration.

3. **Build.** Use **implementer subagents** to implement each frontier ticket, across all lanes, each in its own worktree on its own branch. Each implementer subagent:
   - confirms its worktree is based on its lane's integration branch before starting, and resets onto it if not;
   - calls the Skill tool with `tdd` to build the ticket;
   - runs the checks the repo's AGENTS.md asks for its ticket, database and integration suites included, and stops anything it started before reporting done;
   - never stops to ask the user. On an ambiguous point it takes the safest reversible reading, comments `Assumption: <what, and why>` on its ticket, and carries on;
   - merges its lane's integration branch tip into its own branch before reporting done.

4. Once an **implementer subagent** completes, merge its work to its lane's integration branch with a **merger subagent**. If this changes the lane's **frontier**, kick off more implementer subagents straight away.

5. **Verify.** When every ticket in a lane has merged, queue the lane. Run the repo's database and integration verification on the lane's integration branch, starting and stopping the database the way the repo's AGENTS.md says; other lanes keep building meanwhile. A repo with no database or integration checks skips this step. When a test fails, re-run that test alone once: if it passes, record it as flaky for the lane's PR body; if it fails again, it is a finding for step 7.

6. **Review.** For each verified lane, call the Skill tool with `code-review`. The fixed point is the merge-base of the lane's integration branch with the base branch. Each lane gets exactly one review.

7. **Fix.** Give each lane's review report to a single **implementer subagent**, which fixes every finding inside the lane's issue. A finding that belongs to other work becomes a new follow-up issue, linked to the lane's issue. Then re-check the lane with focused checks only: back through step 5 if the fixes touched the database schema or migrations, otherwise the affected test files. A fix never triggers a second review.

8. **Cross-lane checks.** Update each lane onto the latest base branch tip the way the repo's history rules say. Then check what parallel lanes can break between them:
   - every new migration version is unique across all lanes and the base branch; renumber the later lane's with the repo's migration tool and regenerate what it generates;
   - files touched by more than one lane are named, with a suggested merge order, in each of those PRs' bodies.

9. **PR.** If the issue tracker closes work through PRs, or the user asks for one, open each lane's PR marked as closing its issue and tickets. A lane that is safe to merge now opens ready for review. A lane that is not opens as a **draft**, for example one stacked on another lane's unmerged PR or one whose step 5 verification has not run. Its body says what it waits on. Once that clears, mark it ready with `gh pr ready <number>`. Call the Skill tool with `pr` for the body. Besides the `pr` template, the body carries:
   - `## Assumptions`: every `Assumption:` comment from the lane's issue and tickets, so the user checks them once, at the end;
   - each finding left unfixed, with the reason, and each follow-up issue created in step 7;
   - each flaky test from step 5.

   Otherwise, resolve each ticket the way the issue tracker closes work, and report the integration branch.

10. **Clean up and report.** Remove every implementer subagent worktree, stop every database stack the run started (releasing it the way the repo says), and run the repo's cleanup command again. End with one table, a row per lane: issue, PR or branch, draft or ready (and what a draft waits on), verification result, findings fixed and left, assumptions, flaky tests.

## Unattended

Never wait for the user. A lane that cannot proceed without a decision no `Assumption:` can settle is reported as blocked in the final table and left there; every other lane keeps going.

## After the run

The final table ends the unattended run. From then on the user drives: do only what each message asks, and leave the run's remaining plan behind.
