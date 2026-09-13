---
name: ai-native-sdlc
description: "Runs the AI-native software lifecycle from first principles: Intent, Spike / Research, Spec, Guardrails, Plan and Build, Checks, Independent QA, Deploy, Monitor, Outcome Review, Incident, Regression, Prevention, with right-sized human gates and Claude Code hooks. Use whenever the user starts or kicks off a feature, fix, or idea through a process; mentions SDLC, intent doc, spec, spike, prototype or research before building, plan mode, REVIEW.md, deploy gates, monitoring bands, incidents, post-mortems, regression evals, or reviewing whether a shipped feature worked or whether a process gate is worth keeping. Pairs with poteto-mode for the build itself."
---

# AI-Native SDLC

This skill governs the lifecycle: which stage you're in, what it produces, who signs off. For how to write the code (Stages 5 and 8), invoke **poteto-mode** alongside it.

---

## Core Principle — First Principles Over Ceremony

The lifecycle exists for one purpose: **ship the right thing, safely, and learn.** Every stage and gate is a means to that end, and earns its place only by reducing a real risk or uncertainty. Copying process because "good teams do it" is reasoning by analogy.

At each stage, run its **first-principles check** (listed per stage). The general form:

1. **What are we trying to achieve?** Name the user or business outcome, not the solution.
2. **What is known vs assumed?** Write assumptions down with how you'd validate each.
3. **What must be true?** State constraints as numbers and invariants, not preferences.
4. **What is the smallest step that retires the biggest uncertainty?** Do that next.

---

## Step 0 — Size, Then Route

State the tier and why. Run only its path. Keep replies to the user short — tier, next action, who approves — and put detail in the artifacts.

| Tier | When | Path |
|---|---|---|
| **Trivial** | Typo, copy, a config value that is not a threshold, cutoff, or limit changing what the system decides (a label, colour, or page size is fine), patch-level dependency bump | Build → Checks → QA → Merge |
| **Standard** | Bug fix or small feature in one area; need and approach are clear | Intent → [Spike / Research, if a research trigger applies] → Plan & Build → Checks → QA ⇄ Fix → Merge → Deploy |
| **Exploratory** | High uncertainty: do users want it? Is it feasible, fast, or cheap enough? | Intent → Spike / Research → decide: Standard, Full, build smaller, pivot, or stop |
| **Full** | Cross-cutting, new service, or anything on the escalation list | All stages |
| **Incident** | Monitoring band breached or production defect | Incident → Regression → Prevention |

**Escalate to Full** when the change touches auth, payments, PII, schema or migrations, public API contracts, or infrastructure. When unsure, choose the higher tier. Exploratory work that touches the escalation list still runs Spike / Research first, then proceeds as Full.

**Research triggers.** Run Spike / Research (evidence mode) before building when any of these hold: the change **sets or moves a threshold, cutoff, or limit that changes what the system decides**; whether it works **can only be judged on real data**; or the intent marks an assumption **high-risk and unvalidated**. A threshold change is never Trivial, however small the diff. "The approach is clear" is exactly when an untested data assumption slips past every gate and surfaces mid-build, where the fix is rework and any threshold it moves looks chosen after seeing the result. Full tier always runs evidence mode; Exploratory runs the mode its question needs and always checks repo prior art first.

**Locate the current stage from committed artifacts:** no approved intent → Intent · open riskiest assumption or research trigger → Spike / Research · Full tier without approved spec → Spec · no plan → Plan & Build (Full runs Guardrails first) · PR open → Checks, QA, or Fix · shipped and review date reached → Outcome Review.

---

## The Loop

```
Intent → [Spike / Research] → Spec → Guardrails → Plan & Build → Checks → Independent QA ⇄ Fix
→ Merge → Deploy → Monitor → Outcome Review → keep | iterate | delete
                       ↘ Incident → Regression Test/Eval → Prevention Update → next Build
Quarterly: Process Review (gate budget)
```

