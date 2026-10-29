---
name: cross-genome-scrna-seq-comparison
description: Use when comparing how two TE-expression quantification tools (e.g.,
  SoloTE vs scTE) recover known cell populations across two genome builds (hg38, mm10)
  using Seurat clustering and UMAP on simulated or ground-truth matrices.
when_to_use_negative:
- Input is already a clustered result or dimensionality-reduced embedding rather than
  a gene/TE cell expression matrix requiring Seurat preprocessing.
- The goal is locus-specific TE quantification from raw scRNA-Seq alignments (BAM)
  — that is SoloTE's own pipeline role, not this downstream cross-tool clustering
  comparison.
- Analysis targets bulk RNA-Seq or other TEs than those classifiable as young/old
  by consensus divergence ≤10%/ >10%.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Normalization, variable-feature identification, scaling, PCA, UMAP dimensional
    reduction, and graph-based clustering of the ground-truth/SoloTE/scTE matrices
- name: SoloTE
  role: Produces the locus-specific gene+TE expression matrix (test case) whose clustering
    is compared against ground truth and scTE
  repo: github.com/bvaldebenitom/SoloTE
- name: scTE
  role: Family-level (metagene) TE expression summarizer benchmarked against SoloTE
    in the clustering/UMAP comparison
- name: BEDTools
  role: SoloTE dependency (v2.29.2 or higher) used upstream to assess read overlap
    with TE annotation
  repo: github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:8a1eedcdbdef7e5bb0da1e1f8427c1b7cd5b15bf767daa8a7b23a95d1c17eed8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BEDTools
  - Seurat
  - SoloTE
  - scTE
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-genome-scRNA-Seq-comparison

## Summary

Benchmark workflow that processes ground-truth, SoloTE, and scTE gene/TE cell expression matrices from simulated hg38 and mm10 scRNA-Seq experiments through Seurat's standard preprocessing, PCA, UMAP, and graph-based clustering, then judges tool performance by how many of the four expected cell clusters are recovered, stratified by TE age (young: divergence from consensus ≤10%; old: >10%).

## When to use

Use when comparing how two TE-expression quantification tools (e.g., SoloTE vs scTE) recover known cell populations across two genome builds (hg38, mm10) using Seurat clustering and UMAP on simulated or ground-truth matrices.

## When NOT to use

- Input is already a clustered result or dimensionality-reduced embedding rather than a gene/TE cell expression matrix requiring Seurat preprocessing.
- The goal is locus-specific TE quantification from raw scRNA-Seq alignments (BAM) — that is SoloTE's own pipeline role, not this downstream cross-tool clustering comparison.
- Analysis targets bulk RNA-Seq or other TEs than those classifiable as young/old by consensus divergence ≤10%/ >10%.

## Inputs

- Ground-truth gene/TE cell expression matrix from simulated hg38 dataset
- Ground-truth gene/TE cell expression matrix from simulated mm10 dataset
- SoloTE-output combined gene and TE expression matrix per cell (compatible with Seurat)
- scTE-output TE expression matrix
- TE age class labels (divergence from consensus: ≤10% young, >10% old)
- Ground-truth cell cluster labels for the four expected clusters

## Outputs

- Per-cell cluster assignments for each matrix (ground-truth, SoloTE, scTE) per genome
- UMAP embeddings per matrix
- Count of recovered clusters per condition, stratified by genome (hg38, mm10) and TE age (young/old)
- Comparison table of recovered vs expected clusters against ground-truth labels

## How to apply

Load the ground-truth, SoloTE, and scTE expression matrices from the simulated hg38 and mm10 experiments into R (v4 or higher) and build Seurat objects. Run Seurat's standard preprocessing: normalize counts, identify variable features, scale the gene/TE features, then run PCA followed by UMAP dimensional reduction on each matrix. Cluster cells with Seurat's graph-based clustering and record per-cell cluster assignments for each matrix. Stratify results by TE age class — young if divergence from the consensus sequence is ≤10%, old if >10% — and by genome (hg38, mm10), counting recovered clusters per condition. Compare the recovered clusters and assignments against the four expected clusters and the ground-truth labels for the SoloTE and scTE matrices; the published result is that old TEs yield four clusters with either tool in both genomes, SoloTE recovers the four young-TE clusters, whereas scTE young-TE clusters are closer together and even mixed.

## Related tools

- **Seurat** (Normalization, variable-feature identification, scaling, PCA, UMAP dimensional reduction, and graph-based clustering of the ground-truth/SoloTE/scTE matrices)
- **SoloTE** (Produces the locus-specific gene+TE expression matrix (test case) whose clustering is compared against ground truth and scTE) — github.com/bvaldebenitom/SoloTE
- **scTE** (Family-level (metagene) TE expression summarizer benchmarked against SoloTE in the clustering/UMAP comparison)
- **BEDTools** (SoloTE dependency (v2.29.2 or higher) used upstream to assess read overlap with TE annotation) — github.com/arq5x/bedtools2

## Examples

```
python SoloTE_pipeline.py --threads NumberOfThreads --bam BAMfile --teannotation BEDfile --outputprefix Prefix --outputdir OutputDirectory
```

## Evaluation signals

- Exactly four expected clusters are recovered for old TEs (divergence >10%) in both hg38 and mm10 with either tool.
- SoloTE matrices for young TEs (divergence ≤10%) recover the four expected clusters matching ground-truth labels.
- scTE young-TE clusters appear closer together or mixed in the UMAP, diverging from the four-cluster ground truth — this negative control must be reproduced.
- Cluster counts must be reported per condition (genome × tool × TE age), and per-cell assignments must be comparable to ground-truth labels.
- Seurat run must follow the standard preprocessing chain (normalization → variable features → scaling → PCA → UMAP → graph-based clustering) consistently across all matrices so the comparison is fair.

## Limitations

- scTE indexes do not filter non-TE repeats such as Satellites, so non-TE repeats can appear in the resulting matrix.
- scTE summarizes TE expression on metagenes corresponding to TE families, losing the genomic location from which each TE is expressed.
- Results were established on simulated data from hg38 and mm10; in organisms with a greater proportion of transpositionally-active TEs, most TE copies could be genetically intact and multi-mapping, which SoloTE's locus-level assignment (MAPQ ≥ threshold, default 255 = uniquely mapped reads) may not resolve.
- SoloTE's EM-algorithm performance on 3' end 10X data has not been assessed, so this comparison design should not be extrapolated to 3'-biased protocols without revalidation.

## Evidence

- [discussion] TEs were divided into young and old, if their divergence from the consensus sequence was ≤10% or >10%, respectively: "TEs were divided into young and old, if their divergence from the consensus sequence was ≤10% or >10%, respectively"
- [discussion] simulated hg38/mm10 data: "we simulated data from the human genome hg38 version, and from the mouse genome mm10 version"
- [discussion] Seurat-compatible output matrix: "a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools, such as Seurat67"
- [intro] scTE benchmark objective: "First, we compare it to scTE, in terms of (i) detecting TE expression, (ii) UMAP dimensional reduction, and (iii) detection of cell marker TEs."
- [readme] README SoloTE dependencies: [evidence span withheld — anchor status: not_found]
- [readme] README TE age in BED IDs: "column 4, the ID, is a concatenation of the locus of the TE and its identifiers at the Subfamily, Family and Class level"
