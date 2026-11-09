---
name: auroc-statistical-comparison-across-methods
description: Use when several locus-gene link inference methods are scored on identical
  candidate peak-gene pairs and need a rigorous retrieval comparison against eQTL-supported
  ground truth with per-sample AUROC and statistical significance testing.
when_to_use_negative:
- The multimodal dataset comes from tissue not undergoing a dynamic process (e.g.
  differentiation or disease), since pseudotemporal context for DAG construction is
  a stated reliance of the approach
- Methods are evaluated on different candidate peak-gene pair sets — AUROC comparison
  requires identical candidate pairs and DAG across all methods
- No eQTL ground truth is available for the tissue/organism, so retrieval accuracy
  cannot be validated (the eQTL cross-referencing only mimics perturbation-based validation)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GrID-Net
  role: Method under benchmark; produces Granger causal peak-gene scores (L=10 GNN
    layers, Adam, lr 0.001, batch 1024) converted to pair-level AUROC
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Learns joint RNA+ATAC cell representations used to build the kNN cell DAG
    shared by all benchmarked methods
  repo: https://github.com/rs239/schema
- name: PyTorch
  role: Implementation framework in which the GrID-Net models compared in the benchmark
    are trained
- name: scanpy
  role: Provides the diffusion pseudotime implementation used to orient DAG edges
    before scoring
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:972ad11e295fcc9c8e956ffaf42697b21d614aeddb2ee3e34e9b1078eccb3902
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/alexw16/gridnet
  repo_url: https://github.com/alexw16/gridnet
  tools:
  - GrID-Net
  - PyTorch
  - Schema
  - scanpy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# auroc-statistical-comparison-across-methods

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking skill for comparing peak-gene link inference methods (GrID-Net vs Pearson/pseudocell correlation, Linear Granger, GVAR, ABC) by per-sample AUROC against eQTL Catalogue ground-truth pairs, aggregated across 5 samples per multimodal dataset and tested with a Welch one-tailed t-test.

## When to use

Use when several locus-gene link inference methods are scored on identical candidate peak-gene pairs and need a rigorous retrieval comparison against eQTL-supported ground truth with per-sample AUROC and statistical significance testing.

## When NOT to use

- The multimodal dataset comes from tissue not undergoing a dynamic process (e.g. differentiation or disease), since pseudotemporal context for DAG construction is a stated reliance of the approach
- Methods are evaluated on different candidate peak-gene pair sets — AUROC comparison requires identical candidate pairs and DAG across all methods
- No eQTL ground truth is available for the tissue/organism, so retrieval accuracy cannot be validated (the eQTL cross-referencing only mimics perturbation-based validation)

## Inputs

- Unprocessed RNA-seq and ATAC-seq AnnData objects per sample (5 samples per dataset; 10x Multiome, SHARE-seq, SNARE-seq, sci-CAR) with TSS and peak genomic coordinate annotations in adata.var
- Candidate peak-gene pair list (gridnet.identify_all_peak_gene_link_candidates DataFrame with atac_id/gene columns)
- DAG adjacency matrix of cells from gridnet.construct_dag (kNN k=15, pseudotime_algo='dpt')
- Significant eQTLs (P<10^-10) from the eQTL Catalogue defining ground-truth peak-gene pairs

## Outputs

- Per-method, per-sample AUROC values against eQTL-supported peak-gene pairs
- Per-method AUROC mean ± SD aggregated across the 5 samples per dataset
- Welch one-tailed t-test results (GrID-Net vs next best method per dataset)
- Consolidated benchmark table written to eqtl_benchmark_auroc.csv

## How to apply

