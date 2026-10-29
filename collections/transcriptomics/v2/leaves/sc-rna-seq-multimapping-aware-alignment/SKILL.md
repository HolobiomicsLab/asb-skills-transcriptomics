---
name: sc-rna-seq-multimapping-aware-alignment
description: Use when preprocessing scRNA-Seq data for transposable element expression
  analysis, where default gene-alignment pipelines discard multi-mapping reads originating
  from repetitive TEs.
when_to_use_negative:
- When the goal is gene-level expression only, since permissive multi-mapping is unnecessary
  and gene-aligning pipelines already discard TE reads
- When TEs are extremely young/intact in the organism so that most copies multi-map
  and cannot be uniquely resolved at locus level
- When using 3' end 10X data where not all TEs are uniquely mappable with 100 nt reads
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: STAR
  role: Spliced aligner run with multi-mapping-permissive options (--winAnchorMultimapNmax
    100, --outFilterMultimapNmax 100, --outSAMmultNmax 1) to retain TE-derived multi-mapping
    reads in the BAM
- name: SoloTE
  role: Consumes the multi-mapping-permissive BAM, selects reads not assigned to genes,
    and summarizes TE expression at locus level (MAPQ >= 255) or family level (multi-mappers)
  repo: https://github.com/bvaldebenitom/SoloTE
- name: BEDTools
  role: Assesses overlap of unassigned reads with the TE annotation (v2.29.2 or higher)
  repo: https://github.com/arq5x/bedtools2
- name: Samtools
  role: BAM handling dependency (v1.16 or higher) required by SoloTE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:4544e18a0dc5af10e5ce847c4d5e896792a57a78b3e2043fa6e45bb7facc81cb
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
  - SoloTE
  - scTE
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# sc-rna-seq-multimapping-aware-alignment

## Summary

Align single-cell RNA-Seq reads with STAR using multi-mapping-permissive parameters (--winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, --outSAMmultNmax 1) so that reads from repetitive transposable elements are retained in the BAM and can be assigned to TE loci by downstream tools such as SoloTE.

## When to use

Use when preprocessing scRNA-Seq data for transposable element expression analysis, where default gene-alignment pipelines discard multi-mapping reads originating from repetitive TEs.

## When NOT to use

- When the goal is gene-level expression only, since permissive multi-mapping is unnecessary and gene-aligning pipelines already discard TE reads
- When TEs are extremely young/intact in the organism so that most copies multi-map and cannot be uniquely resolved at locus level
- When using 3' end 10X data where not all TEs are uniquely mappable with 100 nt reads

## Inputs

- Single-cell RNA-Seq FASTQ reads (e.g., early gastric cancer dataset, accessions in Supplementary Table 3 from GEO)
- Reference genome (e.g., hg38)
- STAR aligner with multi-mapping-permissive parameters

## Outputs

- BAM file retaining multi-mapping TE-derived alignments (one alignment per read via --outSAMmultNmax 1)
- Downstream combined gene + TE expression matrix compatible with Seurat

## How to apply

Align the scRNA-Seq reads (e.g., the early gastric cancer dataset from GEO) with STAR using multi-mapping-permissive options: --winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, and --outSAMmultNmax 1, so repetitive reads are not discarded and at most one alignment per read is emitted to the BAM. The resulting BAM can feed either CellRanger- or STAR-based workflows; SoloTE can start directly from it. Downstream, SoloTE selects reads not assigned to genes, intersects them with a RepeatMasker-derived TE BED annotation using BEDTools, and summarizes TE expression at the locus level for alignments with MAPQ >= 255 (uniquely mapped) and at the family level for multi-mappers, producing a combined gene+TE matrix compatible with Seurat. Correct application is judged by whether multi-mapping repetitive reads survive alignment and can be resolved to loci such as L1PA7 (chrY:18989629-18990627).

## Related tools

- **STAR** (Spliced aligner run with multi-mapping-permissive options (--winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, --outSAMmultNmax 1) to retain TE-derived multi-mapping reads in the BAM)
- **SoloTE** (Consumes the multi-mapping-permissive BAM, selects reads not assigned to genes, and summarizes TE expression at locus level (MAPQ >= 255) or family level (multi-mappers)) — https://github.com/bvaldebenitom/SoloTE
- **BEDTools** (Assesses overlap of unassigned reads with the TE annotation (v2.29.2 or higher)) — https://github.com/arq5x/bedtools2
- **Samtools** (BAM handling dependency (v1.16 or higher) required by SoloTE)

## Examples

```
python SoloTE_pipeline.py --threads 8 --bam aligned.bam --teannotation hg38_te.bed --outputprefix gastric --outputdir out
```

## Evaluation signals

- Verify the BAM was produced with STAR options --winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, --outSAMmultNmax 1 so multi-mappers are retained but emitted once
- Confirm that SoloTE on the BAM yields locus-level assignments only for alignments with MAPQ >= 255 (uniquely mapped reads)
- In the early gastric cancer benchmark, the pipeline should reproduce 6 cell clusters and 24 marker TEs, including L1PA7 (chrY:18989629-18990627) and THE1D (chr8:61579930-61580458)
- The output matrix should contain both gene and TE features and load into Seurat without errors
- Family-level comparator (scTE) on the same data should yield only 3 marker TEs, confirming the locus-resolution advantage

## Limitations

- For organisms with a greater proportion of transpositionally-active TEs, most copies may be genetically intact and multi-mapping, limiting locus-level resolution
- Not all TEs can be uniquely mappable with the short 100 nucleotide reads generated by 3' end 10X protocols
- Events in which the TE acts as an alternative transcription start site cannot be detected due to 3' bias
- scTE indexes do not filter non-TE repeats such as Satellites, which appear in resulting matrices

## Evidence

- [discussion] Multi-mapping-permissive STAR alignment options: [evidence span withheld — anchor status: not_found]
- [discussion] MAPQ threshold for locus-level assignment: "a MAPQ equal to or greater than a user-defined threshold (default = 255, only uniquely mapped reads)"
- [discussion] Reads assigned to genes discarded by default: "reads assigned to known genes are discarded, to avoid mistakenly reporting TEs fully contained within genes, as expressed"
- [discussion] SoloTE can start from BAM: "SoloTE can start directly from the BAM file obtained with either CellRanger or STAR"
- [readme] BEDTools dependency: "BEDTools v2.29.2 or higher (https://github.com/arq5x/bedtools2/releases)"
