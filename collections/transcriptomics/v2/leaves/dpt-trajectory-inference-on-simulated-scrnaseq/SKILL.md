---
name: dpt-trajectory-inference-on-simulated-scrnaseq
description: Use when comparing dimensionality-reduction methods on simulated scRNA-seq
  lineage data with a known ground-truth cell ordering, and you need pseudotime inferred
  on each embedding and quantified by Kendall-tau correlation.
when_to_use_negative:
- Data are empirical/real cells without a ground-truth trajectory ordering, so Kendall-tau
  validation against simulated truth is impossible
- Input is raw unembedded scRNA-seq data — DPT must follow dimensionality reduction
  per the scGAE pipeline (scTransform preprocessing, KNN graph, autoencoder embedding)
- The goal is discrete cluster separation rather than developmental trajectory recovery
  (scGAE and scGNN were compared on clustering separately)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGAE
  role: Produces the low-dimensional, topology-preserving embeddings on which DPT
    is run; preprocessing with scTransform, KNN cell graph with default K=35
  repo: github.com/ZixiangLuo1161/scGAE
- name: DCA
  role: Competitor embedding method applied with default settings before DPT scoring
- name: scVI
  role: Competitor embedding method applied with default settings before DPT scoring
- name: PHATE
  role: Competitor embedding method applied with default settings before DPT scoring
- name: SAUCIE
  role: Competitor embedding method applied with default settings before DPT scoring
- name: scGNN
  role: Competitor embedding method applied with default settings before DPT scoring;
    close trajectory-inference rival of scGAE
- name: ZINB-WaVE
  role: Competitor embedding method applied with default settings before DPT scoring
- name: Ivis
  role: Competitor embedding method applied with default settings before DPT scoring
- name: scanpy
  role: Python environment (v1.8.0 per README requirements) used to run DPT on each
    embedding
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:a8b23415f4633e8e4d4d1e7b499c7a41043cb78276d73778282fd10d1dc24789
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
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# dpt-trajectory-inference-on-simulated-scrnaseq

## Summary

Benchmarks diffusion pseudotime (DPT) trajectory inference on low-dimensional embeddings of simulated scRNA-seq lineage data (scenario1/data5 and scenario2/data2), scoring each method's pseudotime ordering against the ground truth by Kendall-tau correlation, as in the scGAE paper's Fig. 3a,b comparison.

## When to use

Use when comparing dimensionality-reduction methods on simulated scRNA-seq lineage data with a known ground-truth cell ordering, and you need pseudotime inferred on each embedding and quantified by Kendall-tau correlation.

## When NOT to use

- Data are empirical/real cells without a ground-truth trajectory ordering, so Kendall-tau validation against simulated truth is impossible
- Input is raw unembedded scRNA-seq data — DPT must follow dimensionality reduction per the scGAE pipeline (scTransform preprocessing, KNN graph, autoencoder embedding)
- The goal is discrete cluster separation rather than developmental trajectory recovery (scGAE and scGNN were compared on clustering separately)

## Inputs

- Simulated scRNA-seq lineage count matrices (data5 = scenario1, data2 = scenario2) shipped with the scGAE repository notebook Run_scGAE.ipynb
- Low-dimensional cell embeddings (one per method: scGAE, DCA, scVI, PHATE, SAUCIE, scGNN, ZINB-WaVE, Ivis)
- Ground-truth simulated developmental trajectory cell ordering

## Outputs

- DPT pseudotime ordering of cells for every method embedding and both scenarios
- kendall_tau_comparison.csv: one row per method per scenario with Kendall-tau values in [-1, 1]

## How to apply

Load the simulated lineage datasets data5 (scenario1) and data2 (scenario2) from the scGAE repository notebook Run_scGAE.ipynb. Preprocess each count matrix with scTransform (Seurat), build a KNN cell graph with the default K=35, and obtain latent embeddings with scGAE via get_adj, model.train, and model.embedding(); embed the same datasets with DCA, scVI, PHATE, SAUCIE, scGNN, ZINB-WaVE, and Ivis using default settings. Run diffusion pseudotime (DPT) trajectory inference on every low-dimensional embedding for both scenarios, excluding scDeepCluster as in the paper's Fig. 3a,b. Compute the Kendall-tau correlation between each inferred pseudotime ordering and the ground-truth simulated trajectory ordering, then assemble one row per method per scenario into kendall_tau_comparison.csv.

## Related tools

- **scGAE** (Produces the low-dimensional, topology-preserving embeddings on which DPT is run; preprocessing with scTransform, KNN cell graph with default K=35) — github.com/ZixiangLuo1161/scGAE
- **DCA** (Competitor embedding method applied with default settings before DPT scoring)
- **scVI** (Competitor embedding method applied with default settings before DPT scoring)
- **PHATE** (Competitor embedding method applied with default settings before DPT scoring)
- **SAUCIE** (Competitor embedding method applied with default settings before DPT scoring)
- **scGNN** (Competitor embedding method applied with default settings before DPT scoring; close trajectory-inference rival of scGAE)
- **ZINB-WaVE** (Competitor embedding method applied with default settings before DPT scoring)
- **Ivis** (Competitor embedding method applied with default settings before DPT scoring)
- **scanpy** (Python environment (v1.8.0 per README requirements) used to run DPT on each embedding)

## Examples

```
import scanpy as sc; sc.tl.dpt(adata); from scipy.stats import kendalltau; tau = kendalltau(adata.obs['dpt_pseudotime'], ground_truth_ordering)[0]
```

## Evaluation signals

- Every reported Kendall-tau value lies within [-1, 1]
- The relative ordering of methods matches Fig. 3a,b: scGAE, scGNN, and scVI best recover the original developmental trajectory on both scenario1 and scenario2
- The comparison table contains one row per method per scenario (scGAE plus DCA, scVI, PHATE, SAUCIE, scGNN, ZINB-WaVE, Ivis), with scDeepCluster excluded
- DPT is run on every low-dimensional embedding for both scenario1 (data5) and scenario2 (data2), not a subset

## Limitations

- The optimal K for KNN cell-graph construction is unresolved: 'the optimal K varies among different datasets and different parts of a dataset'
- No random seed or stochastic-variation control is reported for the autoencoder training, so tau values may vary across reruns
- The article specifies no software/package versions for the competitor tools, complicating exact reproduction of Fig. 3a,b

## Evidence

- [abstract] Simulated-data benchmark scope: "Analyses of simulated data showed that scGAE accurately reconstructs developmental trajectory and separates discrete cell clusters under different scenarios"
- [intro] Competitor method set: "Recently developed competitive methods include DCA7, scVI8, scDeepCluster9, PHATE10, SAUCIE11, scGNN12, ZINB-WaVE13 and Ivis14"
- [discussion] KNN graph default parameter: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
- [discussion] Preprocessing step: "The scRNA-seq data preprocessing was conducted using scTransform40 in The Seurat package41."
- [discussion] Trajectory-inference rivalry: "scGAE and scGNN have similar performance on the trajectory inference while scGAE has better performance on clustering."
- [readme] Notebook with running example and parameters: [evidence span withheld — anchor status: external_doc]