Preprocess each of the four multimodal datasets (10x Multiome, SHARE-seq, SNARE-seq, sci-CAR; 5 samples each) with gridnet.preprocess_multimodal, filtering genes expressed in <1% of cells (rna_filter_gene_percent) and peaks accessible in <0.1% of cells (atac_filter_peak_percent). Assemble identical candidate peak-gene pairs per sample with gridnet.identify_all_peak_gene_link_candidates using distance_thresh=1e6 (250 kb for 10x Multiome) so all methods are scored on the same pair set and DAG. Score pairs with GrID-Net (L=10 GNN layers, Adam optimizer, learning rate 0.001, mini-batch size 1024 candidate peak-gene pairs) and with the baselines (Pearson correlation, pseudocell correlation, Linear Granger, GVAR, ABC). For each method and sample, compute AUROC against ground-truth peak-gene pairs overlapping significant eQTLs (P<10^-10) from the eQTL Catalogue, aggregate mean ± SD across the 5 samples per dataset, and test GrID-Net vs the next best method with a Welch one-tailed t-test (significance threshold P<0.01). Correct application yields a 7-22% AUROC gain for GrID-Net in 3 of 4 datasets, with GVAR on sci-CAR as the only non-significant exception.

## Related tools

- **GrID-Net** (Method under benchmark; produces Granger causal peak-gene scores (L=10 GNN layers, Adam, lr 0.001, batch 1024) converted to pair-level AUROC) — https://github.com/alexw16/gridnet
- **Schema** (Learns joint RNA+ATAC cell representations used to build the kNN cell DAG shared by all benchmarked methods) — https://github.com/rs239/schema
- **PyTorch** (Implementation framework in which the GrID-Net models compared in the benchmark are trained)
- **scanpy** (Provides the diffusion pseudotime implementation used to orient DAG edges before scoring)

## Examples

```
results_df = gridnet.gridnet_multimodal(rna_adata,atac_adata,root_cell_marker_gene='TOP2A')
```

## Evaluation signals

- GrID-Net AUROC exceeds the next best method by 7-22% in 3 of the 4 datasets
- Welch one-tailed t-test gives P < 0.01 for GrID-Net vs next best method across all datasets, with GVAR on sci-CAR as the only exception
- All methods were scored on the identical candidate peak-gene pairs and DAG per sample (distance_thresh=1e6, or 250 kb for 10x Multiome)
- Per-dataset aggregation reports mean ± SD across 5 samples per method, not single-run values
- Consolidated table eqtl_benchmark_auroc.csv contains per-method AUROC mean ± SD and t-test results for all four datasets

## Limitations

- eQTL-based evaluation only mimics perturbation validation; inferred locus-gene associations are causal hypotheses requiring perturbation follow-up
- Requires multiomic single-cell data from matching tissue undergoing a dynamic biological process to provide pseudotemporal context
- Computational burden of large single-cell datasets may exceed GPU memory capacity (models trained on a single NVIDIA Tesla V100)
- The statistical comparison is not universally significant: GVAR on sci-CAR is the one case where GrID-Net was not significantly more accurate

## Evidence

- [intro] We demonstrate that it outperforms existing state-of-the-art approaches for accurately identifying noncoding locus-gene links across a variety of cell types and single-cell multimodal assays.: "We demonstrate that it outperforms existing state-of-the-art approaches for accurately identifying noncoding locus-gene links across a variety of cell types and single-cell multimodal assays."
- [discussion] All GrID-Net models consisted of L=10 GNN layers and were trained using the Adam optimizer with a learning rate of 0.001 and a mini-batch size of 1024 candidate peak-gene pairs.: [evidence span withheld — anchor status: not_found]
- [discussion] we trained a logistic regression model to predict peak-gene pairs overlapping significant skin eQTLs (P < 10−3) for the SHARE-seq mouse skin dataset: "we trained a logistic regression model to predict peak-gene pairs overlapping significant skin eQTLs (P < 10−3) for the SHARE-seq mouse skin dataset"
- [discussion] the locus-gene associations it infers are causal hypotheses that need to be validated by perturbations. In the eQTL-based evaluation, we sought to mimic such validations: "the locus-gene associations it infers are causal hypotheses that need to be validated by perturbations. In the eQTL-based evaluation, we sought to mimic such validations"
- [readme] The output is a table of statistics that describes the significance of each of the tested candidate Granger causal relationships.: "The output is a table of statistics that describes the significance of each of the tested candidate Granger causal relationships."
