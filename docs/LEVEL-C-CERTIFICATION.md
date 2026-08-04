# Level-C Certification — Deep Content Audit (Parked Project)

> **At a glance:** Upgrade the `certified` tier from "signed + statically validated" to "signed + statically validated + adversarially audited, with the audit report published per skill version." Parked until the catalog ships at Level B. This doc is self-contained for a future session.

Status: PARKED · Prereq: catalog live with signed releases (see [PROJECT-DESIGN.md](PROJECT-DESIGN.md))

## What Level C adds

Every published skill VERSION passes an adversarial content audit before certification, and the audit report ships in the signed release bundle — "pre-audited" becomes literal: a user reads exactly what was tested before trusting the skill.

## Scope

1. **Adversarial audit harness** (runs in this repo's CI, per skill per version):
   - Prompt-injection red-team: seeded payloads embedded in skill body/frontmatter/supporting files; an agent-under-test executes the skill against tripwire tasks; harness asserts no exfiltration, no AI-config modification, no instruction override
   - Behavior-vs-description: skill's actual behavior matches its `description`/`when_to_use` claims (LLM-judge with a published rubric)
   - Tool-permission exercise: skill runs in a sandbox logging every tool call; log diffed against declared `allowed-tools`
2. **Payload corpus**: seeded from AI Defense Lab (https://defenselab.app/agent) injection/kill-chain attack patterns. Corpus is versioned; every report states its corpus version.
3. **Report format**: per skill+version markdown — checks run, payloads survived, judge scores + rubric, corpus version, harness version, judge model version (pinned, so judge drift can't silently change the standard). Hash-listed in the signed manifest like any other file.
4. **Re-certification triggers**: any skill file change → version bump → full C pass before release. Corpus/harness upgrade → batch re-run, reports refreshed.
5. **Badging**: catalog index (and any future front-end) distinguishes C-audited versions from B-only.

## Open questions for the implementing session

- Harness runtime: Claude Agent SDK scripted runs vs. CI scripts driving headless agent invocations (cost/determinism trade)
- Judge rubric calibration and inter-run stability
- Whether Defense Lab gains a public "skill audit" mode (cross-product tie-in)
- CI cost ceiling per release; cadence for corpus-upgrade batch re-runs

## Definition of done

A skill version cannot carry `tier: certified` at Level C without a current audit report in the signed manifest; users can open the report from the catalog docs; the trust-model doc states precisely what C does and does not prove.
