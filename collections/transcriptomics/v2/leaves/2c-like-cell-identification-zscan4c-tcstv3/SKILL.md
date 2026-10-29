---
name: 2c-like-cell-identification-zscan4c-tcstv3
description: Use when analyzing a murine embryonic two-cell stage single-cell RNA-Seq
  dataset where a 2C-like subpopulation must be defined and its differentially expressed
  transposable elements identified at locus level.
when_to_use_negative:
- Input is bulk RNA-Seq or a pre-computed TE expression matrix — SoloTE requires single-cell
  BAM files with multi-mapped reads retained
- Analysis targets family-level TE expression only (e.g. scTE metagenes) and does
  not need locus-specific genomic location
- The cell type of interest is not a transcriptionally active-TE population (e.g.
  datasets without known 2C-marker TE activity); STAR multi-mapping alignment overhead
  may not be justified
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: STAR
  role: Aligns two-cell stage FASTQ reads to mm10 with multi-mapping options (--winAnchorMultimapNmax
    100, --outFilterMultimapNmax 100, --outSAMmultNmax 1) so repetitive TE reads are
    not lost
- name: SoloTE
  role: Generates the locus-specific TE expression matrix from the STAR BAM using
    the mm10 RepeatMasker BED annotation; outputs a Seurat-compatible combined gene+TE
    matrix
  repo: https://github.com/bvaldebenitom/SoloTE
- name: BEDTools
  role: Assesses overlap of unassigned reads with the TE BED annotation during SoloTE
    processing (v2.29.2 or higher required)
  repo: https://github.com/arq5x/bedtools2
- name: Seurat
  role: Normalizes and clusters the combined gene+TE matrix; 2C-like cells defined
    by Zscan4c/Tcstv3; FindAllMarkers reports marker TEs with adjusted P ≤0.05 and
    average log2FC ≥1
- name: scTE
  role: Benchmark tool summarizing TE expression at family level; recovered 28 marker
    TEs vs SoloTE's 106
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:df59ed4792a95dbe5bc058e20dbd49ebff4a05a0e95d4e2b1c36fe6e9046c208
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - BEDTools
  - STAR
  - Seurat
  - SoloTE
  - scTE
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# 2c-like-cell-identification-zscan4c-tcstv3

## Summary

Identifies 2C-like cells in the murine embryonic two-cell stage scRNA-Seq data by Zscan4c/Tcstv3 expression, then recovers differentially expressed marker transposable elements at locus level with SoloTE (106 marker TEs vs 28 with scTE under adjusted P ≤0.05, log2FC ≥1).

## When to use

Use when analyzing a murine embryonic two-cell stage single-cell RNA-Seq dataset where a 2C-like subpopulation must be defined and its differentially expressed transposable elements identified at locus level.

## When NOT to use

- Input is bulk RNA-Seq or a pre-computed TE expression matrix — SoloTE requires single-cell BAM files with multi-mapped reads retained
- Analysis targets family-level TE expression only (e.g. scTE metagenes) and does not need locus-specific genomic location
- The cell type of interest is not a transcriptionally active-TE population (e.g. datasets without known 2C-marker TE activity); STAR multi-mapping alignment overhead may not be justified

## Inputs

- Murine two-cell stage scRNA-Seq FASTQ files from Gene Expression Omnibus (Supplementary Table 3 accessions)
- mm10 RepeatMasker annotation (*.rm.out from UCSC, converted to SoloTE BED format)
- STAR-aligned BAM file with multi-mapping options enabled
- Seurat normalized gene+TE expression matrix with 2C-like vs non-2C-like cell identity assignments

## Outputs

- Combined gene and locus-specific TE expression matrix compatible with Seurat
- Table of 106 marker TEs differentially expressed in 2C-like cells (adjusted P ≤0.05, average log2FC ≥1), e.g. solute_2c_marker_TEs.csv
- Comparison count against scTE (28 marker TEs at family/metagene level)

## How to apply

