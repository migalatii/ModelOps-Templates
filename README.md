# ModelOps-Templates

Welcome to **ModelOps-Templates**! This repository serves as the centralized source of truth for raw, reusable Azure DevOps (ADO) pipeline templates and component job definitions for developing and evaluating AI applications on Google Cloud Platform (GCP).

> [!IMPORTANT]
> **REPOSITORY CONSTRAINT**: This repository is strictly restricted to **YAML templates and pipeline definitions only** (`.yml` / `.yaml`). No Python execution scripts, dataset files, or configuration text files are maintained in this repository. Executable code and test harnesses belong in `ModelOps-Tests`.

---

## 🗺️ Repository Structure & Template Sitemap

```
ModelOps-Templates/
├── components/
│   ├── evaluate-metric-based-google/
│   │   ├── evaluate-metric-based-google.yml  # Job template for computation-based metrics (exact_match, rouge_l, bleu)
│   │   ├── build_python_environment.yml      # Job template to build & publish the Python virtualenv artifact (.venv)
│   │   └── setup-wif-template.yml            # Step template for Workload Identity Federation (WIF) GCP authentication
│   └── evaluate-model-based-google/
│       └── evaluate-model-based-google.yml   # Job template for model-based/LLM-as-a-judge metrics (safety, groundedness, fluency)
└── pipelines/
    └── conversational-agent/
        └── conversational-agent.yml          # Top-level orchestration pipeline template
```

---

## 🧩 Component Templates Overview

### 1. `evaluate-metric-based-google`
Job template for executing deterministic, computation-based metric evaluations against deployed Reasoning Engine endpoints.
* **Key Parameters**: `projectId`, `gcpRegion`, `agentId`, `datasetPath`, `traceBucketName`, `metrics` (default: `exact_match,rouge_l`), `scriptPath`.

### 2. `evaluate-model-based-google`
Job template for executing model-based (LLM-as-a-judge) quality and safety evaluations.
* **Key Parameters**: `projectId`, `gcpRegion`, `agentId`, `datasetPath`, `traceBucketName`, `metrics` (default: `groundedness,safety,fluency,coherence`), `safetyThreshold` (default: `0.95`).
* **Manual Gate**: Includes a `HoldOnError` job (`ManualValidation@0`) that pauses the ADO pipeline if metric thresholds drop or raise issues.

### 3. `build_python_environment.yml`
Job template that initializes a Python 3.11 virtualenv (`.venv`), installs dependencies from `requirements.txt`, and publishes the virtualenv as an ADO Pipeline Artifact named `python-venv`.

### 4. `setup-wif-template.yml`
Step template for setting up keyless authentication between Azure DevOps and Google Cloud Platform via Workload Identity Federation (WIF).

---

## 💡 How an AI Agent / Developer Consumes This Repo

To reference a template from `ModelOps-Templates` in a consumer repository (such as `ModelOps-Tests`):

```yaml
jobs:
  - template: path/to/ModelOps-Templates/components/evaluate-metric-based-google/evaluate-metric-based-google.yml
    parameters:
      projectId: $(projectId)
      gcpRegion: $(gcpRegion)
      agentId: $(resolvedAgentId)
      datasetPath: '$(Build.SourcesDirectory)/data/evalset.jsonl'
      traceBucketName: $(traceBucketName)
      scriptPath: '$(Build.SourcesDirectory)/scripts/evaluate_metric_based.py'
```
