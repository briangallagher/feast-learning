# Architecture: Feast in the RHOAI Data Strategy

## Five-Pillar Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  Consumers: Enterprise Copilots │ AI Agents │ Domain Apps │ BI  │
├─────────────────────────────────────────────────────────────────┤
│  Pillar 5: Unified Data & AI Application Experience             │
│  (OGX, Dashboard/AI Hub, MCP)                                   │
│  Module 09 (MCP), Module 13 (UI)                                │
├─────────────────────────────────────────────────────────────────┤
│  Pillar 4: Orchestration, Lineage & Governance                  │
│  (KFP, MLflow, OpenLineage, Marquez, TrustyAI)                  │
│  Module 07 (OpenLineage), Module 10 (RBAC), Module 11 (KFP)    │
├─────────────────────────────────────────────────────────────────┤
│  Pillar 3: Data Abstraction Layer                               │
│  ┌─────────────────────┐  ┌─────────────────────────────┐      │
│  │ Predictive AI       │  │ Knowledge Retrieval          │      │
│  │ FEAST               │  │ Milvus + OGX + Docling       │      │
│  │ Modules 01-05, 14   │  │ Module 08 (Feast variant),15 │      │
│  └─────────────────────┘  └─────────────────────────────┘      │
├─────────────────────────────────────────────────────────────────┤
│  Pillar 2: Compute Engine Strategy                              │
│  (Ray Data, Spark, OGX file_processor, Docling)                 │
│  Module 06 (Ray)                                                │
├─────────────────────────────────────────────────────────────────┤
│  Pillar 1: Data Ingestion & Connectivity                        │
│  (Connectors, Credentials Registry, CDC)                        │
│  Module 05 (config/connections), Module 12 (operator)           │
├─────────────────────────────────────────────────────────────────┤
│  Foundation: OpenShift │ RBAC/OIDC │ Object Storage │ KubeRay   │
└─────────────────────────────────────────────────────────────────┘
```

## Feast Component Architecture on RHOAI

When deployed via the Feast Operator on RHOAI, the architecture looks like:

```
┌──────────────────────────────────────────────────────────────┐
│  OpenShift Namespace (Data Science Project)                   │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  FeatureStore CR (v1alpha1)                          │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │    │
│  │  │ Registry │  │ Online   │  │ Offline Store     │  │    │
│  │  │ (SQL/    │  │ Store    │  │ (PostgreSQL/      │  │    │
│  │  │  file)   │  │ (Redis/  │  │  Snowflake/       │  │    │
│  │  │          │  │  PG)     │  │  file)            │  │    │
│  │  └──────────┘  └──────────┘  └──────────────────┘  │    │
│  │                                                      │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │    │
│  │  │ Feature  │  │ Feast UI │  │ Materialization   │  │    │
│  │  │ Server   │  │ (Beta)   │  │ (CronJob)        │  │    │
│  │  │ (gRPC/   │  │          │  │                   │  │    │
│  │  │  HTTP)   │  │          │  │                   │  │    │
│  │  └──────────┘  └──────────┘  └──────────────────┘  │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────────────┐  │
│  │ KubeRay  │  │ KFP      │  │ Workbench (Notebook)     │  │
│  │ (compute)│  │ (orch)   │  │ feast SDK client          │  │
│  └──────────┘  └──────────┘  └──────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

## Data Flow Patterns

### Pattern 1: Predictive AI (Scenario A — Credit Scoring)

```
Raw Data (PostgreSQL/S3)
    │
    ▼ [Batch Source — feast data source definition]
Feature View (transformation logic)
    │
    ├──▶ get_historical_features() → Training DataFrame
    │         │
    │         ▼
    │    Model Training (scikit-learn/XGBoost)
    │         │
    │         ▼
    │    MLflow (experiment tracking)
    │
    ▼ [feast materialize — moves data to online store]
Online Store (Redis/PostgreSQL)
    │
    ▼ [get_online_features() — low-latency serving]
Model Inference (KServe / custom endpoint)
```

### Pattern 2: Knowledge Retrieval via Feast (for existing adopters)

```
Documents (PDF, DOCX)
    │
    ▼ [Docling — document processing]
Chunks + Embeddings
    │
    ▼ [feast materialize — stores in vector-enabled online store]
Milvus / pgvector (online store)
    │
    ▼ [retrieve_online_documents_v2 — vector similarity search]
RAG Context → LLM Prompt
```

**📍 RHOAI STATUS**: This pattern uses Alpha-maturity upstream features. The primary RAG stack in RHOAI is OGX + Milvus (no Feast in the path). Feast RAG is for existing adopters who want a unified retrieval layer.

## Key Integration Points

| Integration | How It Works | Module |
|-------------|-------------|--------|
| Feast → OpenLineage | `feature_store.yaml` config enables automatic event emission | 07 |
| Feast → Ray | Ray compute engine for distributed materialization | 06 |
| Feast → KFP | Python SDK calls in KFP pipeline steps | 11 |
| Feast → RBAC/OIDC | Kubernetes auth backend reads RHOAI DSP RoleBindings | 10 |
| Feast → Prometheus | Native metrics + ServiceMonitor auto-generation | 13 |
| Feast → MCP | FastAPI endpoints exposed as MCP tools for agents | 09 |
| Feast → MLflow | Metric charts; no deep lineage integration yet | 13 |

## What's Missing (Platform Gaps)

| Expected Integration | Current State | Impact |
|---------------------|---------------|--------|
| Feast → Platform Connections | CRD uses inline K8s Secrets, not platform connection objects | Each component manages credentials independently |
| Feast → Unity Catalog | No integration | No cross-system data catalog |
| Feast → RHOAI Dashboard | Separate UI, not embedded in AI Hub | No unified data asset view |
| Feast ← MLflow lineage | MLflow doesn't emit OpenLineage events yet | Lineage chain breaks at model training boundary |
| Feast → Auto-mounted credentials | Not implemented | Workshop Decision #4 not yet delivered |
