# ModelOps-Templates Architecture Specification

This document provides a technical overview of the architecture, design principles, and parameter contracts for the templates defined in `ModelOps-Templates`.

---

## 🏛️ Architectural Principles

1. **Strict Template Separation**: `ModelOps-Templates` acts as a pure contract/interface layer containing reusable ADO building blocks. It is completely decoupled from application source code, dataset paths, or test scripts.
2. **Framework Agnostic Evaluation**: Pipeline jobs execute evaluation scripts over the wire against deployed GCP endpoints (`ReasoningEngine` resource IDs) without requiring direct access to local model source code.
3. **Artifact-Driven Dependency Injection**: Python environment packaging is isolated in `build_python_environment.yml` and shared across pipeline stages via published pipeline artifacts (`python-venv`).
4. **Keyless Authentication**: All GCP operations authenticate using Workload Identity Federation (WIF) via `setup-wif-template.yml`.

---

## ⚙️ Template Parameter Contracts

### `components/evaluate-metric-based-google/evaluate-metric-based-google.yml`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `jobName` | string | `EvaluateMetricBasedJob` | Name of the ADO job |
| `projectId` | string | *Required* | GCP Project ID |
| `gcpRegion` | string | `us-central1` | GCP region for Vertex AI |
| `agentId` | string | *Required* | Fully qualified ReasoningEngine resource ID |
| `datasetPath` | string | *Required* | Local or GCS path to JSONL dataset |
| `traceBucketName` | string | *Required* | GCS bucket for output evaluation results |
| `metrics` | string | `exact_match,rouge_l` | Comma-separated list of computation metrics |
| `scriptPath` | string | `$(Build.SourcesDirectory)/scripts/evaluate_metric_based.py` | Path to evaluation script |
| `allowMock` | boolean | `false` | Enable fallback mock dataset for dry-runs |

### `components/evaluate-model-based-google/evaluate-model-based-google.yml`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `jobName` | string | `EvaluateModelBasedJob` | Name of the ADO job |
| `projectId` | string | *Required* | GCP Project ID |
| `gcpRegion` | string | `us-central1` | GCP region for Vertex AI |
| `agentId` | string | *Required* | Fully qualified ReasoningEngine resource ID |
| `datasetPath` | string | *Required* | Local or GCS path to JSONL dataset |
| `traceBucketName` | string | *Required* | GCS bucket for output evaluation results |
| `metrics` | string | `groundedness,safety,fluency,coherence` | Comma-separated model-based metrics |
| `safetyThreshold` | float | `0.95` | Score below which pipeline fails |
| `scriptPath` | string | `$(Build.SourcesDirectory)/scripts/evaluate_model_based.py` | Path to evaluation script |
