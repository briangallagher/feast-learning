# UI Guide: Visual Interfaces for Feast on RHOAI

This guide covers all the visual interfaces available when working with Feast, from the native Feast UI to lineage visualization and platform-level dashboards.

## 1. Feast Web UI

### What It Is

A React-based web interface for browsing feature store metadata: entities, feature views, data sources, feature services, and saved datasets.

### Status

| Environment | Status | Notes |
|-------------|--------|-------|
| Upstream Feast | Beta | Port 8888 by default via `feast ui` |
| RHOAI (operator-managed) | Available | Exposed via Route when UI is enabled in FeatureStore CR |

### How to Access on RHOAI

```bash
# Check if Feast UI route exists
oc get routes -n <your-namespace> | grep feast

# The operator creates a route like:
# feast-ui-<namespace>.apps.<cluster-domain>
```

If deployed via the operator, enable the UI in the FeatureStore CR:

```yaml
apiVersion: feast.dev/v1alpha1
kind: FeatureStore
metadata:
  name: my-feast
spec:
  feastProject: my_project
  services:
    ui: {}  # Enables the UI deployment
```

### What You Can See

- **Feature Views**: browse all registered feature views with schema, entities, TTL
- **Entities**: list of all entities with join keys and types
- **Data Sources**: registered data sources (batch and stream)
- **Feature Services**: groups of features served together
- **Saved Datasets**: profiled training datasets

### Known Gaps

