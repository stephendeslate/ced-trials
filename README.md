# SDD Trials 1-10: Specification-Driven Development

Complete artifacts from 10 iterative trials of **Specification-Driven Development (SDD)** — a self-correcting methodology for AI-augmented software engineering. Each trial builds 3 enterprise applications, scores the results, identifies failure modes, and revises the methodology for the next trial.

## The Central Finding

> **No version of SDD achieves both breadth and depth.**

- **v1.0 (Trial 1)** achieved maximum **breadth**: full-stack monorepos (NestJS + Next.js 15 + shadcn/ui), 7-document spec suites, Turborepo, Docker, CI/CD. But audited quality was 6.5-8.0 — RLS was claimed everywhere and implemented nowhere, encryption was plaintext stored as bytes, CI stages ran `echo` commands.

- **v11.0 (Trial 10)** achieved maximum **depth**: 34 tracked failure modes at 100% resolution, 31 conventions, 28 anti-patterns, scores converged at 9.9-10.0. But scope is backend-only — no frontend, no infrastructure, no specs.

The methodology made a deliberate trade at v2.0: drop full-stack scope to get the backend verifiably correct. It never circled back.

## Convergence Pattern

```
Trial:     1    2    3    4    5    6    7    8    9    10
New FMs:   10   3    3    3    3    3    3    3    3    0
Severity:  ████ ███  ██▌  ██   █▌   █    ▌    ▎    ▏    ·
           Struc Corr Comp Sec  Obs  Sem  Hyg  Cov  Doc  Converged
```

Every trial from 2-9 discovered exactly 3 new failure modes. Severity decreased monotonically from structural (build-breaking) to documentation (cosmetic). Trial 10 discovered 0 new failures — the convergence signal.

## The Three Projects

All 10 trials build the same three enterprise applications:

| Project | Domain | Key Challenges |
|---------|--------|----------------|
| **Analytics Engine (AE)** | Multi-tenant embeddable analytics | Data ingestion pipeline, 4 connectors, 7 widget types, SSE real-time, iframe embed API |
| **Escrow Marketplace (EM)** | Two-sided marketplace with payment holds | State machine lifecycle, Stripe Connect, dual-party isolation, dispute resolution |
| **Field Service Dispatch (FSD)** | Field service management SaaS | GPS tracking, route optimization, WebSocket gateways, work order state machine |

Running 3 architecturally distinct projects per trial is deliberate: different projects stress different conventions, exposing methodology gaps that single-project evaluation would miss.

## Tech Stack (Constant Across All Trials)

- **Backend:** NestJS 11, Prisma 6, PostgreSQL 16 with RLS
- **Queue/Cache:** BullMQ + Redis
- **Validation:** class-validator + class-transformer
- **Testing:** Vitest
- **Language:** TypeScript 5.7+

Trial 1 additionally used: Next.js 15, shadcn/ui, Tailwind CSS 4, Recharts, D3, Leaflet, dnd-kit, Zustand, Turborepo + pnpm workspaces. Trials 2-10 dropped all frontend and monorepo tooling.

## Score Trajectory

| Trial | AE | EM | FSD | Avg | Scoring Method |
|-------|-----|-----|-----|-----|----------------|
| 1 (self) | 9.05 | 9.35 | 8.90 | 9.10 | No rubric |
| 1 (audit) | ~6.75 | ~7.75 | ~6.75 | ~7.08 | Independent deep audit |
| 2 | -- | 9.35 | 8.35 | ~8.6 | v2.0 weighted rubric |
| 3 | 8.9 | 8.9 | 9.1 | 8.97 | v3.0 6-category |
| 4 | 8.9 | 9.1 | 9.2 | 9.07 | v4.0 6-category |
| 5 | 9.2 | 9.4 | 9.3 | 9.30 | v5.0 6-category |
| 6 | 9.4 | 9.3 | 9.4 | 9.37 | v6.0 6-dimension |
| 7 | 9.7 | 9.7 | 9.8 | 9.73 | v7.0 6-dimension |
| 8 | 9.83 | 9.92 | 10.0 | 9.92 | v8.0 6-dimension |
| 9 | 9.83 | 9.92 | 10.0 | 9.92 | v9.0 6-dimension |
| 10 | | | | ~9.95 | Converged |

The gap between Trial 1's self-assessed score (9.1) and independent audit (7.08) exposed the need for rigorous scoring. By Trial 7, scores were reliable and monotonically increasing.

## Failure Mode Registry (34 Total)

