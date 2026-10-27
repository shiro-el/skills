## What it does

`improve-codebase-architecture` surveys a codebase for **deepening opportunities**. These are places where a shallow module (an interface nearly as complex as the thing it hides) could become a deep one. Before you see any of them, it has each candidate debated: one subagent argues the case, an independent opponent (Codex when its CLI is installed, otherwise a fresh Claude subagent) checks the evidence and attacks it. The survivors go into a self-contained HTML report, and the candidates you pick become issues on your tracker, ready for [implement-spec](https://aihero.dev/skills-implement-spec).

It never changes the code. The whole run produces debate notes and one HTML file in your OS temp directory, plus the issues you asked for. The refactor happens later, in a separate [session](https://www.aihero.dev/ai-coding-dictionary/session), when you run `/implement-spec` on those issues. This makes it a survey, not a refactoring tool, so you can run it on a codebase you are not ready to touch yet.

Two filters stop the report from becoming generic cleanup advice. First, every candidate must pass the **deletion test**: if you removed this module, would its complexity move behind a smaller interface, or spread across the callers? Only the first case gets a card. Second, unless you point it at a specific area, it reads recent commit history first and focuses the scan on paths that change often. A deepening in code that nobody touches is a refactor that never pays back.

## When to reach for it

You invoke this by typing `/improve-codebase-architecture`; the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) will not reach for it on its own.

It is not a step in the main build loop. You run it periodically to queue up more work that improves the codebase. People use it in four situations:

| Situation | How it is used |
| --- | --- |
| Routine upkeep | Run it every few days, or when you have a spare moment, so the structure does not decay between features. |
| Before a big build | Point it at the [spec](https://www.aihero.dev/ai-coding-dictionary/spec) and ask "how can we make this change easy?" This is the most effective prompt for it. |
| Brownfield audit | Run it on a large, unstructured or [vibe-coded](https://www.aihero.dev/ai-coding-dictionary/vibe-coding) repo to find out what shape it is in. |
| Legacy test work | Use it to find the missing seams before you write tests against untestable code. |

Where it is confusable with siblings:

- To design one module you have already chosen, use [codebase-design](https://aihero.dev/skills-codebase-design). This skill finds the module to work on, and `codebase-design` is where you design it.
- For a whole effort too big to hold in one session, use [wayfinder](https://aihero.dev/skills-wayfinder).
- For "this specific thing is broken," use [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs). It sends you back here when the real finding is that there is no good seam to lock the bug down.

## Prerequisites

An issue tracker configured by [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills), for the last step. Without one, the run prints the issue bodies instead of creating them. Codex is optional: with the `codex` CLI installed it plays the opponent in each debate, and without it a fresh Claude subagent does. If `GLOSSARY.md` or ADRs in `docs/adr/` exist, it reads them and uses your domain's own nouns. A candidate then reads as "deepen the Order intake module," not "refactor the FooBarHandler."

It writes in three places. The report and the debate notes go to your temp directory, outside the repo. The picked candidates and defects become issues on your tracker. When a disagreement needs settling, it adds or sharpens terms in `GLOSSARY.md`, and creates that file if it does not exist. It also offers to record a rejected candidate as an ADR, so a future run does not suggest it again.

## Depth, and the report that hunts for it

The skill rests on one idea: **depth**. A deep module puts a lot of behaviour behind a small, stable interface. A shallow module exposes its implementation through an interface nearly as wide as the code beneath it. The report looks for three forms of shallowness:

- Pure functions extracted only for testability, while the real bugs are in how callers use them (no **locality**).
- Modules that leak across their **seams**.
- A concept you cannot understand without opening five files.

For each one, it proposes the deepening that fixes it.

Each candidate is a card with the files involved, the friction, a plain-English solution, the benefit in terms of **locality** and **leverage**, a before/after diagram, and a strength badge.

| Badge | What it means for you |
| --- | --- |
| `Strong` | The deletion test passes clearly and the friction is real. Take these seriously. |
| `Worth exploring` | Plausible deepening, but the payoff depends on where the code is going next. |
| `Speculative` | Included for completeness. You can ignore most of these. |

The badge is the debated verdict, with the original strength beside it when the debate changed it. Rejected candidates get one line each at the end, with the reason, so you can see what was cut and why. Above the cards, a **Defects found** section lists real bugs either side of a debate turned up in today's code, each checked against the code before it reaches the report. In practice this section can matter more than the refactors.

The report ends with a **Top recommendation**, the candidate it would do first. Then the skill stops and asks which candidates and defects should become issues. You answer once, with numbers and `D` ids. At that point no code has changed.

## What happens after you pick one

Most picks go straight to an issue. The debate already settled the problem, the rough shape behind the seam, what to cut, and the first step, so a [grilling](https://aihero.dev/skills-grilling) session runs only for a picked candidate whose debate left a disagreement, and only on that disagreement. Say "skip grilling" and those disagreements go into the issue as open questions instead.

Each issue carries the debate's conclusion: problem, solution, what not to do, first step, evidence with file:line, the test surface, decisions and open questions, and how to verify. The run ends with one line to type next, `/implement-spec` followed by every issue it created. That run builds, reviews and opens a PR per issue, and splits a large one into tickets itself, so [to-spec](https://aihero.dev/skills-to-spec) and [to-tickets](https://aihero.dev/skills-to-tickets) are optional here.

## Common questions

**It grilled me for an hour about one idea instead of showing me options. Can I turn that off?**

Mostly, yes, and now by default. This was the most common complaint about the skill. One user liked it as "a convenient way to get a thorough analysis of improvements," but after the grilling loop was added, found it "borderline unusable." In their sessions it proposed a single solution and then asked "10's or 100's of questions." The debate step now settles most of those questions before you see the report. Grilling runs only on a disagreement the debate left open, for a candidate you picked. Say "skip grilling" and even those become open questions in the issue. Weaker [models](https://www.aihero.dev/ai-coding-dictionary/model) may still jump into an interview early; if one does, point it back at the report.

**The report opened as unstyled raw HTML with no diagrams. What happened?**

The report loads Tailwind and Pele (a Mermaid-syntax renderer) from CDNs, so it needs network access when you open it. If something blocks those scripts, the page breaks with no error. In the reported case, a security hook required SRI hashes. The agent added them, but the CDN served different bytes to the browser than to the `curl` that computed the hash, so the browser blocked the script. Offline and locked-down environments have the same problem. The agent cannot see it, because it never renders the page. As a workaround, ask for inline CSS and hand-built SVG diagrams instead of the CDN setup. This is an open issue.

**It gave me twelve candidates. Do I work through them in the same session or start a new one?**

Neither. Pick every one worth doing in one reply; the run turns each into an issue and the code changes happen later, in one `/implement-spec` run that takes them all and gives each its own branch and PR. The report is only a temp file, so the issues are what carry the candidates forward. Leave the ones you don't pick: the report lists why the debate demoted or rejected them, and a future run will find any that still matter.

**How should I prompt it?**

Prompt it with the next thing you are building. If a big build is coming up, point it at the spec and ask "how can we make this change easy?" A run with no prompt looks for hot spots on its own. That is fine for routine upkeep, but a direction makes the report actionable.

**Does it work on a large legacy codebase?**

Partly. It works well on big existing codebases that lack consistent structure, and it is the recommended upkeep tool after any one-time structural setup. But users with out-of-control projects report it "helped a little but still doesn't seem to cut it." One developer with an eight-year-old legacy codebase reported that the model went in circles, though the same skill produces a clean graph on a tidy repo. There is no dedicated `/refactor` skill for that case yet. If the codebase has no shared vocabulary, run [grill-with-docs](https://aihero.dev/skills-grill-with-docs) first to create one. That usually makes this skill's output much better.

**How is this different from `/codebase-design`?**

`/codebase-design` is a reference, not a session driver. It supplies the vocabulary (module, interface, depth, seam, adapter, leverage, locality), and this skill uses it. If you give a fresh agent `/codebase-design` as the task, it fails in a known way. It has no process of its own to follow, so the agent invents one, explores the code again, and runs for a very long time before it asks you anything. Run this skill, and let it use that one.

**Will it ever tell me the codebase is fine?**

Rarely, so know that before you start. The skill exists to output findings, so it tends to produce candidates instead of concluding that nothing is wrong. Use the strength badges to correct for this. If every candidate in a report is `Speculative`, the skill found nothing.

**Does it work in Codex or another harness?**

Partly. The exploration step names Claude Code's `Agent` tool with `subagent_type=Explore` directly. A [harness](https://www.aihero.dev/ai-coding-dictionary/harness) without that tool may skip the parallel exploration instead of using its own equivalent. The skill still runs, but the scan is less thorough. Someone has proposed a harness-neutral rewrite, but it is not merged.

**How do I actually implement deep modules in TypeScript?**

The skill does not ship a good answer. People often ask for a `TYPESCRIPT.md` with concrete file and module layouts for the principles, and it does not exist. The skill tells you where a deepening belongs and what goes behind the seam. You must turn that into a package or directory structure yourself.

## It's working if

- The candidates name your domain's concepts, not invented class names: "the Order intake module," not "the FooBarHandler."
- The candidates are in files you edited recently, not in parts of the repo nobody touches.
- No code changed during the run. The only new files are the report and the debate notes in your temp directory.
- Each candidate has a debate report before the HTML is written, and some verdicts moved: a run where every `Strong` stayed `Strong` probably skipped the opponent.
- It stops after the report and asks once which candidates and defects to register. It does not continue on its own.
- It ends with one `/implement-spec` line naming every issue it created.
- Each card explains the payoff as locality or leverage, and says which tests get simpler, not just "this is cleaner."
- When you reject a candidate for a lasting reason, it offers to record an ADR, so the next run does not suggest it again.

## Where it fits

`improve-codebase-architecture` is **periodic maintenance**. You run it every few days, outside any chain, to queue up work, not to do it. Its neighbours:

- [codebase-design](https://aihero.dev/skills-codebase-design) owns the depth-and-seam vocabulary that every candidate uses.
- [grilling](https://aihero.dev/skills-grilling) settles a disagreement the debate left open, on a candidate you chose.
- [domain-modeling](https://aihero.dev/skills-domain-modeling) keeps `GLOSSARY.md` and the ADRs current as you make the decision.

Its output is a set of agent-ready issues, which go straight to [implement-spec](https://aihero.dev/skills-implement-spec). For a candidate you want to think through at length first, take it into [grill-with-docs](https://aihero.dev/skills-grill-with-docs) instead. Its counterpart at the end of the main flow is [retro](https://aihero.dev/skills-retro). This skill improves the code the agent works in, and `retro` improves the environment around it (checks, standards, steering files) after a build. For which skill fits a situation, [ask-matt](https://aihero.dev/skills-ask-matt) is the router over the whole set.
