---
name: novel-intron-chain-isoform-reconstruction
description: Use when long-read BAM/FASTQ alignments plus a reference annotation (e.g.
  GENCODE mm10 GTF) are available and the goal is to discover novel isoforms by requiring
  end-support from CAGE TSS and polyA site references before collapsing intron chains.
when_to_use_negative:
- Reads are short-read-only (e.g. bulk Illumina junctions alone) and cannot support
  full-length isoform reconstruction with confirmed 5'/3' ends
- The goal is differential isoform expression testing among already-annotated isoforms
  (use DiffSplicingAnalysis/ExonQuant instead)
- No external TSS/polyA reference (FANTOM CAGE, polyASite) is available for the genome
  version, so end-based confidence filtering cannot be applied
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scisorseqr
  role: R package providing MapAndFilter and downstream collapse into novel intron-chain
    isoform records
  repo: https://github.com/noush-joglekar/scisorseqr
- name: STARlong
  role: Splice-aware alignment of PacBio long reads to the mm10 genome to produce
    input BAMs
  repo: https://github.com/alexdobin/STAR
- name: Minimap2
  role: Alternative splice-aware long-read aligner used if STARlong is not used
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: BAM handling for filtered full-length spliced reads
- name: bedtools
  role: Interval operations for CAGE/polyA end-assignment against BED references
- name: Python
  role: Scripting the collapse of retained reads into novel intron chains and GTF
    emission
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:1ada61932339c295f6114cb697fde17b924e7c2a8fbc4b0f02545680b645c54c
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
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# novel-intron-chain-isoform-reconstruction

## Summary

Reconstruct high-confidence novel isoforms by collapsing full-length, spliced long reads (ScISOr-Seq) whose 5' and 3' ends are independently validated against FANTOM CAGE peaks and polyASite 2.0 sites, yielding an enhanced GTF of novel intron-chain records (76 novel isoforms in the mouse P7 brain study).

## When to use

Use when long-read BAM/FASTQ alignments plus a reference annotation (e.g. GENCODE mm10 GTF) are available and the goal is to discover novel isoforms by requiring end-support from CAGE TSS and polyA site references before collapsing intron chains.

## When NOT to use

- Reads are short-read-only (e.g. bulk Illumina junctions alone) and cannot support full-length isoform reconstruction with confirmed 5'/3' ends
- The goal is differential isoform expression testing among already-annotated isoforms (use DiffSplicingAnalysis/ExonQuant instead)
- No external TSS/polyA reference (FANTOM CAGE, polyASite) is available for the genome version, so end-based confidence filtering cannot be applied

## Inputs

- Long-read BAM alignments from STARlong or Minimap2 (STARoutput/MMoutput)
- GENCODE annotation GTF (mm10, annoGZ)
- Per-chromosome mm10 reference genome FASTA directory (seqDir)
- FANTOM CAGE peak BED file for mm10 (cageBed)
- polyASite 2.0 polyA-site BED file (polyABed)

## Outputs

- enhanced_isoform_annotation.gtf — GTF rows of novel transcript records with unique intron chains
- Set of high-confidence novel isoforms supported by CAGE and polyA end assignments (76 in the source study)

## How to apply

Load the long-read alignments (STARoutput/MMoutput BAM) together with the GENCODE annotation GTF (annoGZ), the per-chromosome mm10 genome directory (seqDir), the FANTOM CAGE peak BED (cageBed), and the polyASite 2.0 BED (polyABed). Run scisorseqr MapAndFilter with numThreads=24 and filterFullLength=TRUE so only mapped, spliced, full-length reads are retained. Assign transcript start sites only when the read start lies within cp_distance=50 bp of a FANTOM CAGE peak, and polyA sites only when the read end lies within 50 bp of a polyASite site; apply truncation-exclusion so reads lacking both a confirmed 5' and 3' end are discarded. Collapse the retained reads into novel intron chains and emit an enhanced annotation GTF of novel isoform records (enhanced_isoform_annotation.gtf). In the source study this procedure yielded 76 high-confidence novel isoforms among the top two contributing isoforms of 395 genes with regional differential isoform expression between hippocampus and prefrontal cortex.

## Related tools

- **scisorseqr** (R package providing MapAndFilter and downstream collapse into novel intron-chain isoform records) — https://github.com/noush-joglekar/scisorseqr
- **STARlong** (Splice-aware alignment of PacBio long reads to the mm10 genome to produce input BAMs) — https://github.com/alexdobin/STAR
- **Minimap2** (Alternative splice-aware long-read aligner used if STARlong is not used) — https://github.com/lh3/minimap2
- **samtools** (BAM handling for filtered full-length spliced reads)
- **bedtools** (Interval operations for CAGE/polyA end-assignment against BED references)
- **Python** (Scripting the collapse of retained reads into novel intron chains and GTF emission)

## Examples

```
MapAndFilter(annoGZ='gencode.vM.gtf.gz', numThreads=24, seqDir='~/genomes/mm10/chr*.fa.gz', filterFullLength=TRUE, genomeVersion='mm10', cageBed='fantom_cage_mm10.bed', polyABed='polyasite.bed', cp_distance=50)
```

## Evaluation signals

- Output file parses as valid GTF (file_format_is GTF)
- File contains exactly 76 novel transcript feature rows with unique intron chains (row_count_equals 76)
- Generating script runs without error (script_runs)
- Every retained novel isoform has both a confirmed 5' end within cp_distance=50 bp of a FANTOM CAGE peak and a 3' end within 50 bp of a polyASite 2.0 site
- Truncation-exclusion audit shows no reads lacking confirmed 5' and 3' assignments in the high-confidence set

## Limitations

- Assignment of functional biotypes to the novel isoforms was flagged for expert review and not automated
- Long-read throughput limits isoform-matrix density; clustering discrepancies may stem from sparsity of the isoform matrix
- Whether observed patterns generalize to adult brain regions, disease case-control studies, or other species is untested

## Evidence

- [methods] MapAndFilter end-assignment parameters: "MapAndFilter(annoGZ='[PATH_TO_annotation.gtf.gz]',numThreads=24, 
             seqDir = '[PATH_TO_GENOME_DIR/chr*.fa.gz]',"
- [results] 76 novel isoforms finding: [evidence span withheld — anchor status: not_found]
- [methods] CAGE/GENCODE dataset usage: "annotated [CAGE peaks](https://fantom.gsc.riken.jp/5/datafiles/reprocessed/mm10_latest/extra/CAGE_peaks/)"
- [readme] scisorseqr capability per README: "mapping and
filtering of high confidence, full-length spliced reads, and some handy
tools to conduct differential expression analysis"