`[Spike / Research]` is required on Full, Exploratory, and Standard work with a research trigger; skipped otherwise.

Every artifact is committed. One `<slug>` per work item:

```
docs/intent/<slug>.md    docs/spikes/<slug>.md    docs/specs/<slug>.md
docs/plans/<slug>.md     docs/outcomes/<slug>.md  docs/postmortems/YYYY-MM-DD-<slug>.md
```

---

## 1. Intent — `docs/intent/<slug>.md`

Turn an idea into a committed artifact before engineering starts.

```
problem:            # one sentence, in terms of a user or business outcome
proposed_outcome:
success_criteria:   # measurable, with a target and a review date
assumptions:        # each with: how we'd validate it, and risk if wrong
affected_users:
constraints:
out_of_scope:
open_questions:
```

**First-principles check:** Separate the need from the proposed solution — ask "why" until you reach an outcome. Could we delete, reuse, or configure something instead of building? Which assumption, if wrong, kills the idea? If that assumption is unvalidated, route to Spike / Research.

**Gate:** Product owner approves and commits.

---

## 2. Spike / Research — `docs/spikes/<slug>.md` (Exploratory, Full, Standard with a research trigger)

Two modes, one artifact. Pick by the question: "will it work, will users want it?" → **Prototype**. "Is it already known, and what do real data say?" → **Evidence**. A Full-tier item often needs both.

### Prototype mode

When code is cheap, building a quick throwaway often answers a question faster and more reliably than debating it in a spec.

- **Target the riskiest assumption** from the intent — one question per spike.
- **Time-box it** (typically 1–2 days) and agree the box up front.
- **Keep it disposable:** branch `spike/<slug>`, never merged. Use synthetic or approved data; no production PII.
- **Record the result:**
  ```
  date:
  question:
  prior_art:          # incl. past negative results; always checked first
  method:
  evidence:           # numbers, screenshots, user reactions
  decision: proceed-standard | proceed-full | build-smaller | pivot | stop
  learnings_for_spec:
  ```

### Evidence mode

Find out what is already known before designing. Read-only: throwaway probe scripts, no production code. **Time-box it:** hours on Standard, 1–2 days on Full or Exploratory.

- **Data handling** — read-only access to data its owner has approved; no PII, credentials, or secrets in pasted output (redact or summarise, since the record is committed); anything on the escalation list goes through its owner.
- **Repo prior art first** — `CLAUDE.md` Common Mistakes, existing modules, past specs, and above all past *negative* results. A failed earlier attempt is the most valuable thing research finds.
- **Outside evidence** — papers, official docs, published methods, with links. "Nothing found" is a result; record it.
- **Real-data probes, output pasted** — does the data exist and look sane; the method on a case whose answer is already known; a replay of real logged inputs through the proposed logic; the degenerate cases (empty inputs, changed formats or schedules, one-off outliers).
- **Bars before measuring** — the thresholds the work will be judged on and the result that kills it, fixed here so none is chosen after seeing the numbers.
- **Record the result:**
  ```
  date:
  question:
  prior_art:          # incl. past negative results
  outside_evidence:
  probes:             # command + pasted (redacted) output
  bars:               # thresholds, kill criterion, what counts as unmeasurable
  decision: proceed-standard | proceed-full | build-smaller | pivot | stop
  ```

A probe that contradicts the intent goes back to the human; it is not quietly designed around.

### Both modes

- A record is **finished** when `decision:` is filled; other fields may stay empty when they don't apply. Dates are ISO (`YYYY-MM-DD`).
- `build-smaller` returns to Intent with the reduced scope; `pivot` returns to Intent with the new question.
- A spike decision never moves an item off the escalation list: `proceed-standard` on anything listed there still proceeds as Full.
- If the repo has no evidence-before-build check yet, add it now (see Guardrails) — at any tier, not only Full.
- If a bar must change later, the spec (or the plan, on Standard) records the change and why.

