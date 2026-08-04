# vbonk/skills — Catalog Design

> **At a glance:** This repository is a curated, signed catalog of Claude Code skills. Skills are validated in CI, hash-listed in a manifest, and the manifest is signed with Sigstore cosign (GitHub OIDC keyless). Consumers install via a verifying installer and can independently re-run every check. The catalog — not any website or app — is the trust root.

Status: DESIGN APPROVED 2026-08-03 · This document is the authoritative design. A fresh agent session can execute from this file plus the GitHub issues — no other context required.

## Purpose

Distribute a curated set of production-quality Claude Code skills so that any user — especially users of the vbonk/repo-template starter — can browse, download, and deploy them **with verifiable trust**: provenance (who published it), integrity (nothing modified), and validation (it passed published checks).

## Architecture principles (settled — do not relitigate)

1. **This repo is the trust root.** Certification artifacts (manifest, signatures, audit reports) live here, produced by this repo's CI. Any future website/app front-end is a read-only consumer that renders catalog data; it can never mint or alter certification.
2. **Two trust tiers, never blurred:**
   - `certified` — skills in this catalog: CI-validated, hash-listed, manifest signed via GitHub OIDC. Users can prove provenance and integrity.
   - `validated (local)` — any third-party skill a user checks with the locally-run validator: format/permission checks passed, **no provenance claim**. The validator, installer, and docs must always make the distinction visible.
3. **Skills are executable instructions.** Treat installing a skill with the same seriousness as running a script. Every design decision (signing, verification-before-write, least-privilege review) follows from this.

## Repository structure (target)

```
skills/<name>/SKILL.md      # one directory per skill; optional supporting files beside SKILL.md
manifest.json               # generated in CI — see schema below
validator/                  # standalone validation tool + fixtures
docs/                       # this design, catalog index (generated), trust model, secure-import how-to
.github/workflows/          # validate → manifest → sign → release pipeline
```

**Skill format is `skills/<name>/SKILL.md` directories** (the layout current Claude Code documents; flat `.md` files do not load). Frontmatter uses documented fields only — `name`, `description`, `when_to_use`, `allowed-tools`; NOT the undocumented `triggers:`. Before the first release, runtime-verify in a clean checkout that a directory-form skill actually loads and invokes in current Claude Code (do not trust docs alone).

## manifest.json schema

Per skill entry:

```json
{
  "name": "skeptic",
  "version": "1.0.0",
  "description": "…",
  "tier": "certified",
  "files": { "skills/skeptic/SKILL.md": "sha256:…" },
  "tooling_notes": "requires gh CLI",
  "changelog": "docs/changelogs/skeptic.md"
}
```

Top level: catalog version, generated-at commit SHA, validator version. The manifest is **generated in CI, never hand-edited**, and signed (see pipeline). Any file change in a skill directory requires a version bump — CI enforces this.

## Validator

Standalone tool (shell or node — implementer's choice, must run on macOS bash 3.2 and Linux) with checks:

1. Layout: `skills/<name>/SKILL.md` exists; no stray top-level skill files
2. Frontmatter: parses; documented fields only; `name` matches directory; `description` non-empty
3. `allowed-tools` least-privilege review: WARN (with plain-language explanation) on broad grants — unrestricted Bash, network tools; never a silent pass
4. Injection-pattern scan over all skill files: imperative-override phrases ("ignore previous instructions" class), encoded blobs (base64 walls), URL-exfiltration patterns, instructions to modify AI config files (CLAUDE.md, settings, hooks, workflows)
5. Manifest agreement (when manifest present): file inventory and hashes match
6. Sanity: size/complexity ceilings with WARN

Output: PASS / WARN / FAIL per check with explanations a non-expert can read. **A skipped check reports as a failure, never a pass.** Fixtures in `validator/fixtures/`: valid, malformed-frontmatter, over-permissioned, injection-seeded — CI runs the validator against all fixtures and asserts expected verdicts (the validator itself is tested).

## CI pipeline (per release)

1. Validate every skill (validator, all checks)
2. Generate manifest.json (hashes computed fresh)
3. Sign manifest with cosign keyless via GitHub OIDC (`id-token: write`) — the signing identity IS this repo's CI; that is the provenance claim
4. Tagged release with manifest + signature + bundle; releases immutable
5. Regenerate `docs/` catalog index from manifest

Order matters: nothing is published before signing completes.

## Consumer contract (what installers rely on)

The reference consumer is vbonk/repo-template's `scripts/install-skills.sh` (built in that repo). Contract this catalog guarantees:

- Releases are tagged, immutable, and contain `manifest.json` + cosign signature
- Signature verifies against this repo's GitHub OIDC identity
- Every file listed in the manifest matches its sha256
- Installer behavior (spec'd here so any consumer can implement): fetch pinned release → verify signature → verify hashes → only then write files to `.claude/skills/<name>/`, backing up anything overwritten. **Verification failure = hard stop. Never "install anyway."** Partial failure leaves prior state intact; re-run is idempotent.

## Launch scope

- Curated **5–10 skills**, hand-picked by the maintainer from a private collection (curation is a live task — see issues). Screening criteria: broadly useful, project-agnostic, no personal-workflow coupling (machine paths, personal state files, private-service references), validator-clean.
- `community` tier (third-party listings) is explicitly OUT of scope at launch.
- A future **Level-C certification** (per-version adversarial content audit with published reports) is designed and parked: see [LEVEL-C-CERTIFICATION.md](LEVEL-C-CERTIFICATION.md).
- A future **catalog front-end** (browse/search website) is designed and parked: see [FRONTEND-ASSESSMENT.md](FRONTEND-ASSESSMENT.md).

## Dependencies & sequencing

- vbonk/repo-template is completing a v2.0 modernization that includes migrating its bundled skills to the directory format and shipping the installer + `docs/SKILLS.md` (trust model + secure third-party import how-to referencing this catalog). Coordinate the installer contract above with that work; the catalog can be built in parallel — only the end-to-end install test needs the repo-template side.
- This repo was created from repo-template and still carries template placeholders (README, CLAUDE.md TODOs, stack-specific workflows). Repo initialization/pruning is the first issue.

## Testing (definition of "works")

- Validator fixture suite green in CI
- Release pipeline produces a signed manifest a third party can verify with public cosign tooling (documented command in docs/)
- End-to-end: clean machine → installer → signature+hash verification → skills load and invoke in Claude Code — the served-reality check; a release is not "done" until this passes
