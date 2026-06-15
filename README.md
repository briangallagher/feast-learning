# Feast Learning Path

A structured tutorial for learning Feast on Red Hat OpenShift AI (RHOAI), covering core feature store concepts through to data strategy-relevant deep dives.

## Approach

- **Cluster-first**: exercises target RHOAI on OpenShift (Feast operator, FeatureStore CR, KubeRay, KFP)
- **RHOAI Feast as baseline**: uses the productized Feast distribution shipped in RHOAI. Upstream-only features are called out explicitly with status (Alpha, not yet downstream, etc.)
- **Gaps acknowledged**: known limitations, performance issues, and roadmap items are documented in context rather than hidden
- **Data strategy aligned**: modules map to the five-pillar RHOAI data strategy, with explicit links to strategic relevance

## Prerequisites

- Access to an RHOAI cluster (3.4+ with Feast operator enabled)
- Python 3.11+ workbench (Standard Data Science or custom)
- `oc` CLI authenticated to the cluster
- Familiarity with Python, pandas, and basic ML concepts

## Module Structure

### Part 1: Foundations

| # | Module | What You Learn | Key Concepts |
|---|--------|---------------|--------------|
| 01 | [Core Concepts](modules/01-core-concepts/) | What a feature store solves, Feast architecture, key abstractions | Entities, Feature Views, Data Sources, Registry |
| 02 | [Offline Store & Training](modules/02-offline-store/) | Historical feature retrieval for model training | `get_historical_features()`, point-in-time joins, entity DataFrames |
| 03 | [Online Store & Serving](modules/03-online-store/) | Low-latency feature serving for inference | Materialization, `get_online_features()`, TTL, freshness |
| 04 | [Feature Engineering](modules/04-feature-engineering/) | Transformations and computed features | On-Demand Feature Views, batch/push sources, streaming (Alpha) |
| 05 | [Configuration Deep Dive](modules/05-configuration/) | Multi-backend setup, registry options, operator config | `feature_store.yaml`, FeatureStore CRD, Helm values |

### Part 2: Data Strategy Deep Dives

| # | Module | What You Learn | Data Strategy Pillar |
|---|--------|---------------|---------------------|
| 06 | [Distributed Compute with Ray](modules/06-ray-compute/) | Feast + Ray for materialization and embedding at scale | Pillar 2: Compute Engine Strategy |
| 07 | [OpenLineage & Lineage UI](modules/07-openlineage/) | Data lineage tracking, Marquez backend, lineage visualization | Pillar 4: Orchestration, Lineage & Governance |
| 08 | [RAG & Vector Search](modules/08-rag-vector/) | Embedding features, vector DBs, unified retrieval for GenAI | Pillar 3: Data Abstraction (knowledge retrieval variant) |
| 09 | [MCP for AI Agents](modules/09-mcp-agents/) | Feast as governed context layer for AI agents | Pillar 5: Unified Data & AI Experience |
| 10 | [RBAC & Governance](modules/10-rbac/) | Access control, OIDC auth, multi-tenancy patterns | Pillar 4: Governance |
| 11 | [KFP Integration](modules/11-kfp-integration/) | Feature pipelines, scheduled materialization, pipeline components | Pillar 4: Orchestration |
| 12 | [Operator & Deployment](modules/12-operator/) | FeatureStore CRD, Helm, operator-managed lifecycle on OpenShift | RHOAI productization |
| 13 | [UI & Observability](modules/13-ui-observability/) | Feast UI, lineage visualization, Prometheus metrics, dashboards | Cross-cutting |

### Part 3: End-to-End Scenarios

| # | Module | Scenario | Exercises All Of |
|---|--------|----------|-----------------|
| 14 | [Credit Scoring (Predictive AI)](modules/14-scenario-credit-scoring/) | Full ML lifecycle: ingest → features → train → serve → lineage | Pillars 1-4, Feast core |
| 15 | [Knowledge Retrieval (Feast + RAG)](modules/15-scenario-knowledge-retrieval/) | Feast as unified retrieval: structured features + document embeddings | Pillar 3 (GenAI variant) |

## Priority Learning Path

If short on time, follow this order for maximum data strategy relevance:

1. **Modules 01-03** — essential foundations (can't skip)
2. **Module 07** — OpenLineage (the #1 data strategy workshop decision)
3. **Module 06** — Ray compute (dominant customer demand signal)
4. **Module 13** — UI & Observability (see the system in action)
5. **Module 08** — RAG/Vector (the GenAI story)
6. **Module 14** — End-to-end scenario to tie it together

## Conventions Used in Notebooks

Throughout the notebooks you'll see callout boxes:

- **📍 RHOAI STATUS**: Current state in the RHOAI product (GA, Tech Preview, not yet downstream)
- **🔮 UPSTREAM**: Feature available in upstream Feast but not yet in RHOAI
- **⚠️ GAP**: Known limitation or gap — links to relevant issues/RFEs
- **🗺️ DATA STRATEGY**: How this connects to the five-pillar data strategy
- **🖥️ UI**: Where to find a visual interface for what you just did programmatically

## Related Resources

- [Data Strategy Context](CONTEXT.md) — how this tutorial connects to the RHOAI data strategy
- [UI Guide](docs/ui-guide.md) — all the visual interfaces: Feast UI, lineage, dashboard, Prometheus
- [Architecture Overview](docs/architecture.md) — five-pillar mapping and component relationships
- [Feast upstream docs](https://docs.feast.dev/)
- [Feast on RHOAI blog](https://www.redhat.com/en/blog/feast-open-source-feature-store-ai)
