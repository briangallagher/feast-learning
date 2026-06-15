# Data Strategy Context

This tutorial exists to build deep understanding of Feast in the context of the RHOAI data strategy. This document maps each module to the strategic context so you always know *why* something matters, not just *how* it works.

## The Data Strategy in Brief

Red Hat AI's unified data strategy (Jan 2026 workshop) addresses the critical gap: RHOAI had no coherent "data story." The core thesis: **models are a commodity — data + applications are the competitive advantage.**

Five pillars, four workshop decisions, and Feast sits at the centre of Pillar 3 (Data Abstraction) while touching Pillars 2 and 4.

## Feast's Role in the Strategy

Feast is the **predictive AI** data abstraction layer. It is NOT positioned as:
- A data catalog (that's Unity Catalog)
- A lineage system (that's OpenLineage + Marquez)
- A data gateway (resolved: Feast is a feature store, not a gateway)
- The only path for GenAI/RAG (Milvus + OGX is the primary RAG stack)

Feast IS:
- The feature store for online/offline feature serving
- An OpenLineage emitter (contributing to E2E lineage)
- A potential unified retrieval layer for *existing Feast adopters* who also need RAG
- Integrated with Ray for distributed compute
- The first component with RBAC that integrates with RHOAI identity

## Module → Pillar Mapping

| Module | Primary Pillar | Strategic Relevance |
|--------|---------------|-------------------|
| 01-05 (Foundations) | Pillar 3 | Core feature store — the predictive AI path |
| 06 Ray Compute | Pillar 2 | Feast+Ray productization; dominant customer demand |
| 07 OpenLineage | Pillar 4 | Workshop Decision #1 — E2E lineage via OpenLineage |
| 08 RAG/Vector | Pillar 3 | GenAI unified retrieval for existing Feast adopters |
| 09 MCP Agents | Pillar 5 | Governed context layer for AI agents |
| 10 RBAC | Pillar 4 | Workshop Decision implicit — governance model |
| 11 KFP | Pillar 4 | Pipeline orchestration backbone |
| 12 Operator | Cross-cutting | How Feast ships in RHOAI product |
| 13 UI/Observability | Cross-cutting | Making the invisible visible |
| 14 Credit Scoring | All | Scenario A from data-strategy-proposal |
| 15 Knowledge Retrieval | Pillar 3 | Scenario B (Feast variant) from data-strategy-proposal |

## Four Workshop Decisions (Jan 2026)

1. **E2E Lineage via OpenLineage** → Module 07
2. **Centralized External Connection Auth** → Module 05 (acknowledged as gap in Feast CRD)
3. **Data Hub UI in AI Hub** → Module 13 (Feast UI as one component of future Data Hub)
4. **Credentials Auto-Mounting** → Module 05, 12 (gap: CRD uses inline Secrets today)

## Key Gaps to Be Aware Of

These are documented in-context throughout the notebooks, but the headline items:

| Gap | Impact | Where Covered |
|-----|--------|--------------|
| FeatureStore CRD is v1alpha1 | Breaking changes possible between versions | Module 12 |
| Ray/Spark compute not in CRD | Must configure manually; not operator-managed | Module 06 |
| RAG/Vector at Alpha | No GA timeline; API may change | Module 08 |
| MCP not downstream in RHOAI | Available upstream only | Module 09 |
| RBAC requires Python code | No admin UI for permissions | Module 10 |
| Registry metadata dominates latency | >50% of serving time is metadata resolution | Module 03 |
| No data connection refs in CRD | Uses inline Secrets, not platform connections | Module 12 |
| Streaming at Alpha | Push-only, no continuous aggregation | Module 04 |

## Source Documents

This tutorial draws from:
- `data-strategy-proposal/RHAI-data-strategy-proposal.md` — five-pillar strategy (v4.0)
- `data-oss-landscape/feature-store-analysis/` — Feast strengths, gaps, comparisons
- `data-strategy-proposal/scenarios/scenario-a-credit-scoring/` — predictive AI reference
- `data-strategy-proposal/scenarios/feast-use-case-fit-analysis/` — Feast positioning analysis
- `work-knowledge/knowledge/rhoai/data-strategy/data-strategy.md` — workshop outcomes