- **⚠️ GAP**: Beta maturity — stability and completeness gaps exist
- **⚠️ GAP**: Read-only — cannot create or modify permissions from the UI (Issue [#6192](https://github.com/feast-dev/feast/issues/6192))
- **⚠️ GAP**: No lineage visualization in Feast UI itself — lineage is in Marquez UI (see below)

---

## 2. Feast Lineage UI (via OpenLineage + Marquez)

### What It Is

When OpenLineage is enabled, Feast emits lineage events during `feast apply` and `feast materialize`. These events flow to a lineage backend (Marquez) which provides its own visualization UI.

### Status

| Component | Status | Notes |
|-----------|--------|-------|
| Feast OpenLineage emission | GA upstream (v0.40+) | Enable in `feature_store.yaml` |
| Marquez UI | Not in RHOAI | Deploy separately; standalone webpack app |
| Unified lineage in RHOAI Dashboard | Planned (Pillar 5) | Workshop Decision #1, no timeline yet |

### How to Access

Marquez UI runs as a separate deployment (default port 3000):

```bash
# If Marquez is deployed in your namespace:
oc get routes -n <your-namespace> | grep marquez

# Or port-forward:
oc port-forward svc/marquez-web 3000:3000 -n <your-namespace>
```

### What You Can See

- **Lineage Graph**: visual DAG showing data sources → feature views → feature services
- **Job History**: when materialization ran, duration, status
- **Dataset Versions**: schema evolution over time
- **Cross-System Lineage**: if other producers (Spark, KFP) also emit to the same Marquez instance

### The Lineage Chain

```
Data Source (e.g., PostgreSQL table)
    ↓ [feast apply — creates lineage edges]
Feature View (transformation definition)
    ↓ [feast materialize — records data movement]
Online Store (Redis/PostgreSQL)
    ↓ [future: model training consumption]
    ↓ [future: inference serving access]
```

### What's NOT Covered

- **⚠️ GAP**: Lineage stops at the feature store boundary — no tracking into model training or serving
- **⚠️ GAP**: No column-level lineage (feature view granularity only)
- **⚠️ GAP**: No point-in-time lineage queries ("what did the graph look like last Tuesday?")
- **⚠️ GAP**: Marquez has a 19-month release gap (v0.50.0 since Oct 2024), zero built-in auth, single-maintainer risk

### Data Strategy Note

🗺️ The Jan 2026 workshop decided on E2E lineage via OpenLineage as Decision #1. The vision is to rebase the Feast lineage visualization on the OpenLineage standard and build a unified lineage view in the RHOAI Dashboard (Pillar 5). Today, Marquez is the collector backend under POC evaluation.

---

## 3. RHOAI Dashboard (AI Hub)

### What It Is

The main RHOAI web console — a React 18 + PatternFly v6 application with Module Federation architecture.

### Feast-Relevant Sections

| Section | What You See | Status |
|---------|-------------|--------|
| Data Science Projects | Namespace where Feast is deployed | GA |
| Workbenches | Where you run notebooks that use Feast | GA |
| Model Registry | Models trained with Feast features | GA |
| Data Connections | S3/storage connections (not yet Feast-aware) | GA (limited) |
| **Data Hub** (future) | Unified view: features, datasets, vector sources, connections | Planned (Workshop Decision #3) |

### How to Access

```bash
# Get the dashboard URL
oc get routes -n redhat-ods-applications | grep rhods-dashboard
```

### Future: Data Hub UI

Workshop Decision #3 envisions a "Data Assets" section in AI Hub showing:
- Feast Registry (feature views, entities)
- OGX Sources Registry (vector stores, documents)
- MLflow Registry (experiments, models)
- External Connections
- Unified Lineage Visualization

**⚠️ GAP**: This does not exist today. Feast UI, MLflow UI, and Marquez UI are all separate applications with no unified view.

---

## 4. Prometheus / Grafana (Observability)

### What It Is

Feast exposes native Prometheus metrics (added in v0.61.0+). On OpenShift with monitoring enabled, these are scraped automatically via ServiceMonitor.

### Status

| Feature | Status | Notes |
|---------|--------|-------|
| Prometheus metrics | GA upstream (v0.61.0+) | Latency, freshness, materialization health |
| ServiceMonitor auto-generation | GA upstream (v0.62.0) | Auto-discovered by OpenShift monitoring |
| Grafana dashboards | Community | No OOTB dashboard in RHOAI |

### Available Metrics

| Metric | What It Measures |
|--------|-----------------|
| `feast_feature_server_request_latency` | Feature serving response time |
| `feast_feature_freshness` | Time since last materialization per feature view |
| `feast_materialization_duration` | How long materialization jobs take |
| `feast_odfv_transformation_duration` | On-Demand Feature View compute time |

### How to Access

```bash
# Check if ServiceMonitor exists
oc get servicemonitors -n <your-namespace> | grep feast

# Access via OpenShift monitoring (Observe → Metrics in console)
# Or port-forward Prometheus:
oc port-forward svc/prometheus-k8s 9090:9090 -n openshift-monitoring
```

### Gaps

- **⚠️ GAP**: No alerting when materialization fails or features go stale beyond TTL
- **⚠️ GAP**: No pre-built Grafana dashboard for Feast in RHOAI
- **🔮 UPSTREAM**: Go server metrics and tracing for HTTP/gRPC added in v0.61.0

---

## 5. MLflow UI (Related)

### Why It Matters for Feast

MLflow tracks experiments and models. When you train a model using Feast features, the MLflow UI shows:
- Which experiment used which feature set
- Model performance metrics over time
- Model versions and deployment status

### The Gap

**⚠️ GAP**: There is no link between "this model was trained with these Feast features" visible in any UI today. You must manually log feature metadata to MLflow or rely on OpenLineage to connect the chain.

**🗺️ DATA STRATEGY**: The unified Data Hub vision (Pillar 5) would connect feature store metadata with experiment tracking to answer "which features were used to train this model?" — but this is future state.

---

## Summary: Where to Look

| I want to... | Go to... | Status |
|--------------|----------|--------|
| Browse feature definitions | Feast UI | Available (Beta) |
| See data lineage graph | Marquez UI | Deploy separately |
| Manage RHOAI resources | RHOAI Dashboard | GA |
| Monitor feature freshness | Prometheus/Grafana | Metrics available, no OOTB dashboard |
| See model → feature relationships | Not available | Future (Pillar 5 Data Hub) |
| Manage Feast RBAC permissions | Not available (code only) | Upstream Issue #6192 |
