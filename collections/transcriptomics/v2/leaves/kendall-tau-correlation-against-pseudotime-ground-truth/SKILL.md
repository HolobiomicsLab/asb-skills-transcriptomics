---
name: kendall-tau-correlation-against-pseudotime-ground-truth
description: Use when you need a rank-based metric to score how well an embedding
  (e.g., from scGAE, scVI, scGNN) recovers a known cell developmental trajectory from
  simulated scRNA-seq lineage data.
when_to_use_negative:
- The data are empirical with no ground-truth trajectory ordering, so a rank correlation
  against known pseudotime cannot be computed
- You want to evaluate clustering rather than trajectory preservation — scGAE and
  scGNN show similar trajectory-inference performance, so Kendall tau alone does not
  discriminate clustering quality
- The method's output is only a 2D visualization embedding (scGAE embeds to an intermediate
  dimension plus tSNE for visualization), which may not be the representation intended
  for trajectory inference
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scGAE
  role: Produces the embedding whose trajectory preservation is scored; scTransform
    preprocessing, KNN cell graph (default K=35), multitask-oriented graph autoencoder
  repo: github.com/ZixiangLuo1161/scGAE
- name: DCA
  role: Competitor embedding method applied with default settings before DPT/Kendall-tau
    evaluation
- name: scVI
  role: Competitor embedding method; among the top trajectory-recovering methods alongside
    scGAE and scGNN
- name: PHATE
  role: Competitor embedding method applied with default settings
- name: SAUCIE
  role: Competitor embedding method applied with default settings
- name: scGNN
  role: Competitor embedding method; similar trajectory-inference performance to scGAE
- name: ZINB-WaVE
  role: Competitor embedding method applied with default settings
- name: Ivis
  role: Competitor embedding method applied with default settings
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:6289bb6f9df7d57946271d4ba165fb0620ad780024d5292da55c737da302f590
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# kendall-tau-correlation-against-pseudotime-ground-truth

## Summary

Evaluates a low-dimensional embedding's ability to preserve developmental ordering by running diffusion pseudotime (DPT) on the embedding and computing the Kendall-tau rank correlation between the inferred pseudotime ordering and the ground-truth simulated trajectory, as done in the scGAE paper's Fig. 3a,b comparison across scenarios 1 and 2.

## When to use

Use when you need a rank-based metric to score how well an embedding (e.g., from scGAE, scVI, scGNN) recovers a known cell developmental trajectory from simulated scRNA-seq lineage data.

## When NOT to use

- The data are empirical with no ground-truth trajectory ordering, so a rank correlation against known pseudotime cannot be computed
- You want to evaluate clustering rather than trajectory preservation — scGAE and scGNN show similar trajectory-inference performance, so Kendall tau alone does not discriminate clustering quality
- The method's output is only a 2D visualization embedding (scGAE embeds to an intermediate dimension plus tSNE for visualization), which may not be the representation intended for trajectory inference

## Inputs

- Low-dimensional cell embedding (per method) for a simulated scRNA-seq lineage dataset (e.g., scenario1 data5 or scenario2 data2)
- Ground-truth simulated trajectory ordering of cells (pseudotime labels from the simulation)
- Scenario/method metadata to label each row (scGAE, DCA, scVI, PHATE, SAUCIE, scGNN, ZINB-WaVE, Ivis)

## Outputs

- Kendall-tau correlation value in [-1, 1] per method per scenario
- Comparison table (e.g., kendall_tau_comparison.csv) with one row per method per scenario

## How to apply

For each embedding to be scored, run diffusion pseudotime (DPT) trajectory inference on the low-dimensional representation produced by the dimensionality-reduction method (e.g., scGAE's scTransform-preprocessed, KNN-graph (K=35) multitask graph autoencoder embedding, or default settings of DCA, scVI, PHATE, SAUCIE, scGNN, ZINB-WaVE, Ivis). Compute the Kendall-tau correlation between the inferred pseudotime ordering of cells and the ground-truth simulated trajectory ordering; tau ranges over [-1, 1]. Assemble one row per method per scenario (scenario1 = data5, scenario2 = data2 from the scGAE repository notebook Run_scGAE.ipynb) into a comparison table (kendall_tau_comparison.csv). Exclude methods that the reference comparison excluded (e.g., scDeepCluster as in Fig. 3a,b). Interpretation: methods whose embeddings better preserve topological/cell-cell relationships (scGAE, scGNN, scVI) yield higher tau, indicating better trajectory recovery.

## Related tools

- **scGAE** (Produces the embedding whose trajectory preservation is scored; scTransform preprocessing, KNN cell graph (default K=35), multitask-oriented graph autoencoder) — github.com/ZixiangLuo1161/scGAE
- **DCA** (Competitor embedding method applied with default settings before DPT/Kendall-tau evaluation)
- **scVI** (Competitor embedding method; among the top trajectory-recovering methods alongside scGAE and scGNN)
- **PHATE** (Competitor embedding method applied with default settings)
- **SAUCIE** (Competitor embedding method applied with default settings)
- **scGNN** (Competitor embedding method; similar trajectory-inference performance to scGAE)
- **ZINB-WaVE** (Competitor embedding method applied with default settings)
- **Ivis** (Competitor embedding method applied with default settings)

## Examples

```
from scipy.stats import kendalltau; tau, p = kendalltau(dpt_pseudotime, ground_truth_pseudotime)  # per embedding, per scenario; save rows to kendall_tau_comparison.csv
```

## Evaluation signals

- Every reported Kendall-tau value lies within [-1, 1]
- Relative ordering of methods by tau matches the comparison shown in the paper's Fig. 3a,b for scenario1 and scenario2
- scGAE, scGNN, and scVI achieve higher tau than the other competitive methods on both scenarios
- Exactly one row per method per scenario (2 scenarios x evaluated methods) in the output table
- DPT is run on the same low-dimensional embedding that each method produced, not on raw high-dimensional counts

## Limitations

- Kendall tau requires a ground-truth trajectory ordering, limiting the metric to simulated data with known pseudotime
- scGAE and scGNN have similar trajectory-inference performance, so tau alone does not separate them; clustering metrics are needed for that
- No random seed or stochastic-variation control is reported in the paper, so tau values may vary between runs
- The optimal K for KNN graph construction varies among datasets, affecting the embedding that is subsequently scored
- No software/package versions are specified for the tools, complicating exact reproduction

## Evidence

- [full_text] Kendall tau vs DPT ground truth workflow: [evidence span withheld — anchor status: not_found]
- [results] Task card finding on top-performing methods: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract claim on trajectory reconstruction: "Analyses of simulated data showed that scGAE accurately reconstructs developmental trajectory and separates discrete cell clusters under different scenarios"
- [discussion] Discussion on scGAE vs scGNN trajectory performance: "scGAE and scGNN have similar performance on the trajectory inference while scGAE has better performance on clustering."
- [discussion] Discussion on graph construction parameter: [evidence span withheld — anchor status: not_found]
