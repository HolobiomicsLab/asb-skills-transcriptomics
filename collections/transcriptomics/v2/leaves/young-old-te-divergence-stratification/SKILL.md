---
name: young-old-te-divergence-stratification
description: Use when TE loci carry divergence-from-consensus values in a RepeatMasker-derived
  BED annotation and cluster recovery of young vs old TEs must be assessed separately
  across genomes or TE-detection tools.
when_to_use_negative:
- Your TE annotation lacks divergence-from-consensus scores (e.g. a custom BED without
  the divergence field), since young/old classes cannot be assigned with the ≤10%
  / >10% rule
- Your analysis targets gene expression only, with no TE features in the expression
  matrix to stratify
- You need to detect TEs acting as alternative transcription start sites in 3'-end
  10X data — the article notes these events cannot be detected due to 3' bias, so
  age-stratified clustering of such data is not meaningful
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SoloTE
  role: Produces the locus-specific gene+TE expression matrix and the TE BED annotation
    carrying divergence values used to define young/old classes
  repo: https://github.com/bvaldebenitom/SoloTE
- name: Seurat
  role: Performs normalization, PCA, UMAP dimensional reduction, and graph-based clustering
    whose per-condition cluster counts are stratified by TE age class
- name: scTE
  role: Comparator tool summarizing TE expression on family-level metagenes; its young-TE
    cluster behavior is benchmarked against SoloTE under the same stratification
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:85d19a6c74ee801b12896fcdb1d70b275f048d7d38287def907656daef37fdd9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - Seurat
  - SoloTE
  - scTE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# young-old-te-divergence-stratification

## Summary

Stratifies transposable elements into young (divergence from consensus ≤10%) and old (>10%) age classes, then evaluates whether downstream single-cell TE expression analyses (Seurat clustering/UMAP) recover the expected clusters separately for each age class and genome.

## When to use

Use when TE loci carry divergence-from-consensus values in a RepeatMasker-derived BED annotation and cluster recovery of young vs old TEs must be assessed separately across genomes or TE-detection tools.

## When NOT to use

- Your TE annotation lacks divergence-from-consensus scores (e.g. a custom BED without the divergence field), since young/old classes cannot be assigned with the ≤10% / >10% rule
- Your analysis targets gene expression only, with no TE features in the expression matrix to stratify
- You need to detect TEs acting as alternative transcription start sites in 3'-end 10X data — the article notes these events cannot be detected due to 3' bias, so age-stratified clustering of such data is not meaningful

## Inputs

- Locus-level TE annotation in SoloTE BED format with divergence-from-consensus values (RepeatMasker *rm.out converted via SoloTE_RepeatMasker_to_BED.py)
- Gene/TE cell-by-feature expression matrices from ground-truth, SoloTE, and scTE for simulated hg38 and mm10 datasets (BAM inputs from CellRanger or STAR)

## Outputs

- Per-cell cluster assignments from Seurat graph-based clustering, stratified by TE age class (young/old) and genome (hg38/mm10)
- Cluster-recovery counts per condition (ground-truth vs SoloTE vs scTE; young vs old) compared against the four expected clusters and ground-truth labels

## How to apply

First obtain a locus-level TE annotation in the SoloTE BED format, whose ID column concatenates the locus and Subfamily:Family:Class identifiers together with the divergence-from-consensus value (e.g. chr1|11505|11675|L1MC5a:L1:LINE|25.1). Classify each TE as young if divergence from the consensus is ≤10% and old if divergence is >10%. Process the ground-truth, SoloTE, and scTE gene/TE expression matrices for hg38 and mm10 through Seurat's standard preprocessing (normalization, variable features, scaling), run PCA followed by UMAP, and cluster cells with graph-based clustering. Then record per-cell cluster assignments, stratify by TE age class and genome, and compare the recovered clusters against the four expected clusters and ground-truth labels. The rationale is that young TEs are the hardest case for locus-specific mapping, so age-stratified evaluation exposes where family-level metagene tools like scTE lose resolution relative to SoloTE.

## Related tools

- **SoloTE** (Produces the locus-specific gene+TE expression matrix and the TE BED annotation carrying divergence values used to define young/old classes) — https://github.com/bvaldebenitom/SoloTE
- **Seurat** (Performs normalization, PCA, UMAP dimensional reduction, and graph-based clustering whose per-condition cluster counts are stratified by TE age class)
- **scTE** (Comparator tool summarizing TE expression on family-level metagenes; its young-TE cluster behavior is benchmarked against SoloTE under the same stratification)

## Examples

```
python SoloTE_RepeatMasker_to_BED.py -g hg38
```

## Evaluation signals

- Exactly the four expected clusters are recovered for old TEs (divergence >10%) in both hg38 and mm10 with either tool
- SoloTE clustering of young TEs (divergence ≤10%) recovers the 4 expected clusters matching ground-truth labels
- scTE young-TE clusters are expected to appear closer together or mixed relative to the four expected clusters
- Every TE in the annotation is assigned to exactly one age class using the ≤10% / >10% divergence cutoff with no unclassified loci
- Per-cell cluster assignments are recorded for all six matrices (3 tools × 2 genomes) before stratification

## Limitations

- The ≤10% divergence threshold is a binary young/old split; finer age structure is not captured
- For organisms with a greater proportion of transpositionally-active TEs, most copies could be genetically intact and multi-mapping, affecting locus-level assignment of young TEs
- CELLO-Seq was unable to map reads to specific genomic locations for some 'very young TEs', illustrating the inherent mapping difficulty at the young end of the age spectrum

## Evidence

- [discussion] young/old classification rule: "TEs were divided into young and old, if their divergence from the consensus sequence was ≤10% or >10%, respectively"
- [discussion] SoloTE output compatible with Seurat: "a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools, such as Seurat67"
- [other] finding on young-TE cluster recovery: [evidence span withheld — anchor status: not_found]
- [readme] BED ID carries divergence values used for stratification: "column 4, the ID, is a concatenation of the locus of the TE and its identifiers at the Subfamily, Family and Class level."
- [other] workflow stratification step: [evidence span withheld — anchor status: not_found]
