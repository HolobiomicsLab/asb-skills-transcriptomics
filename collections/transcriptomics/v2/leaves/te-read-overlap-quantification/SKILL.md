---
name: te-read-overlap-quantification
description: Use when starting from CellRanger- or STAR-processed scRNA-Seq BAM files
  and needing to add locus-specific TE expression (not just family-level metagenes)
  to a gene-by-cell expression matrix for downstream tools like Seurat.
when_to_use_negative:
- Input is already a gene+TE feature expression matrix — this skill operates on raw
  BAM alignments, not on quantified tables.
- The goal is detecting TEs acting as alternative transcription start sites in 3'-end
  10X data — 3' bias means such events cannot be detected.
- Only family-level TE metagene expression is needed and genomic location should be
  discarded — a family-summarizing tool (e.g., scTE-style) is the wrong complement
  here; this skill preserves locus identity.
edam_operation: http://edamontology.org/operation_0315
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SoloTE
  role: Pipeline orchestrating gene-read filtering, BEDTools overlap, and locus/family-level
    TE summarization from the input BAM
  repo: github.com/bvaldebenitom/SoloTE
- name: BEDTools
  role: Assesses overlap of non-gene-assigned reads with the TE annotation BED file
  repo: https://github.com/arq5x/bedtools2
- name: samtools
  role: BAM handling dependency required in PATH (v1.16 or higher)
- name: pysam
  role: Python module used by SoloTE to read BAM alignments
- name: pandas
  role: Python module (>=1.5.0) used by SoloTE for expression matrix assembly
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:ae533afa7f20670b3bbd068234c043d9f4a0a2d52f71ab5db353695af658ade2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BEDTools
  - SoloTE
  - pandas
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_0315
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# te-read-overlap-quantification

## Summary

Quantifies locus-specific transposable element (TE) expression from single-cell RNA-Seq BAM files by selecting reads not assigned to genes, intersecting them with a BED-formatted TE annotation via BEDTools, and summarizing expression at the TE locus level (MAPQ ≥ threshold, default 255) or family level for multi-mappers.

## When to use

Use when starting from CellRanger- or STAR-processed scRNA-Seq BAM files and needing to add locus-specific TE expression (not just family-level metagenes) to a gene-by-cell expression matrix for downstream tools like Seurat.

## When NOT to use

- Input is already a gene+TE feature expression matrix — this skill operates on raw BAM alignments, not on quantified tables.
- The goal is detecting TEs acting as alternative transcription start sites in 3'-end 10X data — 3' bias means such events cannot be detected.
- Only family-level TE metagene expression is needed and genomic location should be discarded — a family-summarizing tool (e.g., scTE-style) is the wrong complement here; this skill preserves locus identity.

## Inputs

- BAM file from CellRanger or STAR single-cell RNA-Seq processing
- TE annotation in BED format (chr, start, end, chr|start|end|TE_Subfamily:TE_Family:TE_Class|strand, score, placeholder)
- UCSC RepeatMasker *rm.out file (converted via SoloTE_RepeatMasker_to_BED.py)
- User-defined MAPQ threshold (default 255)

## Outputs

- Combined gene and TE expression matrix per cell, with TE features encoded as Subfamily:Family:Class locus identifiers
- Matrix compatible with downstream tools such as Seurat, written under the given outputprefix/outputdir

## How to apply

First convert a UCSC RepeatMasker (*rm.out) annotation to BED using the bundled helper so column 4 concatenates chr|start|end|TE_Subfamily:TE_Family:TE_Class, strand, and score. Run the pipeline on the CellRanger/STAR BAM: it selects reads not assigned to known genes (discarding gene-assigned reads by default, to avoid falsely reporting TEs fully contained within genes as expressed; the 'dual' option overrides this), then assesses overlap of the remaining reads with the TE annotation using BEDTools. TE expression is summarized at the locus level when alignments have MAPQ equal to or greater than a user-defined threshold (default 255, only uniquely mapped reads), while multi-mapping reads are aggregated at the TE family level. The result is a new matrix with both genes and TEs expression per cell, compatible with downstream analysis tools such as Seurat.

## Related tools

- **SoloTE** (Pipeline orchestrating gene-read filtering, BEDTools overlap, and locus/family-level TE summarization from the input BAM) — github.com/bvaldebenitom/SoloTE
- **BEDTools** (Assesses overlap of non-gene-assigned reads with the TE annotation BED file) — https://github.com/arq5x/bedtools2
- **samtools** (BAM handling dependency required in PATH (v1.16 or higher))
- **pysam** (Python module used by SoloTE to read BAM alignments)
- **pandas** (Python module (>=1.5.0) used by SoloTE for expression matrix assembly)

## Examples

```
python SoloTE_pipeline.py --threads 8 --bam sample.bam --teannotation hg38_te.bed --outputprefix sample_te --outputdir out/
```

## Evaluation signals

- Output directory is created if absent and contains the combined gene+TE expression matrix under the supplied outputprefix.
- TE features in the matrix are encoded as Subfamily:Family:Class locus identifiers (e.g., L1MC5a:L1:LINE), not plain family names.
- Reads assigned to known genes do not contribute to TE counts unless the 'dual' option was set.
- Locus-level TE counts derive only from alignments with MAPQ ≥ the user threshold (default 255 = uniquely mapped reads); multi-mapping reads appear only aggregated at the family level.
- Matrix loads in Seurat as a gene+TE feature matrix per cell.

## Limitations

- Multi-mapping reads can only be summarized at the TE family level, losing locus identity.
- With default MAPQ = 255, only uniquely mapped reads get locus-level assignment; not all TEs are uniquely mappable with short ~100 nt 10X reads.
- 3'-end 10X bias means TEs acting as alternative transcription start sites cannot be detected.
- In organisms with a greater proportion of transpositionally-active TEs, most copies may be genetically intact and multi-mapping, limiting locus-specific resolution.
- Expectation–Maximization read assignment and TE-derived isoform analysis are not yet integrated.

## Evidence

- [discussion] The first step of SoloTE is the selection of reads not assigned to genes.: "The first step of SoloTE is the selection of reads not assigned to genes."
- [discussion] Overlap of remaining reads with TE annotation via BEDTools: "the overlap with the TE annotation is assessed using BEDtools"
- [discussion] Locus-level summarization is MAPQ-gated: "TE expression is summarized at the locus-level if the alignments have a MAPQ equal to or greater than a user-defined threshold"
- [discussion] Combined output matrix is Seurat-compatible: "a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools, such as Seurat"
- [discussion] Gene-assigned reads discarded by default; dual option: [evidence span withheld — anchor status: not_found]
- [readme] BED ID column concatenates locus and TE hierarchy: "column 4, the ID, is a concatenation of the locus of the TE and its identifiers at the Subfamily, Family and Class level."
- [readme] Pipeline CLI invocation from README: "python SoloTE_pipeline.py --threads NumberOfThreads --bam BAMfile --teannotation BEDfile --outputprefix Prefix --outputdir OutputDirectory"
