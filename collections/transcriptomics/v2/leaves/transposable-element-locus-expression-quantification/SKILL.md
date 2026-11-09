---
name: transposable-element-locus-expression-quantification
description: Use when you need locus-specific TE expression quantification from scRNA-Seq
  BAM files (STAR or CellRanger) to identify TE markers of cell populations, especially
  repetitive-element activity in subpopulations like 2C-like cells or cancer clusters.
when_to_use_negative:
- The analysis already provides a combined gene+TE feature matrix and only downstream
  clustering/markers are needed — do not re-quantify
- The biology requires detecting TEs acting as alternative transcription start sites
  in 3'-end 10X data — the 3' bias prevents detection of such events
- The genome has a large proportion of transpositionally-active, genetically intact
  TE copies — most reads will multi-map and locus-level resolution will collapse to
  family-level summaries
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0204
tools:
- name: SoloTE
  role: 'Core pipeline: selects gene-unassigned reads, overlaps with TE BED annotation
    via BEDTools, and outputs a combined gene+TE expression matrix at locus level'
  repo: https://github.com/bvaldebenitom/SoloTE
- name: SoloTE_RepeatMasker_to_BED.py
  role: Helper utility bundled with SoloTE to download RepeatMasker files from UCSC
    and convert them to the required BED format
  repo: https://github.com/bvaldebenitom/SoloTE
- name: STAR
  role: Alignment with multi-mapping-permissive options to retain reads mapping to
    repetitive TE loci
- name: BEDTools
  role: Assesses overlap of gene-unassigned reads with the TE annotation during TE
    expression summarization
  repo: https://github.com/arq5x/bedtools2/releases
- name: Seurat
  role: Downstream normalization, clustering, UMAP and FindAllMarkers on the combined
    gene+TE matrix to identify marker TEs per cell cluster
- name: scTE
  role: Family-level comparator tool used to benchmark SoloTE's locus-resolution advantage
    in marker TE recovery
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:ca08547eb815415650b9d2f6c6c71c8186f5e15d9330ccad9f2dd9a81de5b65b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - BEDTools
  - Pandas
  - Pysam
  - R
  - STAR
  - Samtools
  - Seurat
  - SoloTE
  - SoloTE_RepeatMasker_to_BED.py
  - scTE
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# transposable-element-locus-expression-quantification

## Summary

Quantifies transposable element (TE) expression at individual genomic locus resolution in scRNA-Seq data using the SoloTE pipeline, producing a combined gene+TE expression matrix compatible with Seurat for downstream cell-type marker analysis. Retains genomic location that family-level tools like scTE discard.

## When to use

Use when you need locus-specific TE expression quantification from scRNA-Seq BAM files (STAR or CellRanger) to identify TE markers of cell populations, especially repetitive-element activity in subpopulations like 2C-like cells or cancer clusters.

## When NOT to use

- The analysis already provides a combined gene+TE feature matrix and only downstream clustering/markers are needed — do not re-quantify
- The biology requires detecting TEs acting as alternative transcription start sites in 3'-end 10X data — the 3' bias prevents detection of such events
- The genome has a large proportion of transpositionally-active, genetically intact TE copies — most reads will multi-map and locus-level resolution will collapse to family-level summaries

## Inputs

- STAR or CellRanger BAM file from scRNA-Seq processing
- RepeatMasker annotation (.rm.out from UCSC) or TE BED file in format: chr, start, end, ID (locus|TE_Subfamily:TE_Family:TE_Class), strand, optional score
- Genome version identifier (e.g. hg38, mm10) for annotation conversion

## Outputs

- Combined gene and TE expression matrix per cell, compatible with Seurat
- Locus-level TE expression counts (MAPQ >= 255 reads) and family-level counts (multi-mappers)
- Marker TE list (e.g. solote_2c_marker_TEs.csv) with adjusted P-value <= 0.05 and average log2(FC) >= 1, resolved to genomic loci

## How to apply

