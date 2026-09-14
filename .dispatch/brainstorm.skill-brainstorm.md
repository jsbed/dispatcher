## Skill: brainstorming

Read `/Users/jsbedard/.pi/agent/git/github.com/obra/superpowers/skills/brainstorming/SKILL.md`
now and follow it. If that file is missing, the distillation below is the
contract.

**Classify the request out loud before your first question**, so the human can
override you:

- **Spike** — a feasibility question. Present the question and the probe in 2-3
  sentences, get a nod, find out as cheaply as correctness allows, report a
  recommendation.
- **Bounded** — a well-scoped change to a flow that already exists in this repo
  (you must be able to open that flow and read it; familiarity with the *kind*
  of app is not bounded). Ask the clarifying questions that matter, present a
  short design in chat, stop.
- **Architectural** — new subsystems, or changes that restructure how parts fit
  together. Full process: context, questions one at a time, 2-3 approaches with
  trade-offs and your recommendation, then a sectioned design approved section
  by section.

When in doubt between two paths, take the heavier one. The ratchet is one-way:
hidden complexity discovered mid-session upgrades the path — say so and step up.

Then:

- Explore the project state first — files, docs, recent commits.
- Ask questions **one per message**, focused on purpose, constraints and success
  criteria. Multiple-choice is often kinder than open-ended.
- Propose 2-3 approaches with trade-offs; lead with your recommendation and the
  reason for it. YAGNI ruthlessly.
- Present the design in sections scaled to their complexity, checking after each
  one. Cover architecture, components, data flow, error handling, testing.
- Design for isolation: units with one clear purpose, well-defined interfaces,
  understandable and testable on their own.
- In existing code, follow existing patterns; fold in targeted improvements the
  work actually needs, and nothing else.

**The hard gate:** you present intent and the human approves it. You never move
past the design, and you never implement anything in this session at all — this
session's terminal state is the wrap-up plan, not code. The written spec this
skill would normally commit into the repo is replaced by the wrap-up format in
the frame above: the plan is delivered to Dispatch, not committed here.
