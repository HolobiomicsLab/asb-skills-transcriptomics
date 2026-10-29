---
name: rna-seq-read-simulation
description: 'Use when benchmarking junction/splicing-event detection accuracy on
  controlled data: you need simulated FASTQs with known held-out novel junctions and
  per-event-type recall/precision ground truth.'
when_to_use_negative:
- The goal is to measure runtime and memory scaling on real data — the paper benchmarks
  that separately on 10–90% subsamples of the ~291M-read ASD sample 29, not on simulated
  reads
- You need absolute expression quantification on real biological samples rather than
  controlled comparison of junction-detection accuracy
- Your annotation contains no alternative splicing events, so no held-out truth events
  can be derived via AStalavista
edam_topics:
- http://edamontology.org/topic_3522
- http://edamontology.org/topic_3308
tools:
- name: fortuna
  role: Pseudoaligns simulated reads to a kallisto index of annotated plus guessed-novel
    transcript fragments; produces cnt.tsv signature counts and alt.tsv novel-event
    annotation for scoring against truth events
  repo: https://github.com/canzarlab/fortuna
- name: kallisto
  role: Pseudoalignment engine underlying fortuna's --index/--quant steps on the simulated
    read sets
  repo: https://github.com/pachterlab/kallisto
- name: htslib
  role: Bundled dependency of fortuna for high-throughput sequencing file handling
  repo: https://github.com/samtools/htslib
- name: FluxSimulator
  role: Simulates the 80M paired-end reads from RefSeq release 109 transcripts at
    75 bp and 100 bp
- name: STAR
  role: Genome aligner run in 1-pass and 2-pass modes on the same simulated FASTQs
    as the align-and-count baseline
- name: Whippet
  role: Quantification baseline run on the simulated FASTQs for recall/precision comparison
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:fbb3529e2c688b69dfcb5faafed8c5d4f04535c6d17d31c859bfb047ee645470
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/fortuna
  tools:
  - FluxSimulator
  - STAR
  - Whippet
  - fortuna
  - htslib
  - kallisto
  - samtools
  - sigcount
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3522
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# rna-seq-read-simulation

## Summary

Generates simulated paired-end RNA-seq benchmark data (80M reads, 75/100 bp) with FluxSimulator from RefSeq release 109 transcripts, constructing a partial ground-truth annotation by withholding AStalavista-identified novel junctions, so that recall/precision of splicing tools (fortuna, STAR 1/2-pass, Whippet) can be scored against truth events.

## When to use

Use when benchmarking junction/splicing-event detection accuracy on controlled data: you need simulated FASTQs with known held-out novel junctions and per-event-type recall/precision ground truth.

## When NOT to use

- The goal is to measure runtime and memory scaling on real data — the paper benchmarks that separately on 10–90% subsamples of the ~291M-read ASD sample 29, not on simulated reads
- You need absolute expression quantification on real biological samples rather than controlled comparison of junction-detection accuracy
- Your annotation contains no alternative splicing events, so no held-out truth events can be derived via AStalavista

## Inputs

- Reference transcript annotation in GTF format (RefSeq release 109) for read simulation
- Genome FASTA for index building
- FluxSimulator parameters for 80M paired-end reads at 75 bp and 100 bp read lengths
- AStalavista event calls defining held-out exon skipping / alternative donor / alternative acceptor truth events

## Outputs

- Simulated paired-end FASTQ files per read length (75 bp, 100 bp)
- Partial ground-truth annotation (annotation.rg.gtf) with truth transcripts withheld
- fortuna index per read length plus cnt.tsv (signature counts), alt.tsv (novel event annotation), and unaligned reads
- Per-tool recall and precision by event type, hits-per-read, and runtime comparison table

## How to apply

Simulate 80M paired-end reads with FluxSimulator from RefSeq release 109 transcripts at 75 bp and 100 bp read lengths, then build a partial ground-truth annotation by removing transcripts whose AStalavista-identified exon skipping / alternative donor / alternative acceptor events serve as held-out truth events. Preprocess the partial annotation with fortuna's bin/processGTF.sh to obtain minimal disjoint subexon segments (annotation.rg.gtf), then run fortuna --index per read length (-rl 75 and -rl 100) from the genome FASTA and segmented GTF to generate transcript fragments and a kallisto index. Run fortuna --quant on each simulated FASTQ (-miss 3, multi-threaded) to produce signature counts (cnt.tsv), novel event annotation (alt.tsv), and unaligned reads, alongside STAR 1-pass/2-pass genome alignment and Whippet quantification on the same FASTQs. Score each tool against the withheld AStalavista truth events, computing recall and precision by event type, hits-per-read, and runtime (noting ASGAL's low recall and ~32 h genome-wide runtime).

## Related tools

- **fortuna** (Pseudoaligns simulated reads to a kallisto index of annotated plus guessed-novel transcript fragments; produces cnt.tsv signature counts and alt.tsv novel-event annotation for scoring against truth events) — https://github.com/canzarlab/fortuna
- **kallisto** (Pseudoalignment engine underlying fortuna's --index/--quant steps on the simulated read sets) — https://github.com/pachterlab/kallisto
- **htslib** (Bundled dependency of fortuna for high-throughput sequencing file handling) — https://github.com/samtools/htslib
- **FluxSimulator** (Simulates the 80M paired-end reads from RefSeq release 109 transcripts at 75 bp and 100 bp)
- **STAR** (Genome aligner run in 1-pass and 2-pass modes on the same simulated FASTQs as the align-and-count baseline)
- **Whippet** (Quantification baseline run on the simulated FASTQs for recall/precision comparison)

## Examples

```
bash bin/processGTF.sh /path/to/the/annotation.gtf
```

## Evaluation signals

- The rebuilt benchmark reproduces the Fig. 4 / Supplementary Table S1 pattern: fortuna achieves the highest recall and precision on junction-spanning reads at both 75 bp and 100 bp
- fortuna maps over 99% of the simulated reads (validation threshold stated in the workflow)
- Reported hits-per-read values (~1.02/1.04/1.16) are reproduced within rounding
- Whippet's sensitivity on mismatch-containing reads falls in the reported 68–71% range while detecting the fewest reads
- Scoring is computed only against the withheld AStalavista truth events (exon skipping, alternative donor, alternative acceptor), not the full annotation

## Limitations

- The ground truth is partial: it is derived by removing transcripts from RefSeq release 109 whose AStalavista events are held out, so scoring covers only those withheld event types
- ASGAL comparison is limited by its low recall and ~32 h genome-wide runtime, making full-genome benchmarking impractical
- Hits-per-read exceeds 1 (~1.02–1.16) because pseudoalignment can attribute reads to multiple elementary units, which must be accounted for when comparing tools

## Evidence

- [results] Across the simulated 75 bp and 100 bp datasets, fortuna achieved the highest recall and precision on junction-spanning reads with and without errors: [evidence span withheld — anchor status: not_found]
- [supplementary] Simulate 80M paired-end reads with FluxSimulator from RefSeq release 109 transcripts at 75 bp and 100 bp read lengths: [evidence span withheld — anchor status: not_found]
- [methods] A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites: "A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites"
- [readme] From this set of fragments we build a kallisto [2] index (C) and use it to efficiently pseudoalign reads to fragments (D): "From this set of fragments we build a kallisto [2] index (C) and use it to efficiently pseudoalign reads to fragments (D)"
