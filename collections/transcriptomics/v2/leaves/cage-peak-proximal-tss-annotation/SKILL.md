---
name: cage-peak-proximal-tss-annotation
description: Use when building an enhanced long-read isoform annotation and you need
  to confirm read 5'/3' ends against CAGE peak (FANTOM, mm10) and polyA site (polyASite
  2.0) BED intervals before collapsing novel intron chains.
when_to_use_negative:
- You only need gene-level or exon-level quantification from short reads — this skill
  addresses novel long-read isoform discovery, not bulk expression testing.
- Your reads cannot be assessed for full-length status (e.g. 3'-end-only or degraded
  library) — reads lacking a confirmed 5' and 3' end are excluded by the truncation
  rules.
- No external CAGE peak or polyA site reference exists for your genome version, since
  TSS/polyA assignment depends entirely on proximity to these interval files.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: scisorseqr
  role: R package supplying MapAndFilter and InfoPerLongRead to filter full-length
    spliced reads and assign TSS/polyA by CAGE/polyA proximity
  repo: https://github.com/noush-joglekar/scisorseqr
- name: STARlong
  role: Splice-aware aligner producing the BAM alignments that feed the CAGE/polyA
    end-assignment step for PacBio reads
  repo: https://github.com/alexdobin/STAR
- name: Minimap2
  role: Alternative splice-aware aligner (MMalign) for Oxford Nanopore or PacBio reads
    prior to end-annotation
  repo: https://github.com/lh3/minimap2
- name: bedtools
  role: Interval arithmetic used alongside scisorseqr for proximity overlap of read
    ends against CAGE/polyA BED files
- name: samtools
  role: BAM manipulation of filtered long-read alignments
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:05620ca3ecd61a137e1d28228f515a90d8c6fb21ce90239a9b992f7d92aebf28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noush-joglekar/scisorseqr
  tools:
  - Minimap2
  - Python
  - R
  - STARlong
  - bedtools
  - samtools
  - scisorseqr
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# CAGE-peak-proximal TSS annotation

## Summary

Annotates long-read transcript start sites (TSS) and polyA sites by proximity to external FANTOM CAGE peak and polyASite 2.0 interval files, so that only reads with externally confirmed 5' and 3' ends contribute novel intron chains to an enhanced annotation (76 high-confidence novel isoforms in the source study).

## When to use

Use when building an enhanced long-read isoform annotation and you need to confirm read 5'/3' ends against CAGE peak (FANTOM, mm10) and polyA site (polyASite 2.0) BED intervals before collapsing novel intron chains.

## When NOT to use

