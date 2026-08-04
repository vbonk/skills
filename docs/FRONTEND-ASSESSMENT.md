# Catalog Front-End — Assessment First, Then Build (Parked Project)

> **At a glance:** A future browse/search website for this catalog. Before any build: an explicit assess-reuse-vs-start-fresh decision about the maintainer's existing "agent-toolbox" project (a stale-but-substantial Next.js 15 + Supabase platform with hybrid semantic search). Parked until the catalog ships. Self-contained for a future session.

Status: PARKED · Prereq: catalog live with manifest + releases

## Architectural constraint (settled — do not relitigate)

The front-end is a READ-ONLY DOWNSTREAM CONSUMER of this catalog. It renders manifest data, tier badges, audit reports, and search. It is never the distribution channel and never the trust root — certification lives in this repo's CI, so a compromised front-end cannot forge certification. Install instructions shown in the app always point at the verifying installer + catalog releases.

## Reuse posture (maintainer decision, 2026-08-03): parts-first, revival is NOT the default

The maintainer has a prior project, **agent-toolbox** (local project, private): Next.js 15 + Supabase, hybrid pgvector+FTS semantic search, an MCP server with 18 tools, workflow designer, admin dashboard — alpha maturity, stale ~6 months, 314 TypeScript files. Reviving that full stack is likely a heavy lift (deps, auth, DB all need re-validation).

**Default posture: extract inspiration and usable elements** — the hybrid search implementation, UI patterns, admin scaffolding — rather than resurrect the platform. **The implementing session's first task is an explicit assess-reuse-vs-start-fresh decision**, where "start fresh and strip agent-toolbox for parts" is the leading candidate. All scope below applies to whichever chassis wins.

## Scope sketch

1. Ingest: sync manifest.json + generated docs from catalog releases (webhook or scheduled pull)
2. Browse/search over skill names/descriptions/bodies
3. Skill detail: SKILL.md render, version history, tier badge (B vs C-audited once Level C exists), audit report links, copy-paste install command
4. Fully public read-only v1 (leaning: no auth at all)
5. YAGNI check: the manifest is one JSON file — a static site may beat a database-backed app entirely; the semantic-search value must justify any backend

## Open questions for the implementing session

- Static site vs. app-with-backend (see YAGNI check above)
- Domain/branding: standalone vs. section under an existing property; coherence with AI Defense Lab (https://defenselab.app) as a public story
- Whether the maintainer's older Python "app-agents" project contributes anything (initial read: out of scope)

## Definition of done (to refine)

Public site rendering live catalog data with search, detail pages, correct tier badging; zero write-path from site to catalog; passes a security review before public link-sharing.
