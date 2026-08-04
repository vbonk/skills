# vbonk/skills — Execution Plan

> Generated: 2026-08-03 | Author: Anthony Velte (@vbonk)
> Status: APPROVED — ready for execution (design: [docs/PROJECT-DESIGN.md](docs/PROJECT-DESIGN.md))
> This file + the GitHub issues are the complete pickup context for any agent session.

## What this project is

A curated, signed catalog of Claude Code skills. Skills validated in CI, hash-listed in a signed manifest (Sigstore cosign, GitHub OIDC keyless), installed via a verifying installer. The catalog is the trust root. Full design: [docs/PROJECT-DESIGN.md](docs/PROJECT-DESIGN.md). Parked follow-ons: [Level-C certification](docs/LEVEL-C-CERTIFICATION.md), [front-end](docs/FRONTEND-ASSESSMENT.md).

## Phases (tracked as GitHub issues — the issue list is the task board)

1. **Repo initialization** — prune template placeholders (this repo was created from vbonk/repo-template): README rewritten for the catalog, CLAUDE.md filled, stack-specific workflow stubs removed, labels created. NOTE: do not use the template's `scripts/labels.sh` until repo-template v2.0 lands (known-broken arg parsing) — create labels with direct `gh label create` commands.
2. **Format verification spike** — runtime-verify in a clean checkout that `skills/<name>/SKILL.md` directory skills load and invoke in current Claude Code. Gate for everything downstream.
3. **Manifest schema + versioning ADR** — commit the schema from the design; ADR for version-bump enforcement.
4. **Validator + fixtures** — all six check classes, fixture suite, CI job. Skipped check = fail.
5. **Signing pipeline** — validate → generate manifest → cosign sign → immutable tagged release; third-party-verifiable (documented verify command).
6. **Launch set curation + migration** — 5–10 skills hand-picked WITH THE MAINTAINER (interactive; screening criteria in design doc). Migrate to directory format, version, changelog each.
7. **Docs** — trust model, secure-import how-to, catalog index generation; coordinate the installer contract with vbonk/repo-template's v2.0 work (their `scripts/install-skills.sh` + `docs/SKILLS.md`).
8. **End-to-end release test** — clean environment, installer, signature+hash verify, skills load. Release v1.0.0 of the catalog only when this passes.

## Coordination notes

- vbonk/repo-template v2.0 (separate effort, separate session) ships the consumer installer and migrates its own bundled skills to directory format. The installer CONTRACT is fixed in the design doc — build to it; only the end-to-end test needs both sides.
- Maintainer input needed at: launch-set curation (phase 6) and any trust-model wording changes (the two-tier certified/validated distinction is settled — do not weaken it).

## Not in scope

- `community` (third-party) tier listings
- Level-C adversarial audit (parked, spec'd)
- Any front-end (parked, spec'd — assessment-first re: agent-toolbox reuse)
