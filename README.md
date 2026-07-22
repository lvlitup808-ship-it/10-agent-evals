# 10 Agent Evals for AI Engineers

**Explained with usage.**

> Offline evals tell you it works.  
> Online evals tell you it still works.  
> Both sides matter, but not all ten do.  
> Run the two that would have caught your last outage.

Inspired by [@hanakoxbt](https://x.com/hanakoxbt/status/2079692306337153079).

---

## The 10 Evals

### 1. Golden Set
**A fixed set of cases you never edit, run on every single change.**

Use as the baseline that tells you whether anything moved at all.

- Keep 20–50 high-signal cases that represent real failure modes.
- Never edit them once locked (or version them carefully).
- This is your canary. If golden set score drops, stop the ship.

### 2. LLM as Judge
**A second model scores the output against a written rubric.**

Use when the answer is open-ended and there is no string to match against.

- Write a clear rubric (correctness, helpfulness, tone, safety, etc.).
- Prefer structured JSON output from the judge.
- Always calibrate the judge with human labels periodically.

### 3. Rubric Scoring
**One number per dimension: correctness, tone, safety, cost.**

Use when a single score hides which part actually got worse.

- Break evaluation into orthogonal axes.
- Track each dimension over time so regressions are diagnosable.
- Cost is a first-class citizen for production agents.

### 4. Trajectory Eval
**Grade the path the agent took, not only the answer it landed on.**

Use when the right answer for the wrong reason is going to bite you later.

- Inspect tool call sequences, intermediate reasoning, and recovery from errors.
- Catch “lucky” successes that will fail under distribution shift.
- Especially critical for multi-step agents (clipping, research, coding, ops).

### 5. Tool Unit Tests
**Test each tool on its own, with fixtures, no model in the loop.**

Use always. Most agent bugs are tool bugs wearing a costume.

- Fixtures + deterministic assertions on inputs/outputs.
- Mock external APIs.
- This should be the fastest, cheapest layer in your eval pyramid.

### 6. Regression Suite
**Replay past runs against the new prompt or model and diff the results.**

Use before every prompt change, because prompts have no type system.

- Store full trajectories of production (or high-quality) runs.
- Diff final answers + intermediate steps.
- Surface “this used to work” failures instantly.

### 7. A/B in Prod
**Split live traffic between two versions and compare outcomes, not vibes.**

Use when offline scores stopped predicting what users actually do.

- Define clear success metrics (task completion, user satisfaction, latency, cost).
- Statistical significance matters.
- Kill the loser quickly.

### 8. Human Review
**Sample a slice of runs and have a person grade them honestly.**

Use to calibrate your judge, because a judge nobody checks quietly drifts.

- 5–10% sampling is usually enough if done consistently.
- Use it to improve rubrics and find blind spots in automated graders.
- High-stakes domains need more human coverage.

### 9. Shadow Run
**The candidate runs on real traffic in parallel and its output is shown to nobody.**

Use before a risky rollout, when one bad answer would be expensive.

- Log the shadow output + compare to production online.
- Great for new models, new tools, or major prompt rewrites.
- Zero user impact, maximum learning.

### 10. Red Team
**Deliberately attack it: jailbreaks, injection, exfil, tool abuse.**

Use before anyone external can reach it, not after.

- Prompt injection, tool call injection, data exfiltration attempts, resource exhaustion.
- Automate what you can, but keep creative human red-teaming.
- Treat security as part of the eval suite, not a separate activity.

---

## Recommended Eval Pyramid (for most teams)

1. **Tool unit tests** (always on, every PR)
2. **Golden set + Regression suite** (every change)
3. **Trajectory + Rubric / LLM-as-judge** (nightly or on major changes)
4. **Shadow + A/B** (before production promotion)
5. **Human review + Red team** (continuous + pre-launch)

Start with the two that would have caught *your last outage*. Everything else is optional until you need it.

---

## Why this matters for multi-agent & live systems

If you’re building live stream clippers, autonomous content agents, NIL intelligence systems, or any agent that chains tools and decisions:

- Tool unit tests catch the silent failures (empty JSON, wrong schema, rate limits).
- Trajectory evals catch the “it clipped the wrong moment for the right reason” class of bugs.
- Shadow runs let you test new clipping heuristics or models on real streams without shipping bad clips.
- Golden sets of known high-value moments become your regression safety net.

---

## Quick Start Ideas

```bash
# Example structure you can adopt
evals/
  golden/
    cases.json          # locked test cases
  tools/
    test_tools.py       # pure unit tests
  trajectories/
    store/              # past successful + failed runs
  judges/
    rubric.md
    llm_judge.py
  redteam/
    attacks/
```

Build the cheapest, most deterministic layers first. Add the expensive/stochastic ones only when they buy you signal.

---

**Save this. Use it.**

Original thread + visual: [hanakoxbt on X](https://x.com/hanakoxbt/status/2079692306337153079)
