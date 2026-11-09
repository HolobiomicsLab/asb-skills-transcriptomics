---
name: marker-gene-identification-for-immune-cell-populations
description: Use when an unsupervised clustering of scRNA-seq data (e.g., k-means
  on PCs of PBMCs) yields clusters whose biological identities must be determined
  from cluster-enriched genes such as CD3D, NKG7, CD79A, S100A8/S100A9, PF4, FCER1A.
when_to_use_negative:
- When cell populations are already annotated by orthogonal methods (e.g., flow cytometry
  panels or cell-type correlation assignment to purified reference populations) —
  marker discovery adds no identity information.
- When the tissue is not blood/immune-derived (e.g., tumor or solid-tissue scRNA-seq),
  since the canonical PBMC marker set (CD3D, CD79A, S100A8/S100A9, PF4, FCER1A) will
  not map onto the relevant populations.
- When the input is raw sequencing data needing barcode processing first — Cell Ranger
  demultiplexing/counting must be completed before marker gene identification.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Seurat
  role: R framework in which the log-transformed 68k PBMC gene-cell-barcode matrix
    is analyzed for PCA, k-means clustering, and per-cluster marker gene identification
  repo: https://github.com/10XGenomics/single-cell-3prime-paper
- name: Cell Ranger Single-Cell Software Suite
  role: Upstream pipeline producing the gene-cell-barcode count matrix from which
    marker genes are computed
  repo: https://github.com/10XGenomics/cellranger
- name: 10XGenomics/single-cell-3prime-paper R code
  role: Reference implementation of the 68k PBMC analysis (variable gene selection,
    PCA, k-means, marker gene tables)
  repo: https://github.com/10XGenomics/single-cell-3prime-paper
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:d32f7f7ca61aaa39c4f92ddb6f0d3796f2b05555fc3e9e883bd27c6dda9724d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/single-cell-3prime-paper
  tools:
  - 10XGenomics/single-cell-3prime-paper R code
  - Cell Ranger Single-Cell Software Suite
  - Seurat
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# marker-gene-identification-for-immune-cell-populations

## Summary

Identify cluster-specific marker genes from single-cell 3′ counting data of peripheral blood mononuclear cells (e.g., the 68k PBMC dataset) to assign canonical immune cell identities (T cells, NK cells, B cells, monocytes, dendritic cells, megakaryocytes/platelets) to unsupervised clusters.

## When to use

Use when an unsupervised clustering of scRNA-seq data (e.g., k-means on PCs of PBMCs) yields clusters whose biological identities must be determined from cluster-enriched genes such as CD3D, NKG7, CD79A, S100A8/S100A9, PF4, FCER1A.

## When NOT to use

- When cell populations are already annotated by orthogonal methods (e.g., flow cytometry panels or cell-type correlation assignment to purified reference populations) — marker discovery adds no identity information.
- When the tissue is not blood/immune-derived (e.g., tumor or solid-tissue scRNA-seq), since the canonical PBMC marker set (CD3D, CD79A, S100A8/S100A9, PF4, FCER1A) will not map onto the relevant populations.
- When the input is raw sequencing data needing barcode processing first — Cell Ranger demultiplexing/counting must be completed before marker gene identification.

## Inputs

- Gene-cell-barcode matrix of single-cell 3′ UMI counts (e.g., 68k PBMC matrix from support.10xgenomics.com/single-cell/datasets)
- Log-transformed expression matrix loaded into Seurat
- Per-cell cluster assignments from k-means clustering (k=10) on the first 50 PCs
-  hg19-referenced cell-level expression matrix produced by Cell Ranger

## Outputs

- Per-cluster marker gene table with top cluster-specific genes for each of the 10 clusters
- Per-cell cluster assignment table
- Combined per-cell cluster assignments and per-cluster marker gene table written to pbmc_clusters_markers.csv
- Labeled PBMC subpopulation identities (T cells, NK cells, B cells, monocytes, megakaryocytes/platelets, dendritic cells)

## How to apply