### Tier 1: Structural (Trial 1) — Build-Breaking
| # | Failure Mode | Resolution |
|---|-------------|------------|
| 1 | Write-but-don't-wire — modules written but never registered | v2.0: C3 gate verifies all guards registered |
| 2 | Spec-satisfying stub — placeholder code passes spec check | v2.0: C5 audit checks implementations line-by-line |
| 3 | Type-escape hatch — 80+ `as any` in production code | v2.0: ESLint `no-explicit-any` rule |
| 4 | Duplicated source of truth — two differing state machine maps | v2.0: Single source of truth rule |
| 5 | Mock-as-production-fallback — graceful degradation masks gaps | v2.0: Mock mode must be explicit opt-in |
| 6 | CLAUDE.md time-capsule — claims written at start, never updated | v2.0: Verification Status table |
| 7 | Phantom CI stage — `echo` commands in pipeline | v2.0: CI integrity verification |
| 8 | Mislabeled test type — E2E tests that mock everything | v3.0: Automated grep detection |
| 9 | Security claim inflation — RLS/encryption claimed, not implemented | v2.0: VERIFY tags + spec-vs-code audit |
| 10 | Cross-tenant data leak — GPS gateway no company check | v3.0: Tenant isolation E2E from C2 |

### Tier 2: Correctness (Trial 2)
| # | Failure Mode | Resolution |
|---|-------------|------------|
| 11 | Silent null return — `findFirst` returning null | v3.0: `findFirstOrThrow` default |
| 12 | Route shadowing — parameterized before static routes | v3.0: Static-first route ordering |
| 13 | Phase C6b skip — methodology revision not produced | v3.0: C6 gate check |

### Tier 3: Completeness (Trial 3)
| # | Failure Mode | Resolution |
|---|-------------|------------|
| 14 | Services-only pattern — no controllers, endpoints unreachable | v4.0: Controller requirement per module |
| 15 | Missing request validation — DTOs without decorators | v4.0: class-validator required |
| 16 | Undocumented isolation variant — dual-party not documented | v4.0: Isolation pattern catalog |

### Tier 4: Security (Trial 4)
| # | Failure Mode | Resolution |
|---|-------------|------------|
| 17 | SQL injection in RLS — `$executeRawUnsafe` with interpolation | v5.0: Parameterized `$executeRaw` |
| 18 | `as never` type escape — bypasses `as any` rule | v5.0: Expanded type assertion audit |
| 19 | Per-controller ValidationPipe — inconsistent, endpoints missed | v5.0: Global ValidationPipe in `main.ts` |

### Tier 5: Observability (Trial 5)
| # | Failure Mode | Resolution |
|---|-------------|------------|
| 20 | Background service observability gap | v6.0: Health/status endpoints |
| 21 | Prisma Json type round-trip — raw `as unknown as T` | v6.0: Typed helpers `toJsonField<T>` / `fromJsonField<T>` |
| 22 | WebSocket gateway testing gap | v6.0: `socket.io-client` test pattern |

### Tier 6: Semantics (Trial 6)
| # | Failure Mode | Resolution |
|---|-------------|------------|
| 23 | Missing Prisma exception filter | v7.0: Global PrismaExceptionFilter |
| 24 | Incorrect HTTP status from state machine | v7.0: `BadRequestException` for invalid transitions |
| 25 | SSE reconnection undocumented | v7.0: `retry` field requirement |

### Tier 7: Hygiene (Trial 7)
| # | Failure Mode | Resolution |
|---|-------------|------------|
| 26 | Empty scaffold directories | v8.0: Scaffold cleanup convention |
| 27 | CLAUDE.md module count drift | v8.0: Filesystem reconciliation |
| 28 | Unannotated `findFirst` | v8.0: Justification annotations |

### Tier 8: Coverage (Trial 8)
| # | Failure Mode | Resolution |
|---|-------------|------------|
| 29 | Residual scaffold dirs beyond modules/ | v9.0: Broad scaffold cleanup |
| 30 | Undocumented test coverage scope | v9.0: Test Coverage Scope section |
| 31 | No integration test guidance | v9.0: Optional Phase C5b |

### Tier 9: Documentation Granularity (Trial 9)
| # | Failure Mode | Resolution |
|---|-------------|------------|
| 32 | Non-standard module composition unannotated | v10.0: Convention 5.30 |
| 33 | DTO-less modules undocumented | v10.0: Convention 5.31 |
| 34 | Context window session continuation | Acknowledged (tooling constraint) |

## Methodology Feedback Loop

Each trial follows a 6-phase process:

```
C1: Read methodology → C2: Produce build plan → C3: Build (with gate checks)
→ C4: Frontend (Trial 1 only) → C5: Audit + score → C6: Revise methodology
```

The version numbering is offset by one: Trial N uses version N.0 as input and produces version (N+1).0 as output. Trial 10 produced v11.0, the terminal version for backend scope.

Key innovations accumulated across versions:

| Version | Key Addition |
|---------|-------------|
| v1.0 | Initial 7-document spec suite, full-stack scope |
| v2.0 | Gate checks (C3), spec-vs-code audit (C5), dropped frontend scope |
| v3.0 | 6-category scoring rubric, automated grep detection |
| v4.0 | Controller requirements, class-validator mandate |
| v5.0 | SQL injection prevention, expanded type assertion audit |
| v6.0 | 6-dimension scoring, observability requirements |
| v7.0 | PrismaExceptionFilter, state machine HTTP status conventions |
| v8.0 | Scaffold cleanup, filesystem reconciliation |
| v9.0 | Test coverage documentation, optional integration testing |
| v10.0 | Module composition annotations, DTO documentation |
| v11.0 | Terminal version: 34 failure modes, 31 conventions, 28 anti-patterns |

