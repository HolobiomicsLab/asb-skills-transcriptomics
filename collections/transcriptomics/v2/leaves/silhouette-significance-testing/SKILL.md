---
name: silhouette-significance-testing
description: Use when validating joint clustering/embedding of paired scRNA-seq and
  scATAC-seq against surrogate ground truth labels from bulk data, e.g. comparing
  CoupledNMF joint t-SNE to Cell Ranger or Seurat V3 label transfer via silhouette
  index.
when_to_use_negative:
- When the goal is downstream differential regulatory network comparison (sc_compreg)
  rather than validating the upstream clustering/embedding step
- When no surrogate ground truth labels (e.g. FACS-sorted bulk signatures) exist for
  the constituent subpopulations, since the silhouette index must be computed against
  known labels
- When data come from multiple donors per condition, which the sc-compReg approach
  does not yet accommodate
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: sc-compReg
  role: Provides CoupledNMF clustering and the cnmf_tsne function producing the joint
    t-SNE embeddings whose silhouette indices are tested
  repo: https://github.com/SUwonglab/sc-compReg
- name: R (>= 3.6.0)
  role: Runtime for the scCompReg package and for computing silhouette indices and
    sign rank tests
- name: Cell Ranger
  role: Baseline per-sample clustering strategy whose separate t-SNE silhouette scores
    are compared against CoupledNMF
- name: Seurat V3
  role: Baseline label-transfer strategy used as a third comparator for clustering
    accuracy against surrogate labels
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:56e3f7c9ef6668eef8d475f8f0aa5b2a45e801bbc3d5c87277f42762381537e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/sc-compReg
  tools:
  - Bedtools
  - Cell Ranger
  - Homer
  - R (>= 3.6.0)
  - R (>=)
  - Seurat V3
  - sc-compReg
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# silhouette-significance-testing

## Summary

Validation strategy from sc-compReg that scores clustering/embedding quality on healthy donor BMMC scRNA-seq and scATAC-seq by computing the silhouette index against FACS-bulk-derived surrogate ground truth labels and testing significance with a sign rank test comparing CoupledNMF joint t-SNE to Cell Ranger and Seurat V3 baselines.

## When to use

Use when validating joint clustering/embedding of paired scRNA-seq and scATAC-seq against surrogate ground truth labels from bulk data, e.g. comparing CoupledNMF joint t-SNE to Cell Ranger or Seurat V3 label transfer via silhouette index.

## When NOT to use

- When the goal is downstream differential regulatory network comparison (sc_compreg) rather than validating the upstream clustering/embedding step
- When no surrogate ground truth labels (e.g. FACS-sorted bulk signatures) exist for the constituent subpopulations, since the silhouette index must be computed against known labels
- When data come from multiple donors per condition, which the sc-compReg approach does not yet accommodate

## Inputs

- Healthy donor BMMC scRNA-seq count matrix
- Healthy donor BMMC scATAC-seq peak count matrix
- FACS-sorted bulk cell-type signature surrogate ground truth labels
- peak.bed file and prior_data directory (motif_human.rds, peak_gene_coupling_matrix.txt)

## Outputs

- Per-strategy silhouette index values against surrogate labels
- Sign rank test p-values (reported: 5.7973e-171 scATAC-seq, 5.7588e-08 scRNA-seq)
- joint_clusters_plot.png and joint_data_type_plot.png from cnmf_tsne
- CoupledNMF joint cluster labels for both samples (cnmf.output$H1, $H2)

## How to apply

Load healthy donor BMMC count matrices and FACS-bulk surrogate ground-truth labels, then run three clustering strategies: Cell Ranger on each single-cell sample, Seurat V3 label transfer between samples, and CoupledNMF on the joint data (after preprocessing peaks with cnmf_process_data.sh using one genome version of hg19/hg38/mm9/mm10 and loading the coupling matrix from peak_gene_coupling_matrix.txt). Project each clustering's cells via t-SNE (for CoupledNMF use cnmf_tsne on cnmf.output$H1 and $H2 with perplexity=100), compute the silhouette index of ground truth labels in each embedding, and compare strategies with a sign rank test. Correct application reproduces significantly higher silhouette for the CoupledNMF joint t-SNE, with reported p = 5.7973e-171 for scATAC-seq and p = 5.7588e-08 for scRNA-seq, confirming CoupledNMF as the best-performing clustering method overall.

## Related tools

- **sc-compReg** (Provides CoupledNMF clustering and the cnmf_tsne function producing the joint t-SNE embeddings whose silhouette indices are tested) — https://github.com/SUwonglab/sc-compReg
- **R (>= 3.6.0)** (Runtime for the scCompReg package and for computing silhouette indices and sign rank tests)
- **Cell Ranger** (Baseline per-sample clustering strategy whose separate t-SNE silhouette scores are compared against CoupledNMF)
- **Seurat V3** (Baseline label-transfer strategy used as a third comparator for clustering accuracy against surrogate labels)

## Examples

```
cnmf_tsne(cnmf.output$H1, cnmf.output$H2, path=path, save.plot=F, perplexity=100)
```

## Evaluation signals

- Sign rank test reproduces reported significance: p = 5.7973e-171 for scATAC-seq and p = 5.7588e-08 for scRNA-seq
- CoupledNMF joint t-SNE yields a significantly higher silhouette index than separate Cell Ranger t-SNEs
- cnmf_tsne with perplexity=100 produces joint_clusters_plot.png and joint_data_type_plot.png with same-label cells co-located
- Clustering accuracies match the reported per-strategy values against FACS-bulk surrogate labels
- Peak preprocessing runs with a valid genome_version from {hg19, hg38, mm9, mm10} and coupling matrix loads from peak_gene_coupling_matrix.txt

## Limitations

- Ground truth is surrogate: labels derive from FACS-sorted bulk cell-type signatures, not true single-cell identities
- Differential regulations discovered this way are not yet replicated in additional patients and controls
- The approach does not yet accommodate multiple donors per condition; extension awaits data availability

## Evidence

- [other] reported silhouette p-values: [evidence span withheld — anchor status: not_found]
- [abstract] joint embedding claim: "provides joint clustering and embedding of the cells from both scRNA-seq and scATAC-seq"
- [intro] validation objective: "To validate a single-cell analysis method, we apply it to scRNA-seq and/or scATAC-seq samples from a heterogeneous population with constituent subpopulations"
- [methods] cnmf run for cluster labels: "Run `cnmf` to get the cluster labels for sample 1 and sample 2."
- [readme] cnmf_tsne visualization: "an R package that provides coupled clustering and joint embedding of scRNA-seq and scATAC-seq on one sample"
