---
name: locus-specific-te-expression-summarization
description: Use when processing single-cell RNA-Seq BAM files (CellRanger or STAR
  output) to detect locus-specific transposable element activity that standard pipelines
  discard, and to merge TE counts into a gene+TE expression matrix for downstream
  tools like Seurat.
when_to_use_negative:
- Input is already a processed expression/feature table rather than an unprocessed
  CellRanger/STAR BAM file
- The goal is family-level TE metagene summarization only (e.g. scTE-style analysis)
  without locus resolution
- Reads assigned to genes should also be counted against overlapping TEs and the 'dual'
  behavior is not desired as the default discard rule
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: SoloTE
  role: Pipeline that selects unassigned reads, intersects them with TE annotation,
    and summarizes locus/family-level TE expression into a Seurat-compatible matrix
  repo: https://github.com/bvaldebenitom/SoloTE
- name: BEDTools
  role: Assesses overlap of gene-unassigned reads with the TE annotation (v2.29.2
    or higher required)
  repo: https://github.com/arq5x/bedtools2/releases
- name: samtools
  role: Required dependency (v1.16 or higher) available in PATH for BAM handling
- name: pysam
  role: Python module required alongside Pandas (>=1.5.0) for the SoloTE pipeline
    scripts
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:50ab10d5aa807370d379d8c4d2b1aee637c7e28462caf0848d34f06e6b5ca7e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - BEDTools
  - Samtools
  - SoloTE
  - edgeR
  - pysam
  - samtools
  - scTE
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# locus-specific-te-expression-summarization

## Summary

Summarize transposable element (TE) expression at individual genomic loci in scRNA-Seq data using SoloTE, which overlaps unassigned reads from CellRanger/STAR BAMs against a TE BED annotation and emits a combined gene+TE expression matrix compatible with Seurat.

## When to use

Use when processing single-cell RNA-Seq BAM files (CellRanger or STAR output) to detect locus-specific transposable element activity that standard pipelines discard, and to merge TE counts into a gene+TE expression matrix for downstream tools like Seurat.

## When NOT to use

- Input is already a processed expression/feature table rather than an unprocessed CellRanger/STAR BAM file
- The goal is family-level TE metagene summarization only (e.g. scTE-style analysis) without locus resolution
- Reads assigned to genes should also be counted against overlapping TEs and the 'dual' behavior is not desired as the default discard rule

## Inputs

- BAM file from CellRanger or STAR scRNA-Seq alignment (with STAR options --winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, --outSAMmultNmax 1)
- TE annotation in SoloTE BED format (chr, start, end, ID=chr|start|end|Subfamily:Family:Class, strand, score, placeholder)
- UCSC RepeatMasker *rm.out file for the relevant genome (e.g. hg38, mm10), converted via the bundled helper

## Outputs

- Combined gene + TE expression matrix per cell, compatible with Seurat, written under the supplied outputprefix/outputdir
- TE features encoded as Subfamily:Family:Class locus identifiers alongside gene features

## How to apply

First convert a UCSC RepeatMasker (*rm.out) file to SoloTE's BED format using SoloTE_RepeatMasker_to_BED.py, where column 4 concatenates chr|start|end|TE_Subfamily:TE_Family:TE_Class, followed by strand and score. Then run SoloTE_pipeline.py, which selects reads not assigned to known genes (to avoid falsely reporting TEs fully contained within genes as expressed), assesses overlap of the remaining reads with the TE annotation using BEDTools, and summarizes TE expression at the locus level for alignments with MAPQ >= the user-defined threshold (default 255, i.e. uniquely mapped reads), aggregating multi-mapping reads at the TE family level. The output is a new matrix having both genes and TEs expression per cell, compatible with downstream analysis tools such as Seurat. When aligning reads upstream, use STAR with the recommended multi-mapping options --winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, and --outSAMmultNmax 1 so multi-mapping reads are retained for family-level summarization.

## Related tools

- **SoloTE** (Pipeline that selects unassigned reads, intersects them with TE annotation, and summarizes locus/family-level TE expression into a Seurat-compatible matrix) — https://github.com/bvaldebenitom/SoloTE
- **BEDTools** (Assesses overlap of gene-unassigned reads with the TE annotation (v2.29.2 or higher required)) — https://github.com/arq5x/bedtools2/releases
- **samtools** (Required dependency (v1.16 or higher) available in PATH for BAM handling)
- **pysam** (Python module required alongside Pandas (>=1.5.0) for the SoloTE pipeline scripts)

## Examples

```
python SoloTE_pipeline.py --threads 8 --bam sample.bam --teannotation mm10_te.bed --outputprefix sample --outputdir SoloTE_out
```

## Evaluation signals

- The output directory is created (if absent) and contains the combined gene+TE expression matrix under the supplied outputprefix
- TE features in the matrix are encoded as chr|start|end|Subfamily:Family:Class locus identifiers alongside gene features per cell
- Gene-assigned reads are absent from TE counts by default, so TEs fully contained within genes are not falsely reported as expressed
- Only alignments with MAPQ >= 255 (uniquely mapped) contribute locus-level counts; multi-mapping reads appear aggregated at family level
- The BED annotation passed via --teannotation follows the required 8-column format with the concatenated ID in column 4

## Limitations

- For organisms with a greater proportion of transpositionally-active TEs, most copies could be genetically intact and multi-mapping, so locus-level assignment (MAPQ >= 255) may capture little signal
- Not all TEs can be uniquely mappable with short 100 nucleotide reads generated by 3' end 10X data
- Events in which the TE acts as an alternative transcription start site cannot be detected due to the 3' bias of 10X data
- TEs are discarded when assigned to genes by default (unless the 'dual' option is used), which may miss TEs genuinely co-transcribed with their host genes

## Evidence

- [discussion] Read selection and MAPQ-based summarization rationale: "The first step of SoloTE is the selection of reads not assigned to genes."
- [discussion] Gene-assigned read discard rule and dual override: "reads assigned to known genes are discarded, to avoid mistakenly reporting TEs fully contained within genes, as expressed."
- [discussion] MAPQ threshold definition: "a MAPQ equal to or greater than a user-defined threshold (default = 255, only uniquely mapped reads)"
- [discussion] Output matrix compatibility: "a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools, such as Seurat"
- [readme] BED format specification from README: "column 4, the ID, is a concatenation of the locus of the TE and its identifiers at the Subfamily, Family and Class level."
- [discussion] STAR multi-mapping alignment recommendation: [evidence span withheld — anchor status: not_found]
- [discussion] BEDTools overlap assessment: "the overlap with the TE annotation is assessed using BEDtools"
