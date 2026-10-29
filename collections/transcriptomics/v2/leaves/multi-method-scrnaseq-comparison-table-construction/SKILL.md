---
name: multi-method-scrnaseq-comparison-table-construction
description: Use when comparing dimensionality-reduction or embedding methods on scRNA-seq
  data by a single downstream metric (e.g., pseudotime recovery), and a per-method-per-scenario
  results table with bounded scores is required.
when_to_use_negative:
- When the goal is clustering performance comparison rather than trajectory recovery
  — the paper notes scGAE and scGNN have similar trajectory performance but differ
  on clustering, so Kendall tau alone would not separate them there.
- When the input data lack a ground-truth trajectory ordering (e.g., purely empirical
  datasets like HSPCs HRA000084 or pancreatic cells GSE84133), since Kendall-tau correlation
  against simulated ground truth cannot be computed.
- When comparing against scDeepCluster — it is explicitly excluded from the paper's
  Fig. 3a,b trajectory comparison.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGAE
  role: 'Primary dimensionality-reduction method: scTransform preprocessing, KNN cell
    graph (K=35), multitask-oriented graph autoencoder embedding'
  repo: github.com/ZixiangLuo1161/scGAE
- name: scGNN
  role: Competitor embedding method benchmarked for trajectory recovery
- name: scVI
  role: Competitor embedding method benchmarked for trajectory recovery
- name: DCA
  role: Competitor embedding method benchmarked for trajectory recovery
- name: PHATE
  role: Competitor embedding method benchmarked for trajectory recovery
- name: SAUCIE
  role: Competitor embedding method benchmarked for trajectory recovery
- name: ZINB-WaVE
  role: Competitor embedding method benchmarked for trajectory recovery
- name: Ivis
  role: Competitor embedding method benchmarked for trajectory recovery
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:8da6808ea2b0e2f9c3fbe08f8d7f0d903eb3918de12ce19a7daa76d47151c062
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DCA
  - Ivis
  - PHATE
  - SAUCIE
  - ZINB-WaVE
  - scGAE
  - scGNN
  - scVI
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# multi-method-scrnaseq-comparison-table-construction

## Summary

Builds a benchmark comparison table of Kendall-tau correlation values for trajectory inference across multiple scRNA-seq dimensionality-reduction methods (scGAE, DCA, scVI, PHATE, SAUCIE, scGNN, ZINB-WaVE, Ivis) on simulated lineage datasets, mirroring the scGAE paper's Fig. 3a,b evaluation.

## When to use

Use when comparing dimensionality-reduction or embedding methods on scRNA-seq data by a single downstream metric (e.g., pseudotime recovery), and a per-method-per-scenario results table with bounded scores is required.

## When NOT to use

- When the goal is clustering performance comparison rather than trajectory recovery — the paper notes scGAE and scGNN have similar trajectory performance but differ on clustering, so Kendall tau alone would not separate them there.
- When the input data lack a ground-truth trajectory ordering (e.g., purely empirical datasets like HSPCs HRA000084 or pancreatic cells GSE84133), since Kendall-tau correlation against simulated ground truth cannot be computed.
- When comparing against scDeepCluster — it is explicitly excluded from the paper's Fig. 3a,b trajectory comparison.

## Inputs

- Simulated scRNA-seq lineage datasets data5 (scenario1) and data2 (scenario2) from the scGAE repository (Run_scGAE.ipynb)
- Ground-truth simulated trajectory cell orderings for each scenario
- Trained low-dimensional embeddings from scGAE and competitor methods DCA, scVI, PHATE, SAUCIE, scGNN, ZINB-WaVE, Ivis

## Outputs

- kendall_tau_comparison.csv with one row per method per scenario and tau values in [-1, 1]
- Method ranking per scenario matching the relative ordering in the paper's Fig. 3a,b

## How to apply

Load the simulated scRNA-seq lineage datasets (data5 for scenario1 and data2 for scenario2) distributed with the scGAE repository notebook Run_scGAE.ipynb. Embed each dataset with scGAE — preprocessing with scTransform (Seurat), building a KNN cell graph with default K=35, and applying the multitask-oriented graph autoencoder — then embed the same data with each competitor (DCA, scVI, PHATE, SAUCIE, scGNN, ZINB-WaVE, Ivis) under default settings. Run diffusion pseudotime (DPT) trajectory inference on every low-dimensional embedding, compute Kendall-tau correlation of each inferred ordering against the ground-truth simulated trajectory ordering, and exclude scDeepCluster as in the paper's Fig. 3a,b comparison. Assemble one row per method per scenario into kendall_tau_comparison.csv.

## Related tools

- **scGAE** (Primary dimensionality-reduction method: scTransform preprocessing, KNN cell graph (K=35), multitask-oriented graph autoencoder embedding) — github.com/ZixiangLuo1161/scGAE
- **scGNN** (Competitor embedding method benchmarked for trajectory recovery)
- **scVI** (Competitor embedding method benchmarked for trajectory recovery)
- **DCA** (Competitor embedding method benchmarked for trajectory recovery)
- **PHATE** (Competitor embedding method benchmarked for trajectory recovery)
- **SAUCIE** (Competitor embedding method benchmarked for trajectory recovery)
- **ZINB-WaVE** (Competitor embedding method benchmarked for trajectory recovery)
- **Ivis** (Competitor embedding method benchmarked for trajectory recovery)

## Examples

```
jupyter notebook Run_scGAE.ipynb
```

## Evaluation signals

- Every reported Kendall-tau value lies within [-1, 1]
- Relative ordering of methods matches the paper's Fig. 3a,b: scGAE, scGNN, and scVI recover the trajectory better than other competitors on both scenario1 and scenario2
- The table has exactly one row per method per scenario (8 methods × 2 scenarios, excluding scDeepCluster)
- Each embedding was produced by applying the method with its default settings (e.g., scGAE with KNN K=35 and scTransform preprocessing)

## Limitations

- No software/package versions are specified in the article beyond the README's pinned versions (tensorflow 2.3.0, spektral 0.6.1, scanpy 1.8.0, etc.); results may vary across versions
- No random seed or stochastic-variation control is reported, so Kendall-tau values may vary between runs of stochastic embeddings
- The optimal K for KNN graph construction varies among datasets and different parts of a dataset, so default K=35 may not be optimal for new data

## Evidence

- [other] Run DPT on embeddings and compute Kendall tau: [evidence span withheld — anchor status: not_found]
- [discussion] Graph construction parameters: [evidence span withheld — anchor status: not_found]
- [discussion] Preprocessing step: "The scRNA-seq data preprocessing was conducted using scTransform40 in The Seurat package41."
- [results] Exclusion of scDeepCluster: [evidence span withheld — anchor status: not_found]
- [readme] README invocation guidance: [evidence span withheld — anchor status: external_doc]
- [readme] Requirement pinning: "tensorflow --- 2.3.0"
