---
name: single-cell-clustering-accuracy-evaluation
description: Use when you need to judge whether a single-cell clustering/joint-embedding
  method (e.g. CoupledNMF, Cell Ranger, Seurat V3) produces subpopulations consistent
  with external labels derived from FACS-sorted bulk signatures.
when_to_use_negative:
- Data come from multiple donors per condition — sc-compReg's approach does not yet
  accommodate multiple donors under each condition to be compared
- You lack external (bulk/FACS-derived) reference labels — silhouette significance
  against surrogate ground truth cannot be computed
- The goal is downstream differential regulatory network construction (use the sc_compreg
  workflow instead of the clustering-validation skill)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3474
tools:
- name: sc-compReg / scCompReg
  role: Provides the CoupledNMF joint clustering, cnmf_tsne embedding visualization,
    and cnmf_process_data.sh / cnmf_load_coupling_matrix utilities whose outputs are
    being evaluated
  repo: https://github.com/SUwonglab/sc-compReg
- name: R (>= 3.6.0)
  role: Runtime environment for running scCompReg, computing silhouette indices, and
    performing sign rank tests
- name: Bedtools
  role: Linux prerequisite for the cnmf peak preprocessing step (cnmf_process_data.sh)
- name: Homer
  role: Linux prerequisite listed in the scCompReg system requirements for motif/peak
    processing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:7b5db106956816628c70a5c175b12d2888a88c857c20046652860dbe8011384a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/sc-compReg
  tools:
  - Bedtools
  - Homer
  - R (>= 3.6.0)
  - R (>=)
  - sc-compReg
  - sc-compReg / scCompReg
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Single-cell clustering accuracy evaluation

## Summary

Validates single-cell clustering and joint embedding (CoupledNMF vs Cell Ranger vs Seurat V3 label transfer) on healthy donor BMMC scRNA-seq/scATAC-seq by scoring cluster assignments against surrogate ground truth from FACS-sorted bulk cell-type signatures, using silhouette index and sign rank tests.

## When to use

Use when you need to judge whether a single-cell clustering/joint-embedding method (e.g. CoupledNMF, Cell Ranger, Seurat V3) produces subpopulations consistent with external labels derived from FACS-sorted bulk signatures.

## When NOT to use

- Data come from multiple donors per condition — sc-compReg's approach does not yet accommodate multiple donors under each condition to be compared
- You lack external (bulk/FACS-derived) reference labels — silhouette significance against surrogate ground truth cannot be computed
- The goal is downstream differential regulatory network construction (use the sc_compreg workflow instead of the clustering-validation skill)

## Inputs

- Healthy donor BMMC scRNA-seq count matrix
- Healthy donor BMMC scATAC-seq peak/count matrix (peak.bed for preprocessing)
- FACS-sorted bulk-derived surrogate ground-truth labels for constituent subpopulations
- peak_gene_coupling_matrix.txt (coupling matrix for CoupledNMF)
- Genome version identifier from {hg19, hg38, mm9, mm10}
- prior_data directory (e.g. motif_human.rds) from the sc-compReg repository

## Outputs

- Per-cluster assignments for each strategy (Cell Ranger, Seurat V3, CoupledNMF)
- Clustering accuracy scores per strategy against FACS-bulk surrogate labels
- Silhouette index values and sign rank test p-values for embedding comparison
- joint_clusters_plot.png and joint_data_type_plot.png from cnmf_tsne

## How to apply

Load healthy donor BMMC scRNA-seq and scATAC-seq count matrices plus the FACS-bulk-derived surrogate ground-truth labels for constituent subpopulations. Run each candidate strategy: Cell Ranger clustering on each sample independently, Seurat V3 label transfer between the two modalities, and CoupledNMF joint clustering after preprocessing peaks with cnmf_process_data.sh and loading the coupling matrix via cnmf_load_coupling_matrix. Evaluate each strategy's per-cluster assignments against the FACS-bulk surrogate labels, then compare joint t-SNE embeddings via the silhouette index using a sign rank test; correct application reproduces CoupledNMF silhouettes significantly higher than separate Cell Ranger t-SNEs (sign rank test p = 5.7973e-171 for scATAC-seq and p = 5.7588e-08 for scRNA-seq), with CoupledNMF the best-performing method overall. Visualize the joint clustering with cnmf_tsne on cnmf.output$H1 and cnmf.output$H2 (perplexity=100) to confirm cells sharing a ground-truth label colocate in the joint embedding.

## Related tools

- **sc-compReg / scCompReg** (Provides the CoupledNMF joint clustering, cnmf_tsne embedding visualization, and cnmf_process_data.sh / cnmf_load_coupling_matrix utilities whose outputs are being evaluated) — https://github.com/SUwonglab/sc-compReg
- **R (>= 3.6.0)** (Runtime environment for running scCompReg, computing silhouette indices, and performing sign rank tests)
- **Bedtools** (Linux prerequisite for the cnmf peak preprocessing step (cnmf_process_data.sh))
- **Homer** (Linux prerequisite listed in the scCompReg system requirements for motif/peak processing)

## Examples

```
cnmf_tsne(cnmf.output$H1, cnmf.output$H2, path=path, save.plot=F, perplexity=100)
```

## Evaluation signals

- Sign rank test on silhouette indices reproduces p = 5.7973e-171 for scATAC-seq and p = 5.7588e-08 for scRNA-seq when comparing CoupledNMF joint t-SNE vs separate Cell Ranger t-SNEs
- CoupledNMF is the best-performing clustering method overall when accuracies are scored against FACS-bulk surrogate labels
- Cells sharing a ground-truth label are placed near each other in the joint t-SNE embedding (visualized in joint_clusters_plot.png)
- cnmf_tsne with perplexity=100 on cnmf.output$H1/H2 generates both joint_clusters_plot.png and joint_data_type_plot.png without error (save.plot=F for interactive check)
- Preprocessing with genome_version restricted to {hg19, hg38, mm9, mm10} completes and produces peak_gene_coupling_matrix.txt loadable by cnmf_load_coupling_matrix

## Limitations

- Ground truth is surrogate, derived from FACS-sorted bulk cell-type signatures rather than single-cell-resolved labels
- The sc-compReg approach does not yet accommodate multiple donors per condition; extension pending data availability
- Differential regulations discovered this way need replication by additional patients and controls

## Evidence

- [abstract] Joint clustering and embedding of cells from both scRNA-seq and scATAC-seq: "provides joint clustering and embedding of the cells from both scRNA-seq and scATAC-seq"
- [methods] cnmf_tsne visualization utility: "`cnmf_tsne` allows users to visualize clustering outputs."
- [methods] CoupledNMF cluster-label step: "Run `cnmf` to get the cluster labels for sample 1 and sample 2."
- [readme] Package capability statement: "an R package that provides coupled clustering and joint embedding of scRNA-seq and scATAC-seq on one sample"
- [intro] Validation strategy with bulk-derived labels: "To validate a single-cell analysis method, we apply it to scRNA-seq and/or scATAC-seq samples from a heterogeneous population with constituent subpopulations that are already analyzed by bulk sample"
- [other] Reproduced silhouette p-values from source task card: [evidence span withheld — anchor status: not_found]