- You only need gene-level or exon-level quantification from short reads — this skill addresses novel long-read isoform discovery, not bulk expression testing.
- Your reads cannot be assessed for full-length status (e.g. 3'-end-only or degraded library) — reads lacking a confirmed 5' and 3' end are excluded by the truncation rules.
- No external CAGE peak or polyA site reference exists for your genome version, since TSS/polyA assignment depends entirely on proximity to these interval files.

## Inputs

- Long-read BAM alignments (STARlong or Minimap2 output directories, e.g. STARoutput/MMoutput)
- GENCODE annotation GTF (annoGZ)
- Per-chromosome mm10 reference genome FASTA directory (seqDir, chr*.fa.gz)
- FANTOM CAGE peak BED file for mm10 (cageBed)
- polyASite 2.0 polyA site BED file (polyABed)
- FASTQ long-read files plus barcode-celltype assignments (from Seurat short-read analysis)

## Outputs

- enhanced_isoform_annotation.gtf containing 76 novel transcript records with unique intron chains
- Per-read end-assignment records (TSS within 50 bp of a CAGE peak; polyA within 50 bp of a polyASite 2.0 site)
- Filtered set of full-length, spliced reads passing truncation-exclusion rules

## How to apply

Load the deposited long-read BAM alignments (from STARoutput/MMoutput), the GENCODE annotation GTF, the per-chromosome mm10 genome directory, the FANTOM CAGE peak BED, and the polyASite 2.0 BED. Run scisorseqr's MapAndFilter (numThreads=24, filterFullLength=TRUE, genomeVersion="mm10") to retain only mapped, spliced, full-length reads. Assign a transcript start site only when the read start lies within cp_distance=50 bp of an annotated FANTOM CAGE peak, and a polyA site only when the read end lies within 50 bp of a polyASite 2.0 site. Apply truncation-exclusion rules so reads lacking both a confirmed 5' and 3' end are dropped from the high-confidence set, then collapse retained reads into novel intron chains and emit an enhanced annotation GTF. The rationale is that end-proximity to independent CAGE/polyA evidence protects against internal priming and 5' truncation artifacts that would otherwise inflate novel isoform calls; in the source study this yielded 76 high-confidence novel isoforms among the top two isoforms of 395 differentially isoform-expressed genes.

## Related tools

- **scisorseqr** (R package supplying MapAndFilter and InfoPerLongRead to filter full-length spliced reads and assign TSS/polyA by CAGE/polyA proximity) — https://github.com/noush-joglekar/scisorseqr
- **STARlong** (Splice-aware aligner producing the BAM alignments that feed the CAGE/polyA end-assignment step for PacBio reads) — https://github.com/alexdobin/STAR
- **Minimap2** (Alternative splice-aware aligner (MMalign) for Oxford Nanopore or PacBio reads prior to end-annotation) — https://github.com/lh3/minimap2
- **bedtools** (Interval arithmetic used alongside scisorseqr for proximity overlap of read ends against CAGE/polyA BED files)
- **samtools** (BAM manipulation of filtered long-read alignments)

## Examples

```
MapAndFilter(annoGZ='annotation.gtf.gz', numThreads=24, seqDir='~/genomes/mm10/chr*.fa.gz', filterFullLength=TRUE, genomeVersion="mm10", cageBed='fantom_cage_mm10.bed', polyABed='polyasite2.bed', cp_distance=50)
```

## Evaluation signals

- Output enhanced_isoform_annotation.gtf parses as valid GTF (file_format_is GTF).
- GTF contains exactly 76 novel transcript feature rows, each with a unique intron chain (row_count_equals 76).
- Generating script runs without error (script_runs).
- Every retained read start lies within cp_distance=50 bp of a FANTOM CAGE peak and every retained read end within 50 bp of a polyASite 2.0 site; reads failing both end confirmations are absent from the high-confidence set.
- Novel isoforms reconcile with the DIE analysis: they appear among the top two contributing isoforms of the 395 genes with regional differential isoform expression.

## Limitations

- Reads lacking a confirmed 5' and 3' end assignment are excluded, so truncated long reads silently drop out and may bias novel isoform discovery toward full-length molecules.
- Assignment of functional biotypes to the 76 novel isoforms is unverified and flagged for expert review.
- cp_distance=50 bp is a fixed tolerance; TSS/polyA assignments are only as good as the FANTOM CAGE (mm10) and polyASite 2.0 reference coverage, which is tissue- and condition-dependent.

## Evidence

- [methods] Assign transcript start sites only when the read start lies within cp_distance=50 bp of an annotated FANTOM CAGE peak: "annotated [CAGE peaks](https://fantom.gsc.riken.jp/5/datafiles/reprocessed/mm10_latest/extra/CAGE_peaks/)"
- [methods] MapAndFilter with full-length filtering parameters: "MapAndFilter(annoGZ='[PATH_TO_annotation.gtf.gz]',numThreads=24, 
             seqDir = '[PATH_TO_GENOME_DIR/chr*.fa.gz]',"
- [discussion] 76 novel isoforms among top two isoforms of 395 DIE genes: "Region-specific isoform testing revealed 395 genes with DIE between hippocampus and PFC, partially caused by isoforms novel to GENCODE."
- [readme] scisorseqr filters high-confidence full-length spliced reads: "integration with long read alignment tools, mapping and
filtering of high confidence, full-length spliced reads, and some handy
tools to conduct differential expression analysis."
- [methods] PolyA sites reference used in the study: "[PolyA sites](https://polyasite.unibas.ch/atlas) for our study"