First convert the RepeatMasker annotation to BED format with SoloTE_RepeatMasker_to_BED.py (e.g. -g mm10 or -g hg38), ensuring column 4 is the locus ID concatenating TE_Subfamily:TE_Family:TE_Class. Align reads with STAR using multi-mapping-permissive options (--winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, --outSAMmultNmax 1), then run SoloTE_pipeline.py, which selects reads not assigned to genes, assesses overlap with the TE annotation using BEDTools, and summarizes expression at locus level for MAPQ >= 255 (uniquely mapped) reads and at family level for multi-mappers. Load the resulting combined gene+TE matrix into Seurat, normalize, cluster (e.g. 6 clusters in the early gastric cancer dataset), and run FindAllMarkers keeping only adjusted P-value <= 0.05 and average log2(Fold Change) >= 1. Locus resolution allows pinpointing specific TE copies, e.g. L1PA7 at chrY:18989629-18990627 as a Cancer-cluster marker. Validate by comparing marker recovery against scTE (SoloTE: 106 murine 2C marker TEs vs 28; 24 GC marker TEs vs 3).

## Related tools

- **SoloTE** (Core pipeline: selects gene-unassigned reads, overlaps with TE BED annotation via BEDTools, and outputs a combined gene+TE expression matrix at locus level) — https://github.com/bvaldebenitom/SoloTE
- **SoloTE_RepeatMasker_to_BED.py** (Helper utility bundled with SoloTE to download RepeatMasker files from UCSC and convert them to the required BED format) — https://github.com/bvaldebenitom/SoloTE
- **STAR** (Alignment with multi-mapping-permissive options to retain reads mapping to repetitive TE loci)
- **BEDTools** (Assesses overlap of gene-unassigned reads with the TE annotation during TE expression summarization) — https://github.com/arq5x/bedtools2/releases
- **Seurat** (Downstream normalization, clustering, UMAP and FindAllMarkers on the combined gene+TE matrix to identify marker TEs per cell cluster)
- **scTE** (Family-level comparator tool used to benchmark SoloTE's locus-resolution advantage in marker TE recovery)

## Examples

```
python SoloTE_pipeline.py --threads 8 --bam aligned.bam --teannotation mm10.bed --outputprefix sample --outputdir solote_out
```

## Evaluation signals

- The TE BED file column 4 follows the ID format locus|TE_Subfamily:TE_Family:TE_Class (e.g. chr1|11505|11675|L1MC5a:L1:LINE|25.1|-)
- On the murine two-cell stage dataset, SoloTE recovers 106 marker TEs in Zscan4c/Tcstv3-defined 2C-like cells at adjusted P <= 0.05 and log2(FC) >= 1 (vs 28 with scTE), with MERVL-int and MT2-Mm showing the expected 2C-marker pattern
- On the early gastric cancer dataset, the output reproduces 6 cell clusters and 24 marker TEs, including L1PA7 at chrY:18989629-18990627 and THE1D at chr8:61579930-61580458
- Locus-level assignment uses only MAPQ >= 255 (uniquely mapped) alignments; multi-mappers are summarized at family level only
- Reads assigned to known genes are discarded by default, so fully gene-contained TEs are not falsely reported as expressed (unless the 'dual' option is set)

## Limitations

- For organisms with a greater proportion of transpositionally-active TEs, most copies could be genetically intact and multi-mapping, so locus-level resolution degrades to family-level summarization
- Not all TEs can be uniquely mappable with the short 100 nucleotide reads generated by 3'-end 10X protocols; EM-based resolution has not been assessed on such data
- TEs acting as alternative transcription start sites cannot be detected in 3'-end 10X data due to the 3' bias
- scTE comparison indexes do not filter non-TE repeats such as Satellites, which can contaminate family-level TE matrices

## Evidence

- [discussion] Locus-level for uniquely mapped reads and family level for multi-mapping reads: "TE expression is summarized at the locus-level if the alignments have a MAPQ equal to or greater than a user-defined threshold"
- [discussion] Combined gene+TE matrix output: "a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools, such as Seurat"
- [discussion] Marker filtering thresholds: [evidence span withheld — anchor status: not_found]
- [readme] ReadMastker-to-BED conversion utility: "The helper utility, `SoloTE_RepeatMasker_to_BED.py`, is packed with SoloTE, and it streamlines the download of RepeatMasker file from UCSC, and conversion to BED format."
- [readme] BED ID encodes locus and TE taxonomy: "column 4, the ID, is a concatenation of the locus of the TE and its identifiers at the Subfamily, Family and Class level."
