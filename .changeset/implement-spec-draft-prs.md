---
"mattpocock-skills": patch
---

`implement-spec` opens a lane's PR as a draft when it is not safe to merge yet (stacked on another lane's unmerged PR, or its verification could not run), names what it waits on, and marks it ready with `gh pr ready` once that clears. The final table shows draft or ready per lane. A new "After the run" section says the final table ends the unattended run, so later messages are followed one at a time instead of resuming the run's old plan. Implementers now run the repo's database and integration suites for their ticket instead of being limited to unit checks, and lane verification no longer runs one lane at a time on a single database: it starts and stops the database the way the repo's AGENTS.md says, and is skipped in a repo with no database or integration checks.
