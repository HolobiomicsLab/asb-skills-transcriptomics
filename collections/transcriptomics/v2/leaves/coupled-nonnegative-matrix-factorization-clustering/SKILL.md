---
name: coupled-nonnegative-matrix-factorization-clustering
description: Use when you have one scRNA-seq and one scATAC-seq sample of the same
  heterogeneous population (e.g. healthy donor BMMC) and need consistent cluster assignments
  across both modalities before running sc_compreg, rather than clustering each modality
  separately.
when_to_use_negative:
- You already have consistent cluster assignments in scRNA-seq and scATAC-seq from
  elsewhere — the README states 'It is not required to obtain cluster assignments
  using the coupled nonnegative matrix factorization workflow'; sc_compreg accepts
  any consistent assignments.
- You only have a single modality (scRNA-seq or scATAC-seq alone) — the coupling term
  A_ij requires paired peaks and genes linked across both samples.
- Your genome build is not one of {hg19, hg38, mm9, mm10} — cnmf_process_data.sh and
  the prior_data directory only support these versions.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: sc-compReg
  role: Provides the CoupledNMF implementation (cnmf_process_data.sh, cnmf_load_coupling_matrix,
    cnmf_tsne) and is the package for which this clustering supplies the default cluster
    labels
  repo: https://github.com/SUwonglab/sc-compReg
- name: Bedtools
  role: Linux dependency used by the cnmf preprocessing script (cnmf_process_data.sh)
    to build the peak-gene coupling matrix
- name: Homer
  role: Linux dependency of the sc-compReg preprocessing pipeline supporting the cnmf
    workflow
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:a12a655a41df50fe4fc0d9399b6572407af13acfb96a88a17d1a7ed06bc151f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/sc-compReg
  tools:
  - Bedtools
  - Homer
  - R (>=)
  - sc-compReg
  - scCompReg
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# coupled-nonnegative-matrix-factorization-clustering

## Summary

Joint clustering of paired scRNA-seq and scATAC-seq cells via the CoupledNMF algorithm from sc-compReg, which factorizes both modalities with a shared basis W and a peak-gene coupling prior so RNA and ATAC cells receive consistent cluster labels for downstream comparative regulatory analysis.

## When to use

Use when you have one scRNA-seq and one scATAC-seq sample of the same heterogeneous population (e.g. healthy donor BMMC) and need consistent cluster assignments across both modalities before running sc_compreg, rather than clustering each modality separately.

## When NOT to use

- You already have consistent cluster assignments in scRNA-seq and scATAC-seq from elsewhere — the README states 'It is not required to obtain cluster assignments using the coupled nonnegative matrix factorization workflow'; sc_compreg accepts any consistent assignments.
- You only have a single modality (scRNA-seq or scATAC-seq alone) — the coupling term A_ij requires paired peaks and genes linked across both samples.
- Your genome build is not one of {hg19, hg38, mm9, mm10} — cnmf_process_data.sh and the prior_data directory only support these versions.

## Inputs

- peak.bed file of scATAC-seq peaks
- genome_version (one of hg19, hg38, mm9, mm10)
- prior_data directory (cloned from github.com/SUwonglab/sc-compReg)
- peak_gene_coupling_matrix.txt output of cnmf_process_data.sh
- peak.name (peak names in bed format: chr\tstart\tend) and symbol (gene symbols) for both samples
- scRNA-seq and scATAC-seq count matrices for sample 1 and sample 2

## Outputs

- cnmf.output object with factor matrices H1 (RNA cells) and H2 (ATAC cells) and cluster indices, savable as cnmf_output.rds
- Consistent per-cell cluster labels for sample 1 and sample 2, passable to sc_compreg as O1.idx, E1.idx, O2.idx, E2.idx
- joint_clusters_plot.png and joint_data_type_plot.png from cnmf_tsne visualization

## How to apply