**First-principles check:** Does the evidence answer the question, or only suggest an answer? A "stop" is a success — it saved the build.

**Gate:** Human reads the result and picks the decision.

---

## 3. Spec — `docs/specs/<slug>.md` (Full tier)

Collapse requirements and design into one session, with org policies (security, compliance, UX, brand skills) as live constraints.

```
date:
evidence:             # link to the finished docs/spikes/<slug>.md (always required); bars come from it
requirements:
invariants:           # what must always be true, e.g. "a user sees only their tenant's data"
constraints:          # numbers: latency, volume, cost, consistency
architecture:         # each component traced to a requirement or constraint
api_contracts:
data_model_changes:
rollout_plan:         # flags, migration order, rollback path
security_concerns:    # route to policy owner
compliance_flags:     # route to policy owner
```

**First-principles check:** Derive the design from the constraints, not from how another system does it. Remove any component you can't trace to a requirement.

**Gate:** Human sign-off; flagged concerns resolved before engineering sees the spec.

---

## 4. Guardrails (Full tier, pre-build)

Turn spec risks into automated checks before code exists. Deliver every change as a PR.

- **`CLAUDE.md`** keeps four sections: `## Build & Test`, `## Architecture`, `## Common Mistakes`, `## Conventions`.
- **Skills** in `.claude/skills/<name>/SKILL.md` for institutional knowledge.
- **Hooks** for blocking checks on Claude's tool calls.
- **An evidence-before-build check:** a repo test that fails when a new spec's `evidence:` links no finished `docs/spikes/` record; when a new plan has no `research_trigger:`, or its trigger is not `none` and it links no finished record; or when a linked record is dated after the spec or plan linking it. A plan with `research_trigger: none` and no spec passes untouched. Add it the first time any work item needs research, whatever its tier. It prevents research happening mid-build, where it becomes rework.

**Each guardrail names the failure it prevents** (a comment or a line in `CLAUDE.md`). That is what makes the quarterly Process Review possible.

**How Claude Code hooks work:**
- **Registration:** declare hooks in `.claude/settings.json` under `hooks`. Scripts in `.claude/hooks/` are only a folder convention; nothing loads them automatically.
- **Exit codes:** `0` = proceed · `2` = block, stderr goes back to Claude · other non-zero = non-blocking error, action proceeds.
- **Human approval:** use `permissions.ask` rules in settings, not exit codes.
- **Scope:** hooks govern Claude only. Mirror each rule for humans with git pre-commit hooks and required CI checks.

```json
{
  "hooks": {
    "PreToolUse": [
      { "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/protect-tests.sh" }] },
      { "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/block-secrets.sh" }] }
    ]
  }
}
```

---

## 5. Plan & Build — `docs/plans/<slug>.md`

1. **Start in plan mode.** Read the spec (or intent) and propose:
   ```
   date:
   research_trigger: none | threshold | real-data | high-risk-assumption | full-tier
   evidence:         # link to the finished docs/spikes/<slug>.md unless research_trigger is none
   files_to_change: []
   work_order: []
   tests_to_write: []
   risks: []
   done_when: []     # quantifiable: "all tests pass", "p99 < 300ms on staging"; use the research bars
   ```
2. The engineer iterates and commits the plan.
3. Execute each `work_order` item under **poteto-mode** (router picks the playbook; verify against the real artifact before the next item).
4. Parallelize only independent items (git worktrees or `.claude/agents/` subagents).

**Rules:**
- **Bug fixes start with a failing test** that reproduces the bug.
- **Existing tests stay intact** — never weaken assertions to go green. New regression tests go in new files.
- **The plan is the review reference.** Explain deviations in the PR description. After merge the plan is disposable; the spec stays as the decision record.

---

## 6. Deterministic Checks

Run the `CLAUDE.md` → `## Build & Test` commands, fastest first, stopping at the first failure:

