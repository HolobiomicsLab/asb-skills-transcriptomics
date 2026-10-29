---
name: single-cell-grn-inference-and-ensemble-merging
description: Use when the goal is to build a whole-transcriptome GRN from a scRNA-seq,
  snRNA-seq, or spatial dataset and the analysis requires robust consensus edges,
  module detection, and downstream perturbation-prediction evaluation (AUROC, TPR
  at FPR 0.05).
when_to_use_negative:
- The task requires explicitly causal or directional regulatory edges — the inferred
  links 'are not necessarily causal or directional' per the article's discussion
- The task requires leveraging spatial coordinates during network construction — SCING
  'does not currently utilize the spatial information' despite GRNs showing spatial
  patterns
- The task requires integrating additional omics (scATAC-seq, scHi-C, ChIP-seq, cis/trans
  eQTL) — these modalities are deferred to future work in SCING
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: SCING
  role: 'Primary GRN inference pipeline: supercell construction, 100 subsampled GBR-based
    GRNs, 20% consensus merging, conditional mutual information edge pruning'
  repo: https://github.com/XiaYangLabOrg/SCING
- name: GRNBOOST2
  role: Baseline gradient boosting method benchmarked against SCING on Perturb-seq
    perturbation prediction
- name: PIDC
  role: Baseline mutual-information-based method benchmarked against SCING
- name: ppcor
  role: Baseline partial-correlation method benchmarked against SCING
- name: pyscenic
  role: Provides AUCell module scoring used for module-specific expression per cell
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:20599611dd8df27a750e4b18fccef43005f7b7ce1abb81dda0389d8b81d39346
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - GRNBOOST2
  - PIDC
  - SCING
  - ppcor
  - pyscenic
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# single-cell-grn-inference-and-ensemble-merging

## Summary

Infers robust gene regulatory networks from full single-cell transcriptomes (scRNA-seq, snRNA-seq, spatial) using SCING's bagging-based consensus: supercells, ~100 gradient-boosted GRNs on subsamples, and edge pruning via conditional mutual information, benchmarkable against GRNBOOST2, PIDC, and ppcor on Perturb-seq perturbation-prediction.

## When to use

Use when the goal is to build a whole-transcriptome GRN from a scRNA-seq, snRNA-seq, or spatial dataset and the analysis requires robust consensus edges, module detection, and downstream perturbation-prediction evaluation (AUROC, TPR at FPR 0.05).

## When NOT to use

- The task requires explicitly causal or directional regulatory edges — the inferred links 'are not necessarily causal or directional' per the article's discussion
- The task requires leveraging spatial coordinates during network construction — SCING 'does not currently utilize the spatial information' despite GRNs showing spatial patterns
- The task requires integrating additional omics (scATAC-seq, scHi-C, ChIP-seq, cis/trans eQTL) — these modalities are deferred to future work in SCING

## Inputs

- scRNA-seq, snRNA-seq, or spatial transcriptomics count matrix (e.g. Perturb-seq matrices from GEO: GSE90063, GSE153056)
- Preprocessed expression data with genes and cells (full transcriptome, not pre-selected gene subsets)

## Outputs

- Merged, pruned GRN (consensus edge list from 100 subsampled networks)
- Leiden subnetworks/modules with AUCell module scores per cell
- Pathway-enrichment annotations of gene modules (enrichr)
- perturbseq_benchmark_results.csv: AUROC and TPR-at-FPR-0.05 for SCING (undirected and directed), GRNBOOST2, ppcor, and PIDC on each dataset

## How to apply

Build supercells by clustering cells with the leiden graph partitioning algorithm and merging subclusters, then construct 100 GRNs in parallel on subsamples of the supercells using gradient boosting regressors (GBR), defining the consensus as edges appearing in 20% of the networks. Merge the 100 networks into a final GRN and prune edges and cycles using conditional mutual information (petrics), evaluating edges as undirected (with an additional directed evaluation for SCING). Partition the merged GRN into subnetworks with leiden, score module-specific expression per cell with AUCell, and annotate modules via enrichr pathway enrichment. For benchmarking, rebuild GRNs on the same Perturb-seq datasets (Dixit et al. GSE90063, Papalexi et al. GSE153056) with GRNBOOST2, ppcor, and PIDC, fit an elastic net regression to predict perturbation effects on downstream genes, and compute AUROC and TPR at FPR 0.05. For overfitting checks, split data into train/test sets, train a GBR per gene from its parents, and compare predicted vs. true test expression via cosine similarity.

## Related tools

- **SCING** (Primary GRN inference pipeline: supercell construction, 100 subsampled GBR-based GRNs, 20% consensus merging, conditional mutual information edge pruning) — https://github.com/XiaYangLabOrg/SCING
- **GRNBOOST2** (Baseline gradient boosting method benchmarked against SCING on Perturb-seq perturbation prediction)
- **PIDC** (Baseline mutual-information-based method benchmarked against SCING)
- **ppcor** (Baseline partial-correlation method benchmarked against SCING)
- **pyscenic** (Provides AUCell module scoring used for module-specific expression per cell)

## Examples

```
git clone https://github.com/XiaYangLabOrg/SCING.git && cd SCING && conda env create -n scing --file install/scing.environment.yml && conda activate scing && pip install pyitlib && pip install -e .
```

## Evaluation signals

- Consensus edges must appear in at least 20% of the 100 subsampled GRNs before merging
- Merged GRN must be pruned for edges and cycles using conditional mutual information before downstream use
- Benchmark table must contain one AUROC and one TPR-at-FPR-0.05 entry for each of the four methods (SCING, GRNBOOST2, PIDC, ppcor) on each of the two Perturb-seq datasets, with SCING directed evaluation reported separately
- Held-out validation: per-gene GBR predictions on the test set should show high cosine similarity to true expression, indicating the network is not overfit
- Leiden partitioning must yield subnetworks whose AUCell module scores support clustering and phenotypic association consistent with known cell types

## Limitations

- Inferred GRN links are not necessarily causal or directional
- Many hyperparameters require selection, and while robustness is shown across a range, selection can be dataset dependent
- Spatial information is not explicitly utilized in SCING GRN construction, even for spatial transcriptomics input
- Additional omics modalities (scATAC-seq, scHi-C, ChIP-seq, cis/trans eQTL) are not yet incorporated and could otherwise improve accuracy

## Evidence

- [abstract] SCING is a gradient boosting + MI ensemble GRN method: "we present Single Cell INtegrative Gene regulatory network inference (SCING), a gradient boosting and mutual information-based approach for identifying robust GRNs from scRNA-seq, snRNA-seq, and"
- [readme] Consensus rule and edge pruning: "We find the consensus as edges that show up in 20% of the networks. We then prune edges and cycles using conditional mutual information"
- [readme] Pipeline notebook workflow: "BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges"
- [methods] Module detection and scoring: "We apply the leiden graph partitioning algorithm to identify GRN subnetworks"
- [methods] AUCell module expression scoring: "We combine these subnetworks with the AUCell method to get module specific expression for each cell"
- [discussion] Directionality caveat: "the links between genes are not necessarily causal or directional"
- [intro] Benchmarking against baselines: "We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics, and disease modeling accuracy."