First preprocess peaks with 'bash cnmf_process_data.sh path/to/peak.bed genome_version path/to/prior_data' (genome_version in {hg19, hg38, mm9, mm10}) to produce peak_gene_coupling_matrix.txt, then load it in R as the coupling matrix D via cnmf_load_coupling_matrix('peak_gene_coupling_matrix.txt', peak.name, symbol). Implement the multiplicative updates for W1, H1, W2, H2 with the coupling term A_ij = R_ij+ * exp(-d_ij/d0), where d0 = 40 kb is the distance-decay scale linking ATAC peaks to co-located genes. Run the coupled optimization over the decreasing alpha series [10000, 1000, 100, 10, 1, 0.1, 0.01, 0.001] (with the corresponding beta scheme, a criterion adapted for 10x Genomics Chromium data) and select the alpha/beta combination giving the most interpretable joint factorization. Extract per-cell cluster labels from H1 (RNA) and H2 (ATAC), save as cnmf_output.rds, and pass them directly to sc_compreg as O1.idx, E1.idx, O2.idx, E2.idx. Optionally visualize with cnmf_tsne(cnmf.output$H1, cnmf.output$H2, path=path, save.plot=F, perplexity=100) to generate joint_clusters_plot.png and joint_data_type_plot.png. Validate against FACS-bulk-derived surrogate labels: the joint t-SNE embedding should achieve a higher silhouette index than separate Cell Ranger t-SNEs (sign rank test p = 5.7973e-171 for scATAC-seq, p = 5.7588e-08 for scRNA-seq), and CoupledNMF should be the best-performing clustering method versus Cell Ranger and Seurat V3 label transfer.

## Related tools

- **sc-compReg** (Provides the CoupledNMF implementation (cnmf_process_data.sh, cnmf_load_coupling_matrix, cnmf_tsne) and is the package for which this clustering supplies the default cluster labels) — https://github.com/SUwonglab/sc-compReg
- **Bedtools** (Linux dependency used by the cnmf preprocessing script (cnmf_process_data.sh) to build the peak-gene coupling matrix)
- **Homer** (Linux dependency of the sc-compReg preprocessing pipeline supporting the cnmf workflow)

## Examples

```
bash cnmf_process_data.sh path/to/peak.bed hg38 path/to/prior_data
```

## Evaluation signals

- The stored cnmf_output.rds cluster assignments load in R and can be passed directly to sc_compreg as O1.idx, E1.idx, O2.idx, E2.idx with consistent labels across RNA and ATAC modalities.
- Clustering accuracy against FACS-bulk surrogate ground-truth labels matches the reported validation, and the joint t-SNE silhouette test reproduces p = 5.7973e-171 for scATAC-seq and p = 5.7588e-08 for scRNA-seq against Cell Ranger.
- cnmf_tsne with perplexity=100 produces joint_clusters_plot.png and joint_data_type_plot.png in which cells of the same ground-truth label lie near each other in the joint embedding.
- cnmf_load_coupling_matrix returns a D matrix whose dimensions align with peak.name and symbol for the two samples.
- The selected alpha/beta pair from the decreasing series [10000 ... 0.001] yields the most interpretable joint factorization of the 10x Genomics Chromium data.

## Limitations

- CoupledNMF clustering is optional for sc-compReg: any consistent cluster assignments in scRNA-seq and scATAC-seq can be supplied to sc_compreg instead.
- The coupling term assumes a 40 kb distance-decay scale (d0) linking ATAC peaks to co-located genes; peak-gene priors depend on the provided prior_data and supported genome builds (hg19, hg38, mm9, mm10).
- The surrogate-ground-truth validation was performed on healthy donor BMMC; differential regulations and downstream comparisons need replication in additional patients and controls, and the method does not yet accommodate multiple donors per condition.

## Evidence

- [abstract] provides joint clustering and embedding of the cells from both scRNA-seq and scATAC-seq: "provides joint clustering and embedding of the cells from both scRNA-seq and scATAC-seq"
- [readme] cnmf preprocessing command: "bash cnmf_process_data.sh path/to/peak.bed genome_version path/to/prior_data"
- [methods] cnmf cluster labels feed sc_compreg: "Run `cnmf` to get the cluster labels for sample 1 and sample 2. The cluster labels should be passed to `sc_compreg` as `O1.idx`, `E1.idx`, `O2.idx`, and `E2.idx`."
- [methods] coupling matrix loading: "D <- cnmf_load_coupling_matrix('peak_gene_coupling_matrix.txt'),
                                           peak.name,
                                           symbol)"
- [readme] cluster assignments are optional: "Note: It is not required to obtain cluster assignments using the coupled nonnegative matrix factorization workflow."
