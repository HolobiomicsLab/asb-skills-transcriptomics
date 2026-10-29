---
name: joint-tsne-embedding-visualization
description: Use when you have CoupledNMF H matrices (cnmf.output$H1, cnmf.output$H2)
  from a paired scRNA-seq/scATAC-seq run and need a joint embedding plot to check
  cluster coherence and cross-modality integration.
when_to_use_negative:
- You only have per-modality cluster labels from Cell Ranger or Seurat V3 label transfer
  and want separate t-SNEs — this skill specifically renders the CoupledNMF joint
  embedding
- You have not yet run CoupledNMF (no cnmf.output$H1/$H2), e.g. you are still at the
  peak preprocessing (cnmf_process_data.sh) or coupling-matrix loading step
- Your goal is downstream comparative regulatory network construction rather than
  embedding visualization — use the sc_compreg function workflow instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCompReg
  role: R package supplying the cnmf_tsne function that renders the joint t-SNE from
    CoupledNMF H matrices, alongside cnmf_process_data.sh and cnmf_load_coupling_matrix
    used to prepare its inputs
  repo: https://github.com/SUwonglab/sc-compReg
- name: R (>= 3.6.0)
  role: Runtime environment in which the scCompReg package (and cnmf_tsne call) is
    executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:9cc4e91dd7d079a085c05bef50f5cb196f422db48a4d843626c88a0528e1d903
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
  - scCompReg
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# joint-tsne-embedding-visualization

## Summary

Visualize the coupled clustering of paired scRNA-seq and scATAC-seq samples from sc-compReg's CoupledNMF as a joint t-SNE, producing cluster-colored and data-type-colored plots used to validate that cells sharing a surrogate ground-truth label co-localize.

## When to use

Use when you have CoupledNMF H matrices (cnmf.output$H1, cnmf.output$H2) from a paired scRNA-seq/scATAC-seq run and need a joint embedding plot to check cluster coherence and cross-modality integration.

## When NOT to use

- You only have per-modality cluster labels from Cell Ranger or Seurat V3 label transfer and want separate t-SNEs — this skill specifically renders the CoupledNMF joint embedding
- You have not yet run CoupledNMF (no cnmf.output$H1/$H2), e.g. you are still at the peak preprocessing (cnmf_process_data.sh) or coupling-matrix loading step
- Your goal is downstream comparative regulatory network construction rather than embedding visualization — use the sc_compreg function workflow instead

## Inputs

- cnmf.output$H1: CoupledNMF cluster-membership matrix for sample 1 (scRNA-seq cells)
- cnmf.output$H2: CoupledNMF cluster-membership matrix for sample 2 (scATAC-seq cells)
- path: directory for writing joint_clusters_plot.png and joint_data_type_plot.png
- FACS-sorted bulk cell-type signature labels (surrogate ground truth) for silhouette-based validation

## Outputs

- joint_clusters_plot.png: joint t-SNE of both samples colored by CoupledNMF cluster label
- joint_data_type_plot.png: joint t-SNE colored by data type (scRNA-seq vs scATAC-seq)
- silhouette index per embedding for comparison with Cell Ranger and Seurat V3 baselines (sign rank test p-values)

## How to apply

Run CoupledNMF first (after preprocessing peaks with cnmf_process_data.sh for the correct genome_version among hg19, hg38, mm9, mm10 and loading D <- cnmf_load_coupling_matrix('peak_gene_coupling_matrix.txt')) so that cnmf.output$H1 and cnmf.output$H2 hold the joint cluster memberships for sample 1 (scRNA-seq) and sample 2 (scATAC-seq). Pass both H matrices to cnmf_tsne with path set to the output directory, save.plot=F for interactive viewing or T to write files, and perplexity=100 as used in the healthy donor BMMC analysis. The function draws a joint t-SNE embedding that yields two outputs: joint_clusters_plot.png (cells colored by cluster label) and joint_data_type_plot.png (cells colored by data type), letting you confirm that cells of the same FACS-bulk-derived surrogate label sit near each other regardless of modality. To judge correctness quantitatively, compute a silhouette index against the surrogate ground truth labels and compare against separate Cell Ranger t-SNEs via a sign rank test; the published validation obtained p = 5.7973e-171 for scATAC-seq and p = 5.7588e-08 for scRNA-seq in favor of the joint embedding.

## Related tools

- **scCompReg** (R package supplying the cnmf_tsne function that renders the joint t-SNE from CoupledNMF H matrices, alongside cnmf_process_data.sh and cnmf_load_coupling_matrix used to prepare its inputs) — https://github.com/SUwonglab/sc-compReg
- **R (>= 3.6.0)** (Runtime environment in which the scCompReg package (and cnmf_tsne call) is executed)

## Examples

```
cnmf_tsne(cnmf.output$H1, cnmf.output$H2, path=path, save.plot=F, perplexity=100)
```

## Evaluation signals

- Both joint_clusters_plot.png and joint_data_type_plot.png are written to the specified path (or rendered when save.plot=F)
- In the joint t-SNE, cells sharing a FACS-bulk surrogate ground-truth label are placed near each other across modalities
- Silhouette index of the joint embedding is significantly higher than separate Cell Ranger t-SNEs (sign rank test; reference values p = 5.7973e-171 for scATAC-seq, p = 5.7588e-08 for scRNA-seq)
- Embedding is reproducible with the published parameter setting perplexity=100
- H1 and H2 have matching cluster-label structure so both modalities overlay in one shared t-SNE space

## Limitations

- Requires CoupledNMF outputs, which in turn depend on the peak-gene coupling matrix (peak_gene_coupling_matrix.txt) and prior_data for the chosen genome version; without that coupling the joint embedding cannot be formed
- cnmf_tsne is a visualization/evaluation step only — it does not itself cluster; poor upstream clustering will be faithfully displayed
- sc-compReg validation used single healthy-donor BMMC samples; the method does not yet accommodate multiple donors per condition, which constrains generalization of the embedding-based validation

## Evidence

- [methods] cnmf_tsne allows users to visualize clustering outputs.: "`cnmf_tsne` allows users to visualize clustering outputs."
- [abstract] provides joint clustering and embedding of the cells from both scRNA-seq and scATAC-seq: "provides joint clustering and embedding of the cells from both scRNA-seq and scATAC-seq"
- [readme] an R package that provides coupled clustering and joint embedding of scRNA-seq and scATAC-seq on one sample, and performs comparative gene regulatory analysis between two conditions.: "is an R package that provides coupled clustering and joint embedding of scRNA-seq and scATAC-seq on one sample, and performs comparative gene regulatory analysis between two conditions."
- [methods] D <- cnmf_load_coupling_matrix('peak_gene_coupling_matrix.txt'): "D <- cnmf_load_coupling_matrix('peak_gene_coupling_matrix.txt'),"
