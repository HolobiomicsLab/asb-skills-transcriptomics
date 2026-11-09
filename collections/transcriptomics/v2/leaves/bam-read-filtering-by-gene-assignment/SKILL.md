---
name: bam-read-filtering-by-gene-assignment
description: Use when starting from a CellRanger or STAR scRNA-Seq BAM and you need
  unassigned (non-gene) reads to quantify locus-specific transposable element expression
  without gene-contamination artifacts.
when_to_use_negative:
- You want reads assigned to genes included in TE quantification and have not decided
  to use the 'dual' option — default behavior discards them.
- Input is raw FASTQ rather than an aligned BAM from CellRanger or STAR; alignment
  must be done first (STAR with --winAnchorMultimapNmax 100, --outFilterMultimapNmax
  100, --outSAMmultNmax 1 is recommended).
- Analysis goal is gene-level expression only; the gene-assignment filter is specific
  to isolating non-gene-assigned reads for TE quantification.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: SoloTE
  role: Pipeline implementing the gene-assignment read selection step and downstream
    TE summarization
  repo: https://github.com/bvaldebenitom/SoloTE
- name: BEDTools
  role: Assesses overlap of the gene-unassigned reads with the TE annotation (v2.29.2
    or higher)
  repo: https://github.com/arq5x/bedtools2/releases
- name: samtools
  role: Required in PATH for BAM processing (v1.16 or higher)
- name: pysam
  role: Python module used by SoloTE for BAM handling
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:fd2f350a537ec889b20420f8fc130fc03f577e01be6a216e530fef4dbf1be302
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - BEDTools
  - SoloTE
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# BAM read filtering by gene assignment

## Summary

SoloTE's first pipeline step: select reads from a CellRanger/STAR scRNA-Seq BAM that are not assigned to known genes, so transposable elements fully contained within genes are not falsely reported as expressed; reads are then eligible for BEDTools overlap against a TE annotation.

## When to use

Use when starting from a CellRanger or STAR scRNA-Seq BAM and you need unassigned (non-gene) reads to quantify locus-specific transposable element expression without gene-contamination artifacts.

## When NOT to use

- You want reads assigned to genes included in TE quantification and have not decided to use the 'dual' option — default behavior discards them.
- Input is raw FASTQ rather than an aligned BAM from CellRanger or STAR; alignment must be done first (STAR with --winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, --outSAMmultNmax 1 is recommended).
- Analysis goal is gene-level expression only; the gene-assignment filter is specific to isolating non-gene-assigned reads for TE quantification.

## Inputs

- BAM file from CellRanger or STAR scRNA-Seq alignment
- TE annotation in BED format with ID column chr|start|end|TE_Subfamily:TE_Family:TE_Class|strand
- User-defined MAPQ threshold (default 255)

## Outputs

- Filtered set of reads not assigned to genes, overlapped with the TE annotation
- Combined gene + TE expression matrix per cell, compatible with Seurat

## How to apply

Launch SoloTE_pipeline.py on the BAM produced by CellRanger or STAR; the first step selects reads not assigned to known genes and discards gene-assigned reads, which prevents TEs fully contained within genes from being mistakenly reported as expressed. Optionally pass the 'dual' option to retain gene-assigned reads when dual gene/TE assignment is desired. After filtering, assess overlap of the remaining reads with the TE annotation (BED format from SoloTE_RepeatMasker_to_BED.py or a UCSC *rm.out conversion) using BEDTools, then summarize expression at the locus level only for alignments with MAPQ ≥ the user-defined threshold (default 255, i.e., uniquely mapped reads) and aggregate multi-mapping reads at the TE family level. Inputs to the launcher are --threads, --bam, --teannotation, --outputprefix, and --outputdir.

## Related tools

- **SoloTE** (Pipeline implementing the gene-assignment read selection step and downstream TE summarization) — https://github.com/bvaldebenitom/SoloTE
- **BEDTools** (Assesses overlap of the gene-unassigned reads with the TE annotation (v2.29.2 or higher)) — https://github.com/arq5x/bedtools2/releases
- **samtools** (Required in PATH for BAM processing (v1.16 or higher))
- **pysam** (Python module used by SoloTE for BAM handling)

## Examples

```
python SoloTE_pipeline.py --threads 8 --bam sample_aligned.bam --teannotation hg38_rmsk.bed --outputprefix sample --outputdir results/
```

## Evaluation signals

- Output directory (--outputdir) is created if absent and contains the combined gene+TE expression matrix under the supplied --outputprefix.
- TE features in the matrix carry Subfamily:Family:Class locus identifiers (e.g., chr1|11505|11675|L1MC5a:L1:LINE), confirming locus-level assignment for MAPQ ≥ threshold reads.
- Gene-assigned reads are absent from the TE quantification unless the 'dual' option was used — TEs fully contained within genes should not appear as expressed.
- Multi-mapping reads (below the MAPQ threshold, default 255) are aggregated at the TE family level rather than the locus level.

## Limitations

- Default discarding of gene-assigned reads may miss TEs actively transcribed within genes unless the 'dual' option is used.
- With 3' end 10X short ~100 nt reads, not all TEs can be uniquely mappable, so locus-level assignment is limited; EM-algorithm handling of multi-mapping reads on 3' end 10X data has not been assessed.
- In organisms with a greater proportion of transpositionally-active TEs, most copies could be genetically intact and multi-mapping, reducing locus-specific resolution.

## Evidence

- [discussion] The first step of SoloTE is the selection of reads not assigned to genes.: "The first step of SoloTE is the selection of reads not assigned to genes."
- [discussion] Gene-assigned reads discarded by default; dual option: [evidence span withheld — anchor status: not_found]
- [discussion] MAPQ locus-level threshold: "a MAPQ equal to or greater than a user-defined threshold (default = 255, only uniquely mapped reads)"
- [readme] README invocation of the pipeline: "python SoloTE_pipeline.py --threads NumberOfThreads --bam BAMfile --teannotation BEDfile --outputprefix Prefix --outputdir OutputDirectory"
- [discussion] SoloTE starts from CellRanger/STAR BAM: "SoloTE can start directly from the BAM file obtained with either CellRanger or STAR"
