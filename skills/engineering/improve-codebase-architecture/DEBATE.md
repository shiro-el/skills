# Debate

Every candidate is cross-examined before the report is written. A **debate subagent** argues the candidate's case; an **opponent** with no stake in it verifies the evidence independently and attacks the weak points. Exploration overstates: in practice debate demotes most `Strong` candidates, rejects a few outright, and turns up real defects the exploration missed. Both outcomes belong in the report.

## Setup

Make a debate folder outside the repo, `<tmpdir>/arch-debate-<timestamp>/`, with one subfolder per candidate (`c01/`, `c02/`, …). Write each candidate's **brief** to `cNN/brief.md`: title, files with line references, the friction, the proposed deepening, the original strength. Copy the opponent instructions below to `opponent.md` in the debate folder.

Pick the opponent once, before spawning anything:

- **Codex**, when the `codex` CLI runs (`codex --version`). The debate subagent calls it itself, read-only.
- **A fresh Claude subagent** otherwise. Then you relay: for each opponent turn, spawn a new subagent whose prompt is `opponent.md` plus pointers to the candidate's files so far, and have it write the turn file.

Spawn one debate subagent per candidate, all at once, in the background.

## Debate subagent brief

Give each debate subagent pointers, not pasted content: its `cNN/` folder, `opponent.md`, the repo path, the glossary and ADRs, and the rules below.

- Read-only. Never edit the repo, start databases, containers or servers, or commit. Write only inside `cNN/`.
- Use the `codebase-design` vocabulary exactly (module, interface, implementation, depth, deep, shallow, seam, adapter, leverage, locality) and the deletion test. A proposal that fights a repo rule or an ADR says so.

Its steps:

1. **Verify the evidence.** Open every file the brief names and check each claim (line references, counts, call sites). Record what is wrong or overstated.
2. **Opening position** in `round1-claude.md` (≤400 words): is the friction real, what sits behind the seam, the rough shape of the small interface (no full type design), which tests survive or get replaced, risks, recommended strength.
3. **Opponent turn** into `round1-opponent.md`. With Codex, run it in the repo, feeding the brief, the position and `opponent.md`:

   ```bash
   cat cNN/brief.md cNN/round1-claude.md opponent.md \
     | timeout 900 codex exec -s read-only --ephemeral -C <repo> -o cNN/round1-opponent.md - > cNN/opponent-log1.txt 2>&1
   ```

   Give the tool call a 900000 ms timeout. On a non-zero exit or empty output, read the log tail and retry once; if it still fails, finish as a self-critique and say so in the report. Without Codex, end the turn here and report back; the orchestrator relays the opponent and resumes you.
4. **Rebuttal.** Check every new code claim the opponent makes yourself, then answer point by point in `round2-claude.md`, conceding what is right. Send the whole transcript to the opponent again for `round2-opponent.md`. Stop after round 2 unless a material disagreement remains; then at most one more round.
5. **Report** in the format below. It is the only thing the orchestrator reads.

## Report format (≤500 words)

```
CANDIDATE: NN, <title>
VERDICT: Strong | Worth exploring | Speculative | Reject   (was: <original>)
EVIDENCE CHECK: claims confirmed or corrected, with file:line
AGREED PROBLEM: one or two sentences
AGREED SOLUTION: what sits behind the seam, the rough shape of its interface, where the seam lives
NOT DOING: what the debate cut from the original proposal
TESTS: what the interface test surface becomes; tests deleted or replaced
RISKS / COSTS: bullets
FIRST STEP: the smallest change that starts it
DEFECTS FOUND: real bugs either side found in the code, with file:line (none is a valid answer)
DISAGREEMENTS LEFT: each with both sides' argument in one line (none is a valid answer)
OPPONENT: codex, N rounds | claude, N rounds | failed (why)
```

## After the debates

Before writing the report, open the code yourself for every `DEFECTS FOUND` entry and every verdict change, and mark each defect confirmed or unconfirmed. A defect is a bug in today's code, independent of whether its candidate survives, so it gets its own place in the report.

## Opponent instructions (`opponent.md`)

```markdown
# Instructions for the opponent

You are the opposing reviewer in an architecture debate about the candidate above. The repository is the current directory; read its AGENTS.md, GLOSSARY.md and ADRs as needed. Read-only: modify nothing, start no databases or servers.

1. Independently verify the code claims in the brief and in the latest position by opening the files. Call out anything wrong, overstated, or missing.
2. Argue against the weak points. Is the friction real or cosmetic? Does the deepened module pass the deletion test (deleting it concentrates complexity rather than moving it)? Is the seam in the right place? Is it a hypothetical seam with only one adapter? Does it fight a repo rule or an ADR? Is there a smaller alternative?
3. Concede plainly where the position is right. Disagree only where the evidence supports it.
4. Report any real bug you find in today's code, even if it is off the candidate's topic.
5. Use the vocabulary module, interface, implementation, depth, seam, adapter, leverage, locality.

Reply in ≤400 words with sections: VERIFIED, CHALLENGES, ALTERNATIVE (if any), CONCESSIONS, DEFECTS, RECOMMENDED STRENGTH (Strong / Worth exploring / Speculative / Reject).
```
