---
name: splice-aware-alignment-benchmarking
description: Use when you must quantify recall/precision of novel splice-junction
  detection against held-out AStalavista truth events, or compare align-and-count
  vs pseudoalignment runtime/memory on tens to hundreds of millions of RNA-seq reads.
when_to_use_negative:
- Goal is plain abundance quantification of annotated transcripts only — a vanilla
  kallisto run suffices without fortuna's novel-fragment guessing or benchmarking
  harness.
- Ground truth involves splice sites with no annotated basis (truly novel donor/acceptor
  pairs) — fortuna --quant alone guesses combinations of annotated splice sites, so
  a spliced aligner plus fortuna --refine is needed instead.
- Reads are already spliced-aligned to a genome BAM and you only want event annotation
  from existing alignments rather than a mapping-accuracy comparison.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: fortuna
  role: Method under evaluation; --index builds the kallisto fragment index per read
    length and --quant pseudoaligns reads to yield signature counts and alt.tsv novel
    events; --refine optionally incorporates STAR-found novel splice sites.
  repo: https://github.com/canzarlab/fortuna
- name: kallisto
  role: Pseudoalignment engine bundled with fortuna; index is built from annotated
    and guessed transcript fragments and equivalence classes yield elementary splicing-unit
    counts.
  repo: https://github.com/pachterlab/kallisto
- name: STAR
  role: Baseline spliced aligner run in 1-pass and 2-pass modes for recall/precision
    comparison and as the align-and-count chain (1-pass + samtools + sigcount) in
    the runtime/memory benchmark.
- name: Whippet
  role: Baseline junction/event quantifier run on the same FASTQs for per-event supporting
    read counts and timing comparison on all subsamples.
- name: htslib
  role: Bundled dependency of fortuna and used with samtools to sort/index STAR BAMs
    in the align-and-count benchmark chain.
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:a5b6c35c43727fdf35a1c100354c6097ce8c430ae6854f272bd8bb7f42fbcfce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/fortuna
  tools:
  - STAR
  - Whippet
  - fortuna
  - htslib
  - kallisto
  - samtools
  - sigcount
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# splice-aware-alignment-benchmarking

## Summary

Benchmark protocol from the fortuna paper for measuring how accurately splice-aware mappers and pseudoaligners (fortuna, STAR 1/2-pass, Whippet) detect novel junctions between annotated splice sites, plus runtime and peak-memory scaling on up to ~291M real RNA-seq reads.

## When to use

Use when you must quantify recall/precision of novel splice-junction detection against held-out AStalavista truth events, or compare align-and-count vs pseudoalignment runtime/memory on tens to hundreds of millions of RNA-seq reads.

## When NOT to use

- Goal is plain abundance quantification of annotated transcripts only — a vanilla kallisto run suffices without fortuna's novel-fragment guessing or benchmarking harness.
- Ground truth involves splice sites with no annotated basis (truly novel donor/acceptor pairs) — fortuna --quant alone guesses combinations of annotated splice sites, so a spliced aligner plus fortuna --refine is needed instead.
- Reads are already spliced-aligned to a genome BAM and you only want event annotation from existing alignments rather than a mapping-accuracy comparison.

## Inputs

- 80M paired-end simulated FASTQ files at 75 bp and 100 bp read lengths (FluxSimulator, RefSeq release 109 transcripts)
- Genome FASTA and transcript annotation GTF (RefSeq 109 for simulation; GENCODE 42 GRCh38.p13 for human indices)
- Segmented subexon annotation (annotation.rg.gtf) produced by bin/processGTF.sh
- Withheld AStalavista truth event list (exon skipping, alternative donor, alternative acceptor)
- ~291M-read ASD lymphoblastoid sample 29 FASTQ and its 10–90% fractional subsamples

## Outputs

- Recall and precision tables per event type (ES, AD, AA) reproducing the Fig. 4 / Supplementary Table S1 pattern
- Per-event supporting read counts (supptable_s3.tsv) cross-tabulating fortuna vs STAR vs Whippet on 8 ASD exon-skipping events
- benchmark_scaling.tsv with runtime and peak memory for each tool across the 9 subsample fractions
- Signature counts (cnt.tsv) and annotated novel splicing events with supporting read counts (alt.tsv)

## How to apply

