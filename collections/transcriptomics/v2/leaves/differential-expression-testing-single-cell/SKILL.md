---
name: differential-expression-testing-single-cell
description: Use when you have a single-cell TAR or gene expression matrix with cell-cluster
  labels and need to identify features (uTARs or genes) that are differentially expressed
  per cluster, e.g. to direct annotation efforts to biologically significant transcripts.
when_to_use_negative:
- Your input is already a gene-level feature table derived from a complete, high-quality
  genome annotation and no novel transcripts (uTARs) are of interest — standard scRNA-seq
  pipelines suffice.
- You lack cell-cluster labels and have not performed clustering; Wilcoxon rank-sum
  differential testing requires pre-defined cell groups per cluster.
- You are starting from raw sequencing files and only need alignment or TAR calling
  — use the STAR/bedtools/groHMM stage of the Snakemake workflow instead.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: Seurat
  role: Single-cell analysis framework in which the TAR expression matrix is analyzed
    and the Wilcoxon rank-sum test is applied to identify differentially expressed
    uTARs per cell cluster
- name: featureCounts
  role: Counts TAR features per cell (strandedness -s 1, paired-end -p) to aggregate
    the TAR expression matrix fed into differential testing
- name: groHMM
  role: Hidden Markov model that identifies transcriptionally active regions from
    scRNA-seq read coverage, producing the aTAR/uTAR features being differentially
    tested
  repo: https://bioconductor.org/packages/release/bioc/html/groHMM.html
- name: rtracklayer
  role: R package used to load genome annotation/TAR files when labeling TARs as aTAR
    or uTAR
  repo: https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- name: Snakemake
  role: Workflow manager for the published from_fastqs and from_cellranger pipelines
    that generate the TAR expression matrix upstream of differential testing
  repo: https://github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:e38b60595e20c4d82468ba65c737c4b3272b09e1244954d39c94beb33e1f2c2c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - STAR
  - Seurat
  - Snakemake
  - bedtools
  - featureCounts
  - groHMM
  - rtracklayer
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# differential-expression-testing-single-cell

## Summary

Performs cluster-level differential expression testing on single-cell expression matrices — including matrices built from unannotated transcriptionally active regions (uTARs) — using Seurat's Wilcoxon rank-sum test, then annotates significant uTARs by sequence homology to recover cell-type-specific transcriptional dark matter.

## When to use

Use when you have a single-cell TAR or gene expression matrix with cell-cluster labels and need to identify features (uTARs or genes) that are differentially expressed per cluster, e.g. to direct annotation efforts to biologically significant transcripts.

## When NOT to use

- Your input is already a gene-level feature table derived from a complete, high-quality genome annotation and no novel transcripts (uTARs) are of interest — standard scRNA-seq pipelines suffice.
- You lack cell-cluster labels and have not performed clustering; Wilcoxon rank-sum differential testing requires pre-defined cell groups per cluster.
- You are starting from raw sequencing files and only need alignment or TAR calling — use the STAR/bedtools/groHMM stage of the Snakemake workflow instead.

## Inputs

- TAR-by-cell expression count matrix (from featureCounts on groHMM-derived TARs, strandedness -s 1, paired-end -p)
- Cell cluster labels from Seurat clustering (e.g. spermatocytes, spermatogonium, spermatids)
- groHMM TAR annotations labeled as aTAR or uTAR by overlap with gene annotations (refFlat and gtf formats)

## Outputs

- Table of differentially expressed uTARs per cell cluster with fold-change and percent-expressed statistics (e.g. ss2_differential_utars.csv)
- Homology-based functional labels for top differentially expressed uTARs (e.g. CCER1, TBXAS1, LOC105867359)

## How to apply

Build a TAR expression matrix by counting transcriptionally active regions per cell with featureCounts (strandedness -s 1, paired-end option -p for Smart-seq2 data), then import the count matrix into Seurat and cluster cells (e.g. spermatocytes, spermatogonium, spermatids in the gray mouse lemur testes case). Apply the Wilcoxon rank-sum test per cluster to identify differentially expressed uTARs versus all other cells, filtering to features expressed in at least 25% of cluster cells with more than 0.25 log-e fold change. Take the top differentially expressed uTARs per cluster and label them by nucleotide sequence homology, since uTARs lie outside existing gene annotations. Validation relies on comparing assignments to known biology, e.g. CCER1 in spermatocytes, TBXAS1 in spermatogonium, and ncRNA LOC105867359 in spermatids matching Supplementary Fig. 2b.

## Related tools

- **Seurat** (Single-cell analysis framework in which the TAR expression matrix is analyzed and the Wilcoxon rank-sum test is applied to identify differentially expressed uTARs per cell cluster)
- **featureCounts** (Counts TAR features per cell (strandedness -s 1, paired-end -p) to aggregate the TAR expression matrix fed into differential testing)
- **groHMM** (Hidden Markov model that identifies transcriptionally active regions from scRNA-seq read coverage, producing the aTAR/uTAR features being differentially tested) — https://bioconductor.org/packages/release/bioc/html/groHMM.html
- **rtracklayer** (R package used to load genome annotation/TAR files when labeling TARs as aTAR or uTAR) — https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- **Snakemake** (Workflow manager for the published from_fastqs and from_cellranger pipelines that generate the TAR expression matrix upstream of differential testing) — https://github.com/fw262/TAR-scRNA-seq

## Examples

```
markers <- FindMarkers(seurat_utars, ident.1 = "spermatocytes", logfc.threshold = 0.25, min.pct = 0.25, test.use = "wilcox")
```

## Evaluation signals

- Differential uTAR table reproduces the reported cell-type assignments: CCER1 in spermatocytes, TBXAS1 in spermatogonium, and ncRNA LOC105867359 in spermatids (Supplementary Fig. 2b).
- All reported uTARs pass the filters: expressed in at least 25% of cells in the cluster and more than 0.25 log-e fold change versus the rest of the cells.
- Each entry in the output table is a genuine uTAR (i.e. its TAR does not overlap existing gene annotations), not an aTAR.
- Top differentially expressed uTARs receive homology-based labels with meaningful biological assignments consistent with cluster identity (e.g. spermatid-expressed ncRNA).

## Limitations

- Differential uTAR hits cannot be named directly from annotations; they must be labeled by nucleotide sequence homology, which may fail for novel or poorly conserved transcripts.
- The degree of relevant information in uTARs is strongly dependent on organism, tissue, cell type, and stage of development, so differential uTAR results may be sparse in well-annotated contexts.
- groHMM bed files must be generated successfully before counting and testing; a missing-file error ('cannot read: chr*_HMM.bed') indicates the groHMM R script did not finish running.

## Evidence

- [abstract] performs single-cell TAR expression analysis to identify biologically significant TARs, and then annotates TARs using gene homology analysis: "performs single-cell TAR expression analysis to identify biologically significant TARs, and then annotates TARs using gene homology analysis"
- [intro] uTARs that are differentially expressed across cell clusters are retrieved and annotated based on nucleotide sequence homology: "uTARs that are differentially expressed across different cell clusters are retrieved and annotated based on nucleotide sequence homology"
- [readme] This error indicates that the groHMM R script did not finish running to generate groHMM bed files.: "This error indicates that the groHMM R script did not finish running to generate groHMM bed files."
