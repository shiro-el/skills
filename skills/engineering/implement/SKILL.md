---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets.

If the user passes a ticket reference, fetch it from the issue tracker and state its title before starting. If the reference is ambiguous, ask.

Call the Skill tool with "tdd" where possible, at pre-agreed seams.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

<!-- fork -->
Once done, close out in order:

1. Commit your work to the current branch.
2. Call the Skill tool with "code-review". The fixed point is the merge-base with the branch this work merges into (the PR base the user or the issue tracker named, else the default branch), so the review sees every commit of the run and needs no answer from the user.
3. Fix every finding the review raises, re-run typechecking and the affected test files, and commit.
4. If the work goes up as a PR, open it now, after the review. Its body lists each finding left unfixed, with the reason.

The run is done when every review finding is fixed in a commit or listed in the PR body with its reason.