Simulate 80M paired-end reads with FluxSimulator from RefSeq release 109 transcripts at 75 bp and 100 bp, and derive a partial ground-truth annotation by removing transcripts whose AStalavista-identified exon skipping / alternative donor / alternative acceptor events serve as held-out truth. Preprocess the GTF with fortuna's bin/processGTF.sh into minimal disjoint subexon segments (.rg.gtf), build a fortuna --index per read length (-rl 75 and -rl 100), and run fortuna --quant with -miss 3 on 4 threads to get cnt.tsv signature counts and alt.tsv novel events. Run STAR 1-pass, STAR 2-pass, and Whippet on the same FASTQs, then score every tool against the withheld events, computing recall and precision by event type, hits-per-read (expected ~1.02/1.04/1.16), and runtime. For scaling, time fortuna --quant, Whippet, and a STAR 1-pass + samtools + sigcount chain on 10–90% subsamples of the ~291M-read ASD sample 29 and record peak memory (Whippet ~2 GB, kallisto ~4 GB, fortuna up to ~24 GB, STAR ~29 GB). Optionally add fortuna --refine with the STAR BAM to incorporate novel splice sites found in reads unmapped during pseudoalignment.

## Related tools

- **fortuna** (Method under evaluation; --index builds the kallisto fragment index per read length and --quant pseudoaligns reads to yield signature counts and alt.tsv novel events; --refine optionally incorporates STAR-found novel splice sites.) — https://github.com/canzarlab/fortuna
- **kallisto** (Pseudoalignment engine bundled with fortuna; index is built from annotated and guessed transcript fragments and equivalence classes yield elementary splicing-unit counts.) — https://github.com/pachterlab/kallisto
- **STAR** (Baseline spliced aligner run in 1-pass and 2-pass modes for recall/precision comparison and as the align-and-count chain (1-pass + samtools + sigcount) in the runtime/memory benchmark.)
- **Whippet** (Baseline junction/event quantifier run on the same FASTQs for per-event supporting read counts and timing comparison on all subsamples.)
- **htslib** (Bundled dependency of fortuna and used with samtools to sort/index STAR BAMs in the align-and-count benchmark chain.) — https://github.com/samtools/htslib

## Examples

```
grep -P "chr2\t190984393\t190986854\t" alt.tsv
```

## Evaluation signals

- Rebuilt table reproduces the Fig. 4 / Supplementary Table S1 pattern: fortuna has highest recall and precision on mismatch-containing junction-spanning reads, while Whippet's sensitivity drops to 68–71%.
- fortuna maps over 99% of the simulated reads, and hits-per-read values match the reported ~1.02/1.04/1.16 within rounding.
- fortuna counts all ~291 million reads of ASD sample 29 in around 15 min using 4 threads, more than 2x (≈7x) faster than STAR 1-pass + sigcount (159 min) and faster than Whippet on all but the smallest subsample.
- Peak memory falls near reported ranges: Whippet ~2 GB, kallisto ~4 GB, fortuna up to ~24 GB, STAR ~29 GB.
- Example-dataset sanity check: grep of alt.tsv for chr2 190984393 190986854 returns an ES event on Locus_18066 supported by 1256 reads, with >99% of the ~240,000 STAT1 reads mapped.

## Limitations

- Ground truth is partial: truth events are held out by removing transcripts with AStalavista-identified events from the simulation annotation, so accuracy is conditional on novel junctions formed between annotated splice sites only.
- Aligner baselines vary greatly in scalability — ASGAL shows low recall and ~32 h genome-wide runtime, which limits a complete comparison.
- fortuna's peak memory (up to ~24 GB) exceeds Whippet (~2 GB) and kallisto (~4 GB), so it is not always the lightest option despite its speed.
- Reads that remain unmapped during pseudoalignment carry potential truly novel splice sites; they must be recovered via fortuna --refine with a spliced aligner's BAM or they are missed.

## Evidence

- [abstract] fortuna was around 7× faster than traditional align and count approaches, and was able to analyze almost 300 million reads in just 15 min when using four threads.: "fortuna was around 7× faster than traditional align and count approaches, and was able to analyze almost 300 million reads in just 15 min when using four threads."
- [methods] Preprocessing: disjoint subexon segmentation of the GTF.: "A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites"
- [intro] Optional --refine step for novel splice sites from unmapped reads.: [evidence span withheld — anchor status: not_found]
- [readme] Example-dataset validation criterion for mapping rate and novel ES support.: "fortuna will map over 99% of those reads and find slightly over 1000 reads supporting a novel exon skipping at chr2:190984393-190986854."
