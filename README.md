# poteto-sdlc

Two complementary Claude skills.

| Skill | Answers |
|---|---|
| [`ai-native-sdlc`](skills/ai-native-sdlc/SKILL.md) | Which lifecycle stage are we in, what artifact does it produce, who signs off? First-principles, right-sized gates: Intent → Spike → Spec → Guardrails → Plan & Build → Checks → QA → Deploy → Monitor → Outcome Review → Incident → Regression → Prevention. |
| [`poteto-mode`](skills/poteto-mode/SKILL.md) | Given we're writing code now, which playbook applies (bug fix, feature, refactor, perf, visual UI) and how do we verify it's done? |

Use them together during Plan & Build and Fix; `poteto-mode` also works alone for one-off tasks.

## Install

Copy a skill folder into `~/.claude/skills/` (personal) or `.claude/skills/` (project):

```bash
cp -r skills/ai-native-sdlc skills/poteto-mode ~/.claude/skills/
```
