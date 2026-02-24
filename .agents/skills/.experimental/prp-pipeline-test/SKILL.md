---
name: "codex-prp-pipeline"
description: "Encodes the Codex-flavored PRP pipeline, describing the phased workflow, guardrails, and traceability to run PRP work through Codex agents."
---


# Codex PRP Pipeline Skill

## When to use

- You are running a PRP-style feature or fix that must proceed through the multi-phase lifecycle (create, quality, execute, test, validate) without manual orchestration.
- You need full traceability and guardrail enforcement as described in the [Codex Agents SDK guide](https://developers.openai.com/codex/guides/agents-sdk).
- You want to reuse the same procedural logic across requests so multiple Codex sessions adopt the same checkpoints and retries.

## Workflow

1. **Intake & Project Manager Setup**: The Codex MCP manager agent collects the feature goal, PRP path, and any required metadata, then chooses the downstream agents (research, quality, implementation, tester, validation gate). Every call and response is logged as a skill trace entry.
2. **Create Phase (Research Agent)**: The manager directs the research agent to gather schema/incident/context, scan `docs-ai/contents/docs/PRPs/<feature>`, generate `prp-<feature>.md` plus `TASKS.md`, and verify resulting artifacts exist before allowing the next handoff.
3. **Quality Phase (Quality Agent)**: The manager insists the quality agent validate template completion, references, and the "No Prior Knowledge" checklist, producing a score and gating further progress on a minimum 8/10 rating.
4. **Execute Phase (Implementation Agent)**: The manager hands the approved PRP and tasks to the implementation agent, which follows the step-by-step tasks, regenerates Supabase types when necessary, updates `TASKS.md`, and records guardrail compliance after each task.
5. **Test Phase (Tester Agent)**: The manager calls the tester agent to run lint/type checks, route validations, Playwright suites, and builds. The skill records pass/fail results, and the manager requires a clean slate before continuing.
6. **Validate Phase (Validation Gate Agent)**: The manager invokes the gate agent documented in `.claude/commands/PRPs/README-PRP-VALIDATION-GATE-AGNET.md` to rerun the validation checklist, confirm `[x]` tasks, and emit the final structured report.
7. **Reporting & Retry Logic**: If any phase fails, the manager automatically retries once with added failure context, then surfaces the detailed error plus trace logs. Successful runs produce a published summary referencing the PRP path, guardrail outcomes, and the Codex trace IDs.

## Agent Handoff Guidelines

- Always store trace metadata (feature name, PRP path, guardrail status, command outputs) as described in the Agents SDK guide; this lets reviewers replay what happened across agents.
- Treat each phase as a separate agent invocation to avoid context overload; the manager should only pass the current phase’s input/output and the accumulated trace.
- Document every gate condition (artifact exists, quality ≥8, tests pass, validation PASSED) in the skill so the manager can compare actual outcomes to expected checkpoints.
- When the validation gate reports a failure, route the skill back to the implementation agent with the reported issues, rerunning tests only once per gate failure.

## Skill Activation

- Invoke this skill with a command like `/prp-pipeline codex-feature` so the manager immediately knows which PRP artifacts to target.
- If the feature already exists, the manager can instead trigger the audit + fix PRP variant, but still rely on the same phased guardrails within this skill.
- Use Codex multi-agent best practices: keep the manager in control, treat downstream agents as tool calls, and store every transition in the skill trace to satisfy compliance and debugging needs.

## References

- [Codex Agents SDK guide](https://developers.openai.com/codex/guides/agents-sdk)
- [Codex Skills creation guide](https://developers.openai.com/codex/skills/create-skill)