After clustering cells (e.g., k-means with k=10 on the first 50 principal components of the top 1,000 most variable genes), compute per-cluster enriched/cluster-specific genes from the log-transformed gene-cell-barcode matrix. Match the top markers against canonical PBMC population markers: CD3D for T cells, NKG7 for NK cells, CD79A for B cells, S100A8/S100A9 for monocytes, PF4 for megakaryocytes/platelets, and FCER1A for dendritic cells. The article applied this to 68k fresh PBMCs profiled with the GemCode droplet system and processed with Cell Ranger, finding >80% T cells enriched for CD3D in clusters 1–3 and 6, at expected ratios. Use Seurat as the analysis framework, with the R analysis code from github.com/10XGenomics/single-cell-3prime-paper as a reference implementation. Write per-cell cluster assignments and the per-cluster marker gene table to a CSV output (e.g., pbmc_clusters_markers.csv) so marker profiles can be validated against the paper's reported populations.

## Related tools

- **Seurat** (R framework in which the log-transformed 68k PBMC gene-cell-barcode matrix is analyzed for PCA, k-means clustering, and per-cluster marker gene identification) — https://github.com/10XGenomics/single-cell-3prime-paper
- **Cell Ranger Single-Cell Software Suite** (Upstream pipeline producing the gene-cell-barcode count matrix from which marker genes are computed) — https://github.com/10XGenomics/cellranger
- **10XGenomics/single-cell-3prime-paper R code** (Reference implementation of the 68k PBMC analysis (variable gene selection, PCA, k-means, marker gene tables)) — https://github.com/10XGenomics/single-cell-3prime-paper

## Examples

```
Rscript pbmc_analysis.R  # from github.com/10XGenomics/single-cell-3prime-paper; writes pbmc_clusters_markers.csv
```

## Evaluation signals

- The recovered clusters' top marker genes include canonical PBMC markers: CD3D (T cells), NKG7 (NK cells), CD79A (B cells), S100A8/S100A9 (monocytes), PF4 (megakaryocytes/platelets), FCER1A (dendritic cells).
- K-means with k=10 on the first 50 PCs of the top 1,000 most variable genes recovers 10 distinct clusters whose compositions match the paper's PBMC analysis.
- Cluster-level cell type proportions are at expected biological ratios, including >80% T cells enriched for CD3D in clusters 1–3 and 6.
- Per-cell cluster assignments and per-cluster marker tables are written to pbmc_clusters_markers.csv in a parseable format.
- Cell assignment confidence is checked by ensuring the top and second-highest population correlations are not trivially close (e.g., cytotoxic T vs NK).

## Limitations

- Cell type assignment by correlation was low-confidence for some cells (e.g., cytotoxic T vs NK cells) because the difference between the highest and second-highest correlation was small.
- Marker-based annotation is only as good as the underlying count matrix; barcodes/UMIs must pass the paper's QC filters (MAPQ=255, non-PCR duplicates, quality score >10 UMIs, posterior probability >=0.975 barcode correction) for markers to be reliable.
- The published analysis uses only the top 1,000 most variable genes for PCA, so markers outside this set may not drive the clustering even if they are cluster-enriched.
- Some libraries in the dataset used 5 nt UMI and 5 bp Read2, introducing protocol heterogeneity that could affect cross-library marker comparisons.

## Evidence

- [discussion] The top 1,000 most variable genes were identified based on their mean and dispersion (variance/mean), which is similar to the approach used by Macoscko et al.7: "The top 1,000 most variable genes were identified based on their mean and dispersion (variance/mean), which is similar to the approach used by Macoscko et al.7"
- [discussion] The gene-cell-barcode matrix of 68k PBMCs was log-transformed as an input to Seurat.: "The gene-cell-barcode matrix of 68k PBMCs was log-transformed as an input to Seurat."
- [discussion] R analysis code from 10XGenomics/single-cell-3prime-paper on GitHub: "The R code used to analyse 68k PBMCs and purified PBMCs can be found here: https://github.com/10XGenomics/single-cell-3prime-paper."
- [discussion] Only confidently mapped (MAPQ=255), non-PCR-duplicate reads with valid barcodes/UMIs used for gene-barcode matrix: "Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix."
- [readme] README describing downloadable data location: [evidence span withheld — anchor status: not_found]
