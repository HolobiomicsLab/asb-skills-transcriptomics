---
name: transposable-element-expression-matrix-analysis
description: Use when scRNA-Seq BAM files (CellRanger or STAR output) need TE expression
  quantified per genomic locus, or when comparing locus-specific vs family-metagene
  TE quantification for cell-type marker discovery.
when_to_use_negative:
- The input is already a gene/TE feature-count matrix — quantification is done; proceed
  directly to Seurat marker analysis
- The goal is to detect TEs acting as alternative transcription start sites — 3'-end
  10X data cannot detect these due to 3' bias
- A non-TE repeat analysis (e.g. Satellites) is needed — note scTE indexes do not
  filter non-TE repeats, so family-metagene matrices may contain non-TE repeat rows
  that must be excluded
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0654
tools:
- name: SoloTE
  role: 'Core pipeline: selects unassigned reads, overlaps with TE BED annotation,
    summarizes TE expression at locus level (MAPQ >= 255) and family level (multi-mappers),
    outputs Seurat-compatible matrix'
  repo: https://github.com/bvaldebenitom/SoloTE
- name: scTE
  role: 'Benchmark comparator: summarizes TE expression on TE-family metagenes, losing
    genomic location; underestimates log2(FC) marker values'
- name: Seurat
  role: Downstream marker analysis of the combined gene+TE matrix via FindAllMarkers
    with adjusted P-value <= 0.05 and average log2(FC) >= 1
- name: BEDTools
  role: Assesses read overlap with TE annotation (v2.29.2 or higher required by SoloTE)
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:a95ea6cfddef297594d1b55a7a1c179efe03ceedb23c490c0bbdc18e77800a5b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - BEDTools
  - Seurat
  - SoloTE
  - scTE
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# transposable-element-expression-matrix-analysis

## Summary

Generate gene + transposable element (TE) expression matrices from single-cell RNA-Seq data at locus-specific resolution using SoloTE, and benchmark family-level summarization (scTE) against locus-level quantification for marker recovery and log2(FC) accuracy.

## When to use

Use when scRNA-Seq BAM files (CellRanger or STAR output) need TE expression quantified per genomic locus, or when comparing locus-specific vs family-metagene TE quantification for cell-type marker discovery.

## When NOT to use

- The input is already a gene/TE feature-count matrix — quantification is done; proceed directly to Seurat marker analysis
- The goal is to detect TEs acting as alternative transcription start sites — 3'-end 10X data cannot detect these due to 3' bias
- A non-TE repeat analysis (e.g. Satellites) is needed — note scTE indexes do not filter non-TE repeats, so family-metagene matrices may contain non-TE repeat rows that must be excluded

## Inputs

- BAM file from standard scRNA-Seq processing (CellRanger or STAR, ideally STAR run with --winAnchorMultimapNmax 100 --outFilterMultimapNmax 100 --outSAMmultNmax 1)
- TE annotation in BED format (columns: sequenceName, start, end, locus|Subfamily:Family:Class ID, strand, score), e.g. converted from UCSC RepeatMasker *.rm.out
- Cell cluster labels for marker analysis (Seurat object)

## Outputs

- Combined gene + TE expression matrix per cell, Seurat-compatible, with TE features named chr|start|end|Subfamily:Family:Class
- Cell marker TE table per cluster (Seurat FindAllMarkers, adjusted P-value <= 0.05, average log2(FC) >= 1)
- Per-cluster marker-recovery rates and log2(FC) deviations comparing SoloTE vs scTE (e.g. marker_recovery_log2fc_comparison.csv)

## How to apply

