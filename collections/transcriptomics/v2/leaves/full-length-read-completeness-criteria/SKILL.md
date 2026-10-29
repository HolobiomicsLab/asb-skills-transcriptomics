---
name: full-length-read-completeness-criteria
description: Use when filtering long-read (PacBio/ONT) alignments for ScISOrSeq analysis
  and you need to decide which reads have confirmed transcript boundaries before collapsing
  novel isoforms.
when_to_use_negative:
- Reads are short Illumina reads rather than full-length long reads — end-based completeness
  criteria cannot be applied
- External CAGE or polyA reference BED files for the organism/genome build (e.g. mm10)
  are unavailable, so TSS/polyA end assignment cannot be verified
- The goal is transcript quantification rather than defining high-confidence full-length
  reads for isoform discovery — use IsoQuant instead
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scisorseqr
  role: Provides the MapAndFilter function enforcing full-length, spliced-read filtering
    and CAGE/polyA boundary assignment
  repo: github.com/noush-joglekar/scisorseqr
- name: STARlong
  role: Produces the BAM alignments of long reads that MapAndFilter screens for full-length
    completeness
  repo: github.com/alexdobin/STAR
- name: Minimap2
  role: Alternative long-read aligner (for ONT or PacBio reads) whose output feeds
    the same full-length filter
  repo: github.com/lh3/minimap2
- name: samtools
  role: Supports BAM manipulation in the mapping and filtering workflow
- name: bedtools
  role: Interval overlap operations for CAGE peak and polyA site assignment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:a6947c8fdc347503da363d5e0cf9f837e25f8aba6a6ba1d5259cde649125b3a7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Minimap2
  - Python
  - R
  - STARlong
  - bedtools
  - samtools
  - scisorseqr
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# full-length-read-completeness-criteria

## Summary

Defines how scisorseqr determines which long reads count as full-length and high-confidence: requiring spliced, mapped reads whose 5' start lies within 50 bp of a FANTOM CAGE peak (cp_distance=50) and whose 3' end lies within 50 bp of a polyASite 2.0 site, with truncation-exclusion rules dropping reads lacking both confirmed ends.

## When to use

Use when filtering long-read (PacBio/ONT) alignments for ScISOrSeq analysis and you need to decide which reads have confirmed transcript boundaries before collapsing novel isoforms.

## When NOT to use

- Reads are short Illumina reads rather than full-length long reads — end-based completeness criteria cannot be applied
- External CAGE or polyA reference BED files for the organism/genome build (e.g. mm10) are unavailable, so TSS/polyA end assignment cannot be verified
- The goal is transcript quantification rather than defining high-confidence full-length reads for isoform discovery — use IsoQuant instead

## Inputs

- Long-read BAM alignments from STARoutput/MMoutput
- GENCODE annotation GTF (annoGZ, gzipped)
- Per-chromosome mm10 reference genome FASTA directory (seqDir)
- FANTOM CAGE peak BED file for mm10 (cageBed)
- polyASite 2.0 atlas BED file (polyABed)

## Outputs

- Filtered set of full-length, spliced, boundary-confirmed long reads
- Novel isoform records for the enhanced annotation (enhanced_isoform_annotation.gtf), containing 76 high-confidence novel isoforms

## How to apply

After aligning PacBio reads with STARlong (or Nanopore reads with Minimap2), call scisorseqr's MapAndFilter with numThreads=24, filterFullLength=TRUE, and genomeVersion='mm10' to retain only mapped, spliced reads. Assign a transcript start site only when the read start lies within cp_distance=50 bp of an annotated FANTOM CAGE peak supplied as a BED file, and a polyA site only when the read end lies within 50 bp of a polyASite 2.0 BED interval. Apply truncation-exclusion so reads lacking both a confirmed 5' and 3' end assignment are excluded from the high-confidence set. Only these boundary-confirmed full-length reads feed downstream novel intron-chain collapse and the enhanced annotation GTF.

## Related tools

- **scisorseqr** (Provides the MapAndFilter function enforcing full-length, spliced-read filtering and CAGE/polyA boundary assignment) — github.com/noush-joglekar/scisorseqr
- **STARlong** (Produces the BAM alignments of long reads that MapAndFilter screens for full-length completeness) — github.com/alexdobin/STAR
- **Minimap2** (Alternative long-read aligner (for ONT or PacBio reads) whose output feeds the same full-length filter) — github.com/lh3/minimap2
- **samtools** (Supports BAM manipulation in the mapping and filtering workflow)
- **bedtools** (Interval overlap operations for CAGE peak and polyA site assignment)

## Examples

```
MapAndFilter(annoGZ='annotation.gtf.gz', numThreads=24, seqDir='~/genomes/mm10/chr*.fa.gz', filterFullLength=TRUE, genomeVersion='mm10', cageBed='cage_peaks.bed', polyABed='polyasite.bed')
```

## Evaluation signals

- Output enhanced annotation GTF parses as valid GTF (file_format_is GTF)
- The enhanced annotation contains exactly 76 novel transcript feature rows with unique intron chains (row_count_equals 76)
- The generating script runs without error (script_runs)
- Every retained read has both a TSS assigned within cp_distance=50 bp of a FANTOM CAGE peak and a polyA site within 50 bp of a polyASite 2.0 site; truncation-exclusion removed all others
- Assignment of functional biotypes to the 76 novel isoforms is flagged for expert review

## Limitations

- Truncated reads lacking a confirmed 5' or 3' end are excluded entirely, so genuinely expressed but boundary-incomplete isoforms are discarded
- The 50 bp cp_distance window ties end assignment strictly to external FANTOM CAGE and polyASite 2.0 references; ends outside these atlases will not be confirmed
- Long-read clustering and filtering discrepancies can stem from sparsity of the isoform matrix due to lower long-read throughput
- Functional biotypes for the novel isoforms are not automatically assigned and require expert review

## Evidence

- [methods] MapAndFilter full-length criteria: "MapAndFilter(annoGZ='[PATH_TO_annotation.gtf.gz]',numThreads=24, seqDir = '[PATH_TO_GENOME_DIR/chr*.fa.gz]',"
- [methods] CAGE/polyA distance windows: "annotated [CAGE peaks](https://fantom.gsc.riken.jp/5/datafiles/reprocessed/mm10_latest/extra/CAGE_peaks/)"
- [methods] PolyA atlas reference: "[PolyA sites](https://polyasite.unibas.ch/atlas) for our study"
- [readme] Full-length filter purpose from README: [evidence span withheld — anchor status: external_doc]
- [discussion] 76 novel isoforms finding: "Region-specific isoform testing revealed 395 genes with DIE between hippocampus and PFC, partially caused by isoforms novel to GENCODE."