```
lint → type-check → unit tests → secret & dependency scan → integration tests
```

Green locally before requesting review; required CI checks green before merge.

---

## 7. Independent QA

A reviewer with no build context checks the diff against policy, plan, and invariants.

- **Managed:** Claude Code Review reads `REVIEW.md` and `CLAUDE.md`. Its check run always finishes neutral, so gate merges by parsing its severity counts in CI.
- **Self-run:** `git diff origin/main...HEAD | claude -p "Review this diff against REVIEW.md, docs/plans/<slug>.md, and the spec's invariants. Output ranked findings."` (Local `/code-review` doesn't read `REVIEW.md`, so name it.)

Findings:
```
severity: critical | high | medium | low
pre_existing: true | false
file: / line:
finding:
evidence:        # file:line citation, not inference from naming
remediation:
```

New critical/high findings block merge; medium/low are advisory; pre-existing findings become new intents.

**First-principles check:** For each spec invariant, find the code that guarantees it. A same-model reviewer shares blind spots with the builder, so Full-tier PRs also get a human reviewer.

**`REVIEW.md`** stays short: Merge Criteria · Auto-block Triggers (secrets, schema change without migration, weakened tests) · Always Check (repo rules) · Do Not Report (generated code, lockfiles, CI-enforced items).

---

## 8. Fix → Re-QA

Fix in severity order, each as a poteto-mode **Bug Fix**. Re-run affected checks, commit each fix separately, and answer disputed findings with evidence for the human to decide. Re-QA runs in a **fresh session**; the fixing session never approves its own work.

---

## 9. Merge → Deploy

- Agent output → PR → human merge. Branch protection blocks direct pushes and deploys.
- Claude triages CI failures (`claude -p "Why did this step fail?" < logs.txt`) and drafts changelogs.
- **Pre-deploy gate** (CI job or deployment protection rule): named human authorization, rollback path from `rollout_plan` confirmed, credentials scoped to this run.
- Every non-interactive run is attributed to its triggering identity.

---

## 10. Monitor — `bands.yaml`

Deterministic scripts watch production; Claude engages only on a breach.

```yaml
# Start from user impact (SLOs), then calibrate against baseline:
# warn ≈ mean + 1σ, alert ≈ + 2σ, act ≈ + 3σ over a trailing 7 days.
metrics:
  error_rate:     { warn: 0.01, alert: 0.05, act: 0.10 }
  p99_latency_ms: { warn: 500,  alert: 1000, act: 2000 }
```

- `warn` → log only.
- `alert` → Claude reads metrics, logs, and traces and posts a read-only diagnosis.
- `act` → page on-call; Claude opens a revert or fix PR and an intent. Claude rolls back only through a **pre-approved runbook** that policy explicitly grants.

---

## 11. Outcome Review — `docs/outcomes/<slug>.md`

Incidents tell you what broke; outcome reviews tell you whether the work was worth doing. Run it on the review date set in the intent's `success_criteria`.

```
success_criteria:   # copied from intent
actual:             # measured with production data, with source
verdict: met | partially-met | not-met
decision: keep | iterate | delete
reasoning:
```

- **Keep** → close the intent.
- **Iterate** → new intent with what was learned.
- **Delete** → removal intent. Unused features still cost maintenance, attack surface, and cognitive load.

**First-principles check:** Would we build this again, knowing what we know now?

**Gate:** Product owner makes the decision.

---

## 12. Incident / Defect

Claude (e.g., in the incident Slack channel) reads MCP-exposed metrics and logs, proposes a hypothesis and next diagnostic step, and verifies recovery against the breached band after a human deploys the fix. Claude proposes; humans merge and deploy. The only exception is a pre-approved rollback runbook. Log every action with a timestamp.

Blameless post-mortem:
```markdown
## Timeline
## Detection      ← how found; why not sooner
## Root Cause     ← why the system allowed it
## Impact
## Fix
## Action Items   ← each becomes an intent
```

**First-principles check:** Keep asking "why did the system allow this?" until you reach a condition you can change: a missing check, a misleading signal, unclear ownership. "Human error" is a starting point, not a root cause.

---

## 13. Regression Test / Eval

- **Product defect** → regression test in the normal suite; it fails on the pre-fix commit and passes after.
- **Agent or process failure** (Claude ignored a rule, skipped a gate) → eval case in `.claude/evals/<slug>.yaml` with `task_description`, `input`, `expected_behavior`, `failure_mode`, `deterministic_checks`. Run evals in CI when `CLAUDE.md`, skills, hooks, or agents change.

---

## 14. Prevention Update → Back to Build

- [ ] `CLAUDE.md` → `## Common Mistakes` updated
- [ ] Skill updated or created
- [ ] Hook added or tightened, naming the failure it prevents
- [ ] Regression test or eval passing in CI
- [ ] `bands.yaml` tuned if detection was late
- [ ] Action items committed as intents

---

## Process Review — Gate Budget (quarterly)

Apply first principles to the process itself. For every gate, hook, check, and approval step:

| Question | Evidence |
|---|---|
| What did it catch this quarter? | Blocked runs, findings, rejections |
| What did it cost? | Wait time, false positives, reruns |
| Is anyone actually reviewing? | Approval time and rejection rate |

- **Caught real issues** → keep.
- **Caught nothing and costs a lot** → propose removing it or making it advisory.
- **Approvals are fast with near-zero rejections** → rubber-stamping; automate the check or remove the step.
- **Repeated escapes in one area** → add or tighten a gate there.

Deliver changes as PRs to this skill and `CLAUDE.md`. Escalation-list gates and production authorization stay unless the security owner signs off.

---

## Governance Rules

1. **No self-approval** — building or fixing sessions never approve their own output.
2. **Commit chain** — intent → spike / research → spec → plan → PR → outcome or post-mortem.
3. **Humans decide** — intent, spike / research decision, spec, merge, production, outcome.
4. **Enforcement as code** — policies live in version control and required checks.
5. **Separation of duties** — branch protection and named authorizers.
6. **Transparency** — every invocation logged with identity and timestamp.
7. **Right-sized and self-pruning** — the tier sets the gates; the gate budget keeps them honest.

**Relationship to poteto-mode:** this skill decides stage, artifact, and sign-off; poteto-mode decides the playbook and verification for each code change. Keep them separate — poteto-mode also serves one-off tasks.

---

## File Map

```
repo/
├── CLAUDE.md  REVIEW.md  bands.yaml  .pre-commit-config.yaml
├── .claude/
│   ├── settings.json            # permissions + hook registration
│   ├── skills/<name>/SKILL.md
│   ├── hooks/*.sh               # referenced from settings.json
│   ├── agents/<name>.md
│   └── evals/<slug>.yaml
├── ci/pre-deploy-gate.*
└── docs/{intent,spikes,specs,plans,outcomes,postmortems}/
```

---

## Measurements

| Area | Metric |
|---|---|
| Intent | Idea → approved intent time |
| Spike / Research | % ending in pivot, build-smaller or stop (cheap learning); bar changes not recorded in the spec or plan (target 0) |
| Build | % of PRs merged on first implementation pass |
| Checks | First-pass CI success rate |
| QA | Defects caught in QA vs escaped to production |
| Deploy | Deploy frequency; change failure rate |
| Outcome | % of shipped work meeting success criteria; features deleted |
| Incident | Time to recovery; recurrence after regression test added |
| Process | Gates added vs removed; median approval wait |

---

## Starting Points

1. **`CLAUDE.md`** — capture build commands and conventions today.
2. **One intent** with measurable success criteria and a review date.
3. **One secret-detection check** — git pre-commit plus a matching `PreToolUse` hook.
4. **One regression test** for the last production bug.