## Repository Structure

```
trials/
├── SDD_METHODOLOGY_ANALYSIS_REPORT.md    # Master cross-trial analysis (49 KB)
├── trial1/                                # v1.0 → v2.0 (max breadth)
│   ├── METHODOLOGY.md                     # Input methodology
│   ├── REVISED_METHODOLOGY.md             # Output methodology
│   ├── METHODOLOGY_NOTES.md               # Additional notes
│   ├── analytics-engine/                  # Full-stack monorepo (Turborepo)
│   │   ├── CLAUDE.md                      # Project instructions
│   │   ├── BUILD_PLAN.md                  # Build specification
│   │   ├── apps/web/                      # Next.js 15 frontend
│   │   ├── apps/api/                      # NestJS backend
│   │   ├── apps/embed/                    # Embeddable widget app
│   │   ├── packages/shared/               # Shared types
│   │   └── specs/                         # BRD, PRD, SRS, PVD, WIREFRAMES
│   ├── escrow-marketplace/                # Same structure
│   └── field-service-dispatch/            # Same structure
├── trial2/                                # v2.0 → v3.0 (transition)
│   ├── METHODOLOGY.md
│   ├── REVISED_METHODOLOGY.md
│   ├── analytics-engine/
│   │   ├── CLAUDE.md, BUILD_PLAN.md, BUILD_REPORT.md, C5_AUDIT.md
│   │   └── src/                           # NestJS backend only
│   ├── escrow-marketplace/
│   └── field-service-dispatch/
├── trial3/                                # v3.0 → v4.0
│   └── (same structure, monorepo layout but backend-only)
├── trial4/ through trial9/                # v4.0-v9.0 → v5.0-v10.0
│   └── (minimal: METHODOLOGY.md, REVISED_METHODOLOGY.md,
│         3 projects with CLAUDE.md + BUILD_PLAN.md + src/ + prisma/)
├── trial10/                               # v10.0 → v11.0 (max depth, converged)
│   └── (flat NestJS: src/, prisma/, vitest.config.ts)
└── .gitignore
```

### Per-Trial Contents

| Trial | Input Version | Output Version | Methodology | Projects | Reports | Specs |
|-------|--------------|----------------|-------------|----------|---------|-------|
| 1 | v1.0 | v2.0 | METHODOLOGY.md + NOTES | 3 (full-stack monorepo) | — | Full spec suites (BRD, PRD, SRS, PVD, WIREFRAMES) |
| 2 | v2.0 | v3.0 | METHODOLOGY.md | 3 (transitional) | BUILD_REPORT.md, C5_AUDIT.md | Partial |
| 3 | v3.0 | v4.0 | METHODOLOGY.md | 3 (monorepo layout) | BUILD_REPORT.md, C5_AUDIT.md | Partial |
| 4 | v4.0 | v5.0 | METHODOLOGY.md | 3 (backend-only) | BUILD_REPORT.md | — |
| 5 | v5.0 | v6.0 | METHODOLOGY.md | 3 (backend-only) | BUILD_REPORT.md | — |
| 6 | v6.0 | v7.0 | METHODOLOGY.md | 3 (backend-only) | — | — |
| 7 | v7.0 | v8.0 | METHODOLOGY.md | 3 (backend-only) | — | — |
| 8 | v8.0 | v9.0 | METHODOLOGY.md | 3 (backend-only) | SCORING_REPORT.md | — |
| 9 | v9.0 | v10.0 | METHODOLOGY.md | 3 (backend-only) | SCORING_REPORT.md | — |
| 10 | v10.0 | v11.0 | METHODOLOGY.md | 3 (flat NestJS) | — | — |

## What's Next: Layered Convergence (Trials 11-49)

The next trial series extends v11.0's depth mechanisms to every breadth domain v1.0 covered:

```
Layer 0 (Complete): Backend API         — v1.0-v11.0, 34 failure modes, converged
Layer 1:            Integration Testing — extend with required E2E + real DB
Layer 2:            Frontend            — Next.js 15, component testing, accessibility
Layer 3:            Specifications      — spec-to-code traceability
Layer 4:            Infrastructure      — Docker, CI/CD, deployment
Layer 5:            Monorepo            — cross-package coordination
Layer 6:            Security (broad)    — OWASP, secrets, CORS/CSP
Layer 7:            Performance         — benchmarking, load testing
Layer 8:            Monitoring          — logging, alerting, APM
```

Each layer runs trials until convergence (0 new failure modes), then the next layer begins. All layers inherit previous conventions and failure modes.

## How It Was Built

All 10 trials were conducted using [Claude Code](https://claude.ai/code) with Claude Opus as the primary model. The methodology document is the only input — no manual coding, no human-written source code. Each trial's source code was generated entirely by the AI agent following the methodology's phased process.

Session logs (1,584 subagent invocations across 206 sessions) are available in the companion [SDD Dashboard](https://github.com/stephendeslate/sdd-dashboard) project, which provides real-time visualization of trial data, convergence patterns, and agent activity.

## License

MIT
