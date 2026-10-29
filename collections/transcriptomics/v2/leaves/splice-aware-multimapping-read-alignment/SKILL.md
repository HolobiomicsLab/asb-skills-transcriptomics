---
name: splice-aware-multimapping-read-alignment
description: Use when preprocessing scRNA-Seq reads (FASTQ) for transposable element
  expression analysis where TEs are repetitive and multi-mapping must be retained
  rather than filtered, e.g., building BAMs for SoloTE or benchmarking against ground
  truth.
when_to_use_negative:
- The analysis goal is standard gene-level expression quantification with CellRanger
  defaults — the TE-oriented multi-mapping options (--outSAMmultNmax 1, multimapNmax
  100) are specific to TE detection workflows
- The input is already a BAM or expression matrix — this skill applies only at the
  FASTQ-to-BAM alignment stage
- Reads come from protocols requiring full-length or 5'-biased transcript reconstruction
  — the article notes TEs acting as alternative transcription start sites cannot be
  detected due to 3' bias, and EM-based multi-mapper resolution on 3' end 10X data
  is unassessed
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: STAR
  role: Splice-aware aligner run with TE-specific multi-mapping options to produce
    BAMs that retain multimappers for downstream TE assignment
- name: SoloTE
  role: Consumes the STAR/CellRanger BAM and BEDTools-intersects non-gene reads against
    TE annotations, summarizing uniquely mapped reads at locus level and multi-mappers
    at family level
  repo: github.com/bvaldebenitom/SoloTE
- name: BEDTools
  role: Assesses overlap of non-gene-assigned reads with the TE BED annotation within
    the SoloTE pipeline (v2.29.2 or higher)
  repo: github.com/arq5x/bedtools2
- name: CellRanger
  role: Alternative standard scRNA-Seq pipeline whose output BAM can be fed directly
    into SoloTE instead of a STAR alignment
- name: Minnow
  role: Simulator used to generate scRNA-Seq reads with known ground-truth TE expression
    for benchmarking the alignment/quantification workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:62deb00c8cbe4c24324ae178271db9dabe57be96e335952b74953d1ba247f735
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BEDTools
  - CellRanger
  - Minnow
  - STAR
  - SoloTE
  - scTE
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Splice-aware multimapping read alignment

## Summary

Align single-cell RNA-Seq reads with STAR configured to retain multi-mapping reads (up to 100 genomic loci), producing the BAM files from which SoloTE assigns locus-specific transposable element expression, with uniquely mapped reads reported at locus level and multi-mappers at family level.

## When to use

Use when preprocessing scRNA-Seq reads (FASTQ) for transposable element expression analysis where TEs are repetitive and multi-mapping must be retained rather than filtered, e.g., building BAMs for SoloTE or benchmarking against ground truth.

## When NOT to use

- The analysis goal is standard gene-level expression quantification with CellRanger defaults — the TE-oriented multi-mapping options (--outSAMmultNmax 1, multimapNmax 100) are specific to TE detection workflows
- The input is already a BAM or expression matrix — this skill applies only at the FASTQ-to-BAM alignment stage
- Reads come from protocols requiring full-length or 5'-biased transcript reconstruction — the article notes TEs acting as alternative transcription start sites cannot be detected due to 3' bias, and EM-based multi-mapper resolution on 3' end 10X data is unassessed

## Inputs

