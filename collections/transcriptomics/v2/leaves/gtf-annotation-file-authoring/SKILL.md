---
name: gtf-annotation-file-authoring
description: Use when long-read alignments must be converted into novel transcript
  records in a valid GTF enhanced annotation, with TSS/polyA end-support filtering
  against external interval BEDs.
when_to_use_negative:
- Input reads are not full-length or not spliced — truncation-exclusion rules would
  discard most of the data instead of yielding novel isoforms
- The goal is differential isoform/exon testing rather than annotation extension —
  use scisorseqr DiffSplicingAnalysis or ExonQuant instead
- No external CAGE/polyA interval references are available for the genome version,
  so 5'/3' end assignments cannot be validated
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3117
tools:
- name: scisorseqr
  role: R-package supplying MapAndFilter for full-length, spliced read filtering and
    the workflow that collapses reads into the enhanced isoform annotation
  repo: https://github.com/noush-joglekar/scisorseqr
- name: STARlong
  role: Splice-aware alignment of PacBio long reads producing the BAM alignments consumed
    by MapAndFilter
  repo: https://github.com/alexdobin/STAR
- name: Minimap2
  role: Alternative splice-aware long-read aligner when STARlong is not used
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Downstream BAM handling in the scisorseqr mapping/filtering steps
- name: bedtools
  role: Interval operations supporting CAGE peak / polyA site end-assignment against
    the BED reference files
- name: Python
  role: Scripting the GTF emission and validation of novel isoform records (python
    >= 3.7, pandas, multiprocessing)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:fb5fd00fa07265a9cb36b4f30ab2c016dd729a3cbe789fb49d0ff48148ec191e
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
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3117
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# gtf-annotation-file-authoring

## Summary

Authoring an enhanced GTF annotation file of novel long-read isoforms by collapsing full-length, spliced ScISOr-Seq reads whose 5' and 3' ends are confirmed against FANTOM CAGE peaks and polyASite 2.0 sites; in this study it yielded exactly 76 high-confidence novel isoform records extending GENCODE.

## When to use

Use when long-read alignments must be converted into novel transcript records in a valid GTF enhanced annotation, with TSS/polyA end-support filtering against external interval BEDs.

## When NOT to use

- Input reads are not full-length or not spliced — truncation-exclusion rules would discard most of the data instead of yielding novel isoforms
- The goal is differential isoform/exon testing rather than annotation extension — use scisorseqr DiffSplicingAnalysis or ExonQuant instead
- No external CAGE/polyA interval references are available for the genome version, so 5'/3' end assignments cannot be validated

## Inputs

- Long-read BAM alignments (STARoutput/MMoutput from STARlong or Minimap2)
- GENCODE annotation GTF/GTF.gz (annoGZ)
- mm10 per-chromosome reference genome directory (seqDir, chr*.fa.gz)
- FANTOM CAGE peaks BED for mm10 (cageBed)
- polyASite 2.0 polyA sites BED (polyABed)

## Outputs

- enhanced_isoform_annotation.gtf containing 76 novel transcript feature rows with unique intron chains
- Novel intron-chain collapse records underlying the enhanced annotation

## How to apply

Load the deposited long-read BAM alignments (from STARoutput/MMoutput), the GENCODE annotation GTF (annoGZ), the per-chromosome mm10 genome directory (seqDir), the FANTOM CAGE peak BED (cageBed), and the polyASite 2.0 BED (polyABed). Run scisorseqr MapAndFilter with numThreads=24, filterFullLength=TRUE, and genomeVersion="mm10" to retain only mapped, spliced, full-length reads. Assign transcript start sites only when the read start lies within cp_distance=50 bp of an annotated FANTOM CAGE peak, and assign polyA sites only when the read end lies within 50 bp of a polyASite 2.0 site. Apply truncation-exclusion rules so reads lacking a confirmed 5' AND 3' end assignment are dropped, then collapse retained reads into novel intron chains and emit enhanced_isoform_annotation.gtf. The rationale is that end-anchoring against CAGE and polyA references prevents 5'-truncated or internally primed artifacts from being written as novel isoforms.

## Related tools

- **scisorseqr** (R-package supplying MapAndFilter for full-length, spliced read filtering and the workflow that collapses reads into the enhanced isoform annotation) — https://github.com/noush-joglekar/scisorseqr
- **STARlong** (Splice-aware alignment of PacBio long reads producing the BAM alignments consumed by MapAndFilter) — https://github.com/alexdobin/STAR
- **Minimap2** (Alternative splice-aware long-read aligner when STARlong is not used) — https://github.com/lh3/minimap2
- **samtools** (Downstream BAM handling in the scisorseqr mapping/filtering steps)
- **bedtools** (Interval operations supporting CAGE peak / polyA site end-assignment against the BED reference files)
- **Python** (Scripting the GTF emission and validation of novel isoform records (python >= 3.7, pandas, multiprocessing))

## Examples

```
MapAndFilter(annoGZ='PATH_TO_annotation.gtf.gz', numThreads=24, seqDir='PATH_TO_GENOME_DIR/chr*.fa.gz', genomeVersion='mm10', filterFullLength=TRUE)
```

## Evaluation signals

- Output file parses as valid GTF (file_format_is GTF)
- enhanced_isoform_annotation.gtf contains exactly 76 novel transcript feature rows (row_count_equals 76)
- Every retained read has both a confirmed 5' TSS (within cp_distance=50 bp of a FANTOM CAGE peak) and a confirmed 3' polyA site (within 50 bp of a polyASite 2.0 site); reads lacking either end are excluded
- Novel transcript rows have unique intron chains not present in the GENCODE reference annotation
- The generating script runs without error (script_runs); functional biotype assignment to novel isoforms is flagged for expert review

## Limitations

- Functional biotype assignment to the novel isoforms is incomplete and flagged for expert review
- Only reads lacking a confirmed 5' AND 3' end are excluded from the high-confidence set, so partially supported transcripts may be dropped conservatively
- Applicability beyond the tested environment is limited: the scisorseqr package 'has only been tested on a CentOS x86_64 machine'
- End-support thresholds (cp_distance=50 bp to CAGE peaks; 50 bp to polyA sites) may miss genuine isoforms with weak or tissue-specific end support

## Evidence

- [methods] CAGE/polyA end-assignment thresholds: [evidence span withheld — anchor status: not_found]
- [methods] MapAndFilter parameters: [evidence span withheld — anchor status: not_found]
- [methods] Expected output count and validation: [evidence span withheld — anchor status: not_found]
- [results] Origin of the novel isoforms: [evidence span withheld — anchor status: not_found]
- [readme] Tool scope from package README: [evidence span withheld — anchor status: external_doc]
