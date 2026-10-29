---
name: multimapping-read-family-level-aggregation
description: Use when building a locus-specific TE expression matrix from scRNA-Seq
  BAMs (CellRanger/STAR) and reads overlap multiple TE copies, so locus-level counts
  would be unreliable for multi-mappers.
when_to_use_negative:
- The input is already a gene/TE feature count matrix — no read-level MAPQ partitioning
  is needed.
- The goal is bulk RNA-Seq TE analysis with EM-based multi-mapper resolution (e.g.,
  Telescope/SQuIRE style), not single-cell locus-level summarization.
- The analysis requires TE-derived isoform detection or reads from TE-derived alternative
  transcription start sites, which SoloTE's 3'-biased 10X workflow cannot detect.
tools:
- name: SoloTE
  role: Pipeline implementing the MAPQ-based locus-level vs family-level read assignment
    for TE expression
  repo: https://github.com/bvaldebenitom/SoloTE
- name: BEDTools
  role: Assesses overlap of gene-unassigned reads with the TE annotation prior to
    MAPQ partitioning (v2.29.2 or higher required)
  repo: https://github.com/arq5x/bedtools2/releases
- name: samtools
  role: Required dependency (v1.16 or higher) for BAM handling in the pipeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:5dc1284d5777bef10fb4d027b79130d10f7ffb937e43cbb12095b9f355b9d80b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - BEDTools
  - SoloTE
  - samtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# multimapping-read-family-level-aggregation

## Summary

A read-assignment strategy from SoloTE that assigns uniquely mapped reads (MAPQ >= user-defined threshold, default 255) to individual TE loci, while multi-mapping reads — which cannot be placed at a single genomic copy — are aggregated at the TE family level to preserve repetitive-element signal without false locus calls.

## When to use

Use when building a locus-specific TE expression matrix from scRNA-Seq BAMs (CellRanger/STAR) and reads overlap multiple TE copies, so locus-level counts would be unreliable for multi-mappers.

## When NOT to use

- The input is already a gene/TE feature count matrix — no read-level MAPQ partitioning is needed.
- The goal is bulk RNA-Seq TE analysis with EM-based multi-mapper resolution (e.g., Telescope/SQuIRE style), not single-cell locus-level summarization.
- The analysis requires TE-derived isoform detection or reads from TE-derived alternative transcription start sites, which SoloTE's 3'-biased 10X workflow cannot detect.

## Inputs

- BAM file from CellRanger or STAR scRNA-Seq alignment (containing gene-assigned and unassigned reads)
- TE annotation in BED format with ID column chr|start|end|TE_Subfamily:TE_Family:TE_Class, strand, and score
- User-defined MAPQ threshold (default 255)

## Outputs

- Combined gene + TE expression matrix per cell, with locus-level rows for uniquely mapped reads and family-level rows for multi-mapping reads
- TE features encoded as Subfamily:Family:Class locus identifiers compatible with Seurat

## How to apply

After selecting reads not assigned to known genes and intersecting them with a BED-format TE annotation via BEDtools, partition alignments by MAPQ: alignments with MAPQ >= the user-defined threshold (default 255, i.e. uniquely mapped reads) are summarized at the locus level, with each TE feature ID encoding chr|start|end|TE_Subfamily:TE_Family:TE_Class. Alignments falling below the threshold are multi-mappers and cannot be confidently placed at one genomic copy, so their counts are aggregated at the TE family level instead. This dual-granularity scheme is the rationale for SoloTE's locus-specific output: unique reads give genomic location, while family-level pooling retains multi-mapping signal. The resulting per-cell matrix combines gene and TE features and is directly usable in downstream tools such as Seurat.

## Related tools

- **SoloTE** (Pipeline implementing the MAPQ-based locus-level vs family-level read assignment for TE expression) — https://github.com/bvaldebenitom/SoloTE
- **BEDTools** (Assesses overlap of gene-unassigned reads with the TE annotation prior to MAPQ partitioning (v2.29.2 or higher required)) — https://github.com/arq5x/bedtools2/releases
- **samtools** (Required dependency (v1.16 or higher) for BAM handling in the pipeline)

## Examples

```
python SoloTE_pipeline.py --threads 8 --bam sample.bam --teannotation hg38_TE.bed --outputprefix sample --outputdir out
```

## Evaluation signals

- With the default MAPQ threshold 255, locus-level TE rows should contain counts only from uniquely mapped alignments; verify no multi-mapping read contributed to a locus-level feature.
- Family-level aggregation rows should be keyed on TE_Family identifiers, consistent with the chr|start|end|Subfamily:Family:Class ID scheme in the annotation BED.
- The output matrix should contain both gene and TE features per cell and load into Seurat without errors.
- Reads assigned to known genes (unless the 'dual' option is set) must be absent from TE counts, so TEs fully contained within genes are not falsely reported as expressed.
- Running with a lower MAPQ threshold should shift more alignments from family-level to locus-level rows, an expected monotonic behavior of the partitioning.

## Limitations

- In organisms with a greater proportion of transpositionally-active TEs, most copies may be genetically intact and multi-mapping, so most signal falls to the family level and locus-specific resolution is lost.
- With short 100 nt 3'-end 10X reads, not all TEs can be uniquely mappable, limiting locus-level assignment.
- Multi-mapping reads at the family level lose the genomic location from which each TE copy is expressed.

## Evidence

- [discussion] MAPQ threshold partition: "a MAPQ equal to or greater than a user-defined threshold (default = 255, only uniquely mapped reads)"
- [discussion] Locus-level vs family-level summarization rule: "TE expression is summarized at the locus-level if the alignments have a MAPQ equal to or greater than a user-defined threshold"
- [discussion] Dual option preserves gene-assigned reads: [evidence span withheld — anchor status: not_found]
- [readme] Family/Class ID encoding in annotation: "column 4, the ID, is a concatenation of the locus of the TE and its identifiers at the Subfamily, Family and Class level"
- [discussion] Combined matrix output compatible with Seurat: "a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools, such as Seurat"