- Single-cell RNA-Seq reads (e.g., simulated FASTQ from Minnow, or 3' end 10X reads)
- Reference genome FASTQ/index for hg38 or mm10 (or other UCSC genome versions listable via SoloTE_RepeatMasker_to_BED.py -l)
- RepeatMasker TE annotation converted to SoloTE BED format (column-4 ID: chr|start|end|Subfamily:Family:Class|strand)

## Outputs

- Sorted BAM file with STAR multi-mapping options applied (one representative alignment per read pair, up to 100 candidate multi-mapping loci)
- Downstream locus-level TE expression matrix (via SoloTE) derived from the BAM

## How to apply

Align the reads with STAR using multi-mapping parameters --winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, and --outSAMmultNmax 1; these options allow up to 100 multi-mapping loci per read while reporting a single representative alignment per read pair in the BAM. This is essential because old versus young TEs (divergence >10% vs ≤10% from consensus) differ drastically in mappability, and discarding multi-mappers would erase young TE signal. The resulting BAM (from STAR or CellRanger) is the direct input to SoloTE ('python SoloTE_pipeline.py --threads N --bam file --teannotation BEDfile --outputprefix P --outputdir D'), where reads not assigned to genes are intersected with the RepeatMasker-format TE annotation via BEDTools, and MAPQ ≥ threshold (default 255, uniquely mapped reads only) determines locus-level summarization versus family-level summarization for multi-mappers. For benchmarking, the same BAMs feed both SoloTE and scTE so TP/FP/FN counts can be compared against simulated ground truth.

## Related tools

- **STAR** (Splice-aware aligner run with TE-specific multi-mapping options to produce BAMs that retain multimappers for downstream TE assignment)
- **SoloTE** (Consumes the STAR/CellRanger BAM and BEDTools-intersects non-gene reads against TE annotations, summarizing uniquely mapped reads at locus level and multi-mappers at family level) — github.com/bvaldebenitom/SoloTE
- **BEDTools** (Assesses overlap of non-gene-assigned reads with the TE BED annotation within the SoloTE pipeline (v2.29.2 or higher)) — github.com/arq5x/bedtools2
- **CellRanger** (Alternative standard scRNA-Seq pipeline whose output BAM can be fed directly into SoloTE instead of a STAR alignment)
- **Minnow** (Simulator used to generate scRNA-Seq reads with known ground-truth TE expression for benchmarking the alignment/quantification workflow)

## Examples

```
STAR --runThreadN N --genomeDir hg38_idx --readFilesIn R1.fq R2.fq --winAnchorMultimapNmax 100 --outFilterMultimapNmax 100 --outSAMmultNmax 1 --outSAMtype BAM SortedByCoordinate
```

## Evaluation signals

- BAM header/flags confirm STAR options --winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, and --outSAMmultNmax 1 were used (at most one SAM record per read pair)
- SoloTE run on the BAM produces a combined gene+TE expression matrix compatible with Seurat
- MAPQ distribution shows uniquely mapped reads (MAPQ 255) distinguishable from multi-mappers so locus-level vs family-level assignment is possible
- In benchmarking, per-locus TP/FP/FN counts against the Minnow ground truth can be aggregated across 2 genomes × 2 TE groups (young ≤10% vs old >10% divergence) × 2 methods
- SoloTE_RepeatMasker_to_BED.py output BED matches the required 4-column ID format chr|start|end|Subfamily:Family:Class|strand

## Limitations

- Not all TEs can be uniquely mappable with the short 100-nucleotide reads from 3' end 10X protocols (Supplementary Figures 4 and 5), so young TEs show higher FP/FN proportions
- EM-algorithm multi-mapper resolution has not been assessed for 3' end 10X data (future work)
- In organisms with a greater proportion of transpositionally-active TEs, most copies could be genetically intact and multi-mapping, limiting locus-level assignment
- For very young TEs, even locus-specific approaches like CELLO-Seq were unable to map reads to specific genomic locations

## Evidence

- [discussion] STAR multi-mapping options: [evidence span withheld — anchor status: not_found]
- [discussion] MAPQ threshold for locus-level assignment: "a MAPQ equal to or greater than a user-defined threshold (default = 255, only uniquely mapped reads)"
- [discussion] SoloTE BAM input flexibility: "SoloTE can start directly from the BAM file obtained with either CellRanger or STAR"
- [readme] SoloTE pipeline invocation: "python SoloTE_pipeline.py --threads NumberOfThreads --bam BAMfile --teannotation BEDfile --outputprefix Prefix --outputdir OutputDirectory"
- [discussion] Young/old TE divergence classification: "TEs were divided into young and old, if their divergence from the consensus sequence was ≤10% or >10%, respectively"
