---
name: star-rnaseq-genome-alignment
description: Use when processing UMI-tagged single-end FASTQ reads (e.g. after fastq_qual_filter
  and fastq_umi_clipper) and a position-sorted SAM file is needed to quantify UMI
  collisions and mapping-shift artefacts.
when_to_use_negative:
- Paired-end sequencing workflows — the UMI-analysis pipeline README states 'Our use
  case only involves single ended sequencing at this stage'
- Analyses requiring splice-aware transcriptome alignment — the authors explicitly
  run STAR 'specifying no introns' for this UMI artefact quantification, not standard
  gene-expression quantification
- Input data that has not yet been demultiplexed and quality-filtered (the pipeline
  assumes 'your FASTQ file has been demultiplexed already, i.e. contains only one
  index')
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: STAR
  role: Aligns UMI-clipped reads to the genomic reference with introns disabled, yielding
    the sorted SAM required by umi_counterizer.pl
  repo: https://github.com/alexdobin/STAR
- name: fastq_umi_clipper
  role: Removes the 5' UMI (and optional protocol Gs) prior to alignment so alignments
    reflect sequence excluding the UMI
  repo: https://github.com/ncgr/UMI-analysis
- name: fastq_qual_filter
  role: Applies mean-quality threshold filtering (e.g. min_ok 30) on the read or UMI
    substring before alignment
  repo: https://github.com/ncgr/UMI-analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-31064-7
    title: Unique Molecular Identifiers reveal a novel sequencing artefact with implications
      for RNA-Seq based gene expression analysis
  source_content_hash: sha256:f7fb482884eb7e2fa41efbc9d148c81c241344ee3ef2492d70f0b0c32db6f844
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexdobin/STAR
  tools:
  - C
  - STAR
  - Text::Levenshtein::XS
  - Try::Tiny
  - fastq_qual_filter
  - fastq_umi_clipper
  - pcregrep
  - uthash
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# star-rnaseq-genome-alignment

## Summary

Align quality-filtered, UMI-clipped single-end reads to a genomic reference with STAR (specifying no introns) to produce a sorted SAM file, which is the required input for downstream UMI duplicate collapsing in the UMI-analysis pipeline.

## When to use

Use when processing UMI-tagged single-end FASTQ reads (e.g. after fastq_qual_filter and fastq_umi_clipper) and a position-sorted SAM file is needed to quantify UMI collisions and mapping-shift artefacts.

## When NOT to use

- Paired-end sequencing workflows — the UMI-analysis pipeline README states 'Our use case only involves single ended sequencing at this stage'
- Analyses requiring splice-aware transcriptome alignment — the authors explicitly run STAR 'specifying no introns' for this UMI artefact quantification, not standard gene-expression quantification
- Input data that has not yet been demultiplexed and quality-filtered (the pipeline assumes 'your FASTQ file has been demultiplexed already, i.e. contains only one index')

## Inputs

- FASTQ file quality-filtered and UMI-clipped by fastq_qual_filter and fastq_umi_clipper (UMI moved into FASTQ header, 5' Gs optionally removed)
- Reference genome index for STAR

## Outputs

- Position-sorted SAM file containing alignments with UMIs embedded in read headers
- Alignment log files (e.g. Log.final.out, SJ.out.tab) produced by STAR

## How to apply

After quality filtering (fastq_qual_filter, e.g. min_ok ~30) and clipping the 5' UMI with fastq_umi_clipper (setting clip_length to the UMI length and g_length to remove protocol-derived trailing Gs from the Clontech SMARter template-switching system), align the clipped FASTQ to the reference genome using STAR with intron detection disabled ('specifying no introns'). Obtain a sorted SAM file, since the next pipeline step, umi_counterizer.pl, requires 'a sorted SAM file' as input and parses UMIs from each alignment's position. The downstream analysis (detecting reads sharing a UMI but mapping to closely spaced positions within ~3 bp) depends on accurate positional mapping, so coordinate-sorted output is essential. Note the authors' use case is single-end sequencing only; paired-end workflows are outside the validated scope.

## Related tools

- **STAR** (Aligns UMI-clipped reads to the genomic reference with introns disabled, yielding the sorted SAM required by umi_counterizer.pl) — https://github.com/alexdobin/STAR
- **fastq_umi_clipper** (Removes the 5' UMI (and optional protocol Gs) prior to alignment so alignments reflect sequence excluding the UMI) — https://github.com/ncgr/UMI-analysis
- **fastq_qual_filter** (Applies mean-quality threshold filtering (e.g. min_ok 30) on the read or UMI substring before alignment) — https://github.com/ncgr/UMI-analysis

## Examples

```
STAR --runThreadN 8 --genomeDir /path/to/genome_idx --readFilesIn clipped.good.fq --outSAMtype SAM SortedByCoordinate --outFileNamePrefix sample1_
```

## Evaluation signals

- Output must be a position-sorted SAM file — umi_counterizer.pl assumes sorted order and fails or misbehaves otherwise
- STAR Log.final.out should report a mapping rate consistent with the quality-filtering stringency (e.g. mean-quality threshold ~30 in fastq_qual_filter)
- Downstream all_sam_umis.txt should contain every UMI observed in the SAM file, confirming UMI headers survived clipping and alignment
- Positional UMI counts (position_umi_counts.bed) should show UMI collisions concentrated at closely spaced positions, reproducing the bell-shaped mapping-shift distributions peaking at zero shift reported in the paper

## Limitations

- The authors use STAR 'specifying no introns', so spliced alignments are not produced; this is appropriate for the UMI-shift analysis but not for general transcript quantification
- No tool version numbers are specified in the paper's methods ('Warning: not much arg checking is done so unexpected behavior may be encountered' applies to the surrounding C/Perl pipeline tools)
- The pipeline assumes single-end reads and pre-demultiplexed FASTQ input

## Evidence

- [methods] STAR with no introns for UMI artefact analysis: [evidence span withheld — anchor status: not_found]
- [methods] UMI clipping removes protocol-derived Gs before alignment: [evidence span withheld — anchor status: not_found]
- [readme] Single-end sequencing assumption: "Our use case only involves single ended sequencing at this stage."
- [readme] Downstream duplicate elimination requires sorted SAM: [evidence span withheld — anchor status: not_found]
- [readme] Pre-demultiplexed, single-index FASTQ assumption: "This assumes your FASTQ file has been demultiplexed already, i.e. contains only one index, or that you don't care what the indices are."
