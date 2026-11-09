---
name: surrogate-groundtruth-label-construction
description: Use when you need to validate clustering or joint embedding of scRNA-seq/scATAC-seq
  from a heterogeneous population (e.g., healthy donor BMMC) for which FACS-sorted
  bulk cell-type signatures are already available.
when_to_use_negative:
- No bulk-derived or FACS-sorted reference for the population exists, so no surrogate
  labels can be constructed
- The population is homogeneous (no constituent subpopulations), so clustering validation
  against bulk signatures is not informative
- You only need differential regulatory network comparison downstream and clustering
  has already been validated
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: sc-compReg
  role: Provides CoupledNMF joint clustering, cnmf_process_data.sh peak preprocessing,
    cnmf_load_coupling_matrix, and cnmf_tsne embedding used to generate candidate
    clusterings evaluated against surrogate labels
  repo: https://github.com/SUwonglab/sc-compReg
- name: R (>= 3.6.0)
  role: Runtime environment in which scCompReg functions (cnmf_tsne, silhouette scoring)
    are executed
- name: Bedtools
  role: Linux dependency used for genomic interval operations during peak preprocessing
- name: Homer
  role: Linux dependency for motif/peak processing supporting the coupling-matrix
    construction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:4bdf44e81686f00e6fdb36146e702fa00603d46ef4b4f624fdcce47238e3dfbf
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# surrogate-groundtruth-label-construction

## Summary

Constructs surrogate ground-truth cell-type labels for validating single-cell clustering by leveraging a heterogeneous population whose constituent subpopulations were already profiled and FACS-sorted at the bulk level, then scoring cluster/embedding agreement against those labels (e.g., silhouette index with sign-rank test).

## When to use

Use when you need to validate clustering or joint embedding of scRNA-seq/scATAC-seq from a heterogeneous population (e.g., healthy donor BMMC) for which FACS-sorted bulk cell-type signatures are already available.

## When NOT to use

- No bulk-derived or FACS-sorted reference for the population exists, so no surrogate labels can be constructed
- The population is homogeneous (no constituent subpopulations), so clustering validation against bulk signatures is not informative
- You only need differential regulatory network comparison downstream and clustering has already been validated

## Inputs

- Healthy donor BMMC scRNA-seq count matrix
- Healthy donor BMMC scATAC-seq count matrix (peaks, peak.bed)
- FACS-bulk-derived surrogate ground-truth labels for constituent subpopulations
- peak_gene_coupling_matrix.txt (peak–gene coupling prior)
- prior_data directory from github.com/SUwonglab/sc-compReg

## Outputs

- Per-cell cluster labels from Cell Ranger, Seurat V3, and CoupledNMF
- Clustering accuracies against FACS-bulk surrogate labels
- Silhouette index values and sign-rank-test p-values per modality
- joint_clusters_plot.png and joint_data_type_plot.png t-SNE plots

## How to apply

Load the healthy donor BMMC scRNA-seq and scATAC-seq count matrices together with FACS-bulk-derived surrogate ground-truth labels for the constituent subpopulations. Run competing strategies on the same cells — Cell Ranger clustering per sample, Seurat V3 label transfer between modalities, and CoupledNMF joint clustering (after preprocessing peaks with cnmf_process_data.sh and loading the coupling matrix from peak_gene_coupling_matrix.txt). Compare each strategy's cluster assignments against the FACS-bulk surrogate labels, and build a joint t-SNE with cnmf_tsne to assess whether cells sharing a ground-truth label co-localize. Judge success by a higher silhouette index relative to Cell Ranger's separate t-SNEs, tested with a sign rank test, and by CoupledNMF ranking best among clustering methods. The article's reported benchmarks are silhouette significance p = 5.7973e-171 for scATAC-seq and p = 5.7588e-08 for scRNA-seq, which a correct reproduction should match.

## Related tools

- **sc-compReg** (Provides CoupledNMF joint clustering, cnmf_process_data.sh peak preprocessing, cnmf_load_coupling_matrix, and cnmf_tsne embedding used to generate candidate clusterings evaluated against surrogate labels) — https://github.com/SUwonglab/sc-compReg
- **R (>= 3.6.0)** (Runtime environment in which scCompReg functions (cnmf_tsne, silhouette scoring) are executed)
- **Bedtools** (Linux dependency used for genomic interval operations during peak preprocessing)
- **Homer** (Linux dependency for motif/peak processing supporting the coupling-matrix construction)

## Examples

```
bash cnmf_process_data.sh path/to/peak.bed hg19 path/to/prior_data
```

## Evaluation signals

- Silhouette sign-rank tests reproduce the published p-values: 5.7973e-171 for scATAC-seq and 5.7588e-08 for scRNA-seq
- Computed clustering accuracies against the FACS-bulk surrogate labels match the reported values for each strategy (Cell Ranger, Seurat V3, CoupledNMF)
- CoupledNMF ranks as the best-performing clustering method overall in the benchmark
- cnmf_tsne produces joint_clusters_plot.png and joint_data_type_plot.png in which cells of the same ground-truth label lie near each other
- Coupling matrix loads successfully from peak_gene_coupling_matrix.txt for the chosen genome version (hg19, hg38, mm9, or mm10)

## Limitations

- Surrogate ground truth is only valid because the constituent subpopulations were already analyzed by bulk samples of FACS-sorted cells; it is not a true ground truth
- Differential regulations discovered this way need replication in additional patients and controls
- The framework does not yet accommodate multiple donors per condition
- Requires installed system dependencies (Bedtools, Homer on Linux) and R >= 3.6.0; large data files require git lfs to download

## Evidence

- [intro] validation rationale via bulk-analyzed populations: "To validate a single-cell analysis method, we apply it to scRNA-seq and/or scATAC-seq samples from a heterogeneous population with constituent subpopulations that are already analyzed by bulk sample"
- [methods] peak preprocessing command with genome versions: "bash cnmf_process_data.sh path/to/peak.bed genome_version path/to/prior_data"
- [methods] running cnmf to obtain cluster labels: "Run `cnmf` to get the cluster labels for sample 1 and sample 2."
- [results] expected silhouette significance benchmarks: [evidence span withheld — anchor status: not_found]
- [methods] t-SNE visualization of clustering outputs: "`cnmf_tsne` allows users to visualize clustering outputs."