Download the murine two-cell stage scRNA-Seq FASTQ files from GEO (accessions in Supplementary Table 3) and align to mm10 with STAR using multi-mapping options --winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, and --outSAMmultNmax 1 to retain multi-mapping TE reads. Convert the mm10 RepeatMasker annotation to BED with SoloTE_RepeatMasker_to_BED.py (-g mm10), then run SoloTE_pipeline.py on the STAR BAM (--teannotation BED; default MAPQ 255 assigns locus-level TE expression for uniquely mapped reads and family-level for multi-mapped reads), discarding gene-assigned reads by default. Load the combined gene+TE matrix into Seurat, normalize, cluster, and define 2C-like cells by Zscan4c/Tcstv3 expression. Run Seurat FindAllMarkers keeping only results with adjusted P-value ≤0.05 and average log2(Fold Change) ≥1, and confirm the expected MERVL-int and MT2-Mm 2C-marker expression pattern.

## Related tools

- **STAR** (Aligns two-cell stage FASTQ reads to mm10 with multi-mapping options (--winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, --outSAMmultNmax 1) so repetitive TE reads are not lost)
- **SoloTE** (Generates the locus-specific TE expression matrix from the STAR BAM using the mm10 RepeatMasker BED annotation; outputs a Seurat-compatible combined gene+TE matrix) — https://github.com/bvaldebenitom/SoloTE
- **BEDTools** (Assesses overlap of unassigned reads with the TE BED annotation during SoloTE processing (v2.29.2 or higher required)) — https://github.com/arq5x/bedtools2
- **Seurat** (Normalizes and clusters the combined gene+TE matrix; 2C-like cells defined by Zscan4c/Tcstv3; FindAllMarkers reports marker TEs with adjusted P ≤0.05 and average log2FC ≥1)
- **scTE** (Benchmark tool summarizing TE expression at family level; recovered 28 marker TEs vs SoloTE's 106)

## Examples

```
python SoloTE_pipeline.py --threads 8 --bam twocell_alignedAligned.sortedByCoord.out.bam --teannotation mm10_repeatmasker.bed --outputprefix solote_2c --outputdir solote_out
```

## Evaluation signals

- Exactly 106 marker TEs recovered by SoloTE in 2C-like cells at adjusted P ≤0.05 and average log2FC ≥1 (vs 28 with scTE)
- MERVL-int and MT2-Mm show the expected 2C-marker expression pattern restricted to the Zscan4c/Tcstv3-positive population
- No TEs called as markers in non-2C-like cells under the log2FC ≥1 threshold
- SoloTE output matrix contains both genes and TEs per cell and loads directly into Seurat without format conversion
- Locus-level TE assignments use the default MAPQ ≥255 threshold (uniquely mapped reads only); multi-mapped reads summarize at family level

## Limitations

- Locus-level assignment is limited to uniquely mapped reads (MAPQ 255 default); in organisms with a greater proportion of transpositionally-active TEs, most copies could be genetically intact and multi-mapping, degrading locus resolution
- Not all TEs can be uniquely mappable with short 100 nucleotide reads, especially in 3' end 10X data
- SoloTE discards gene-assigned reads by default, so TEs fully contained within genes may be missed unless the 'dual' option is used
- scTE indexes do not filter non-TE repeats such as Satellites, complicating direct tool comparison

## Evidence

- [discussion] Seurat FindAllMarkers with adjusted P and log2FC cutoffs: [evidence span withheld — anchor status: not_found]
- [intro] Murine two-cell stage TE expression in a subset of cells: "First, we studied the murine embryonic two-cell stage, in which TE expression is known to occur in a subset of cells14"
- [discussion] STAR multi-mapping recommendation: [evidence span withheld — anchor status: not_found]
- [discussion] SoloTE MAPQ-based locus/family summarization: "TE expression is summarized at the locus-level if the alignments have a MAPQ equal to or greater than a user-defined threshold"
- [readme] SoloTE pipeline CLI from README: "python SoloTE_pipeline.py --threads NumberOfThreads --bam BAMfile --teannotation BEDfile --outputprefix Prefix --outputdir OutputDirectory"