Run the SoloTE pipeline on the scRNA-Seq BAM file with a RepeatMasker-derived TE annotation in BED format (converted via SoloTE_RepeatMasker_to_BED.py for hg38 or mm10); SoloTE selects reads not assigned to genes, checks overlap with TE annotation using BEDTools, and summarizes TE expression at the locus level for reads with MAPQ >= threshold (default 255, uniquely mapped) and at family level for multi-mapping reads. Discard reads assigned to known genes by default (overridable with the 'dual' option) to avoid reporting TEs fully contained within genes as expressed. The output is a combined gene and TE expression matrix compatible with Seurat; for marker analysis run Seurat FindAllMarkers keeping only results with adjusted P-value <= 0.05 and average log2(Fold Change) >= 1. When benchmarking against scTE, note that scTE summarizes TE expression on family metagenes, losing genomic location and severely underestimating average log2(FC) marker values.

## Related tools

- **SoloTE** (Core pipeline: selects unassigned reads, overlaps with TE BED annotation, summarizes TE expression at locus level (MAPQ >= 255) and family level (multi-mappers), outputs Seurat-compatible matrix) — https://github.com/bvaldebenitom/SoloTE
- **scTE** (Benchmark comparator: summarizes TE expression on TE-family metagenes, losing genomic location; underestimates log2(FC) marker values)
- **Seurat** (Downstream marker analysis of the combined gene+TE matrix via FindAllMarkers with adjusted P-value <= 0.05 and average log2(FC) >= 1)
- **BEDTools** (Assesses read overlap with TE annotation (v2.29.2 or higher required by SoloTE)) — https://github.com/arq5x/bedtools2

## Examples

```
python SoloTE_pipeline.py --threads 8 --bam sample_aligned.bam --teannotation hg38_rm.bed --outputprefix sample --outputdir out_soloTE
```

## Evaluation signals

- Output matrix contains both gene and TE rows, with TE IDs in the chr|start|end|Subfamily:Family:Class format
- Marker table rows all satisfy adjusted P-value <= 0.05 and average log2(Fold Change) >= 1
- Recovered marker TEs match expected TE-cluster labels embedded in the simulation ground truth; SoloTE shows high marker recovery (except mm10 young TEs) while scTE has limited recovery
- scTE-predicted average log2(FC) values are markedly lower than ground-truth log2(FC) (underestimation expected for family-metagene summarization)
- TE rows should not include non-TE repeats such as Satellites (a scTE artifact)

## Limitations

- With 3'-end 10X short (100 nt) reads, not all TEs — especially very young ones — can be uniquely mapped to a genomic locus
- For organisms with a greater proportion of transpositionally-active TEs, most copies could be genetically intact and multi-mapping, challenging locus-level assignment
- scTE indexes do not filter non-TE repeats such as Satellites, which appear in the resulting matrix
- Events where the TE acts as an alternative transcription start site cannot be detected due to 3' bias; EM-algorithm performance on 3' end 10X data not yet assessed

## Evidence

- [abstract] SoloTE, a pipeline that outperforms the previous approach in terms of computational resources and by allowing the inclusion of locus-specific TE activity in scRNA-Seq expression matrixes: "SoloTE, a pipeline that outperforms the previous approach in terms of computational resources and by allowing the inclusion of locus-specific TE activity in scRNA-Seq expression matrixes"
- [intro] which summarizes TE expression on metagenes corresponding to TE families, thus, losing the genomic location: "which summarizes TE expression on metagenes corresponding to TE families, thus, losing the genomic location from which each TE is expressed"
- [discussion] only keeping the results having adjusted P-value ≤0.05 and average log2(Fold Change) ≥1: [evidence span withheld — anchor status: not_found]
- [discussion] scTE does not filter repetitive elements in such indexes: "scTE does not filter repetitive elements in such indexes, and thus, in the resulting matrix other types of non-TE repeats, such as Satellites, appear"
- [readme] python SoloTE_pipeline.py --threads NumberOfThreads --bam BAMfile --teannotation BEDfile --outputprefix Prefix --outputdir OutputDirectory: "python SoloTE_pipeline.py --threads NumberOfThreads --bam BAMfile --teannotation BEDfile --outputprefix Prefix --outputdir OutputDirectory"
