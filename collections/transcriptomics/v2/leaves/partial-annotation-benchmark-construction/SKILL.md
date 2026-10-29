---
name: partial-annotation-benchmark-construction
description: 'Use when you must quantify how accurately a splice-aware tool detects
  junctions between annotated splice sites that are absent from the reference annotation:
  simulate reads from full transcriptomes, derive a partial ground truth by removing
  held-out event transcripts, and score recall/precision.'
when_to_use_negative:
- The reference annotation already contains the junctions of interest, so there are
  no held-out novel events to score against — a partial-annotation design would measure
  nothing.
- You need full-assembly evaluation (e.g. novel splice-site discovery from unaligned
  reads), not recall/precision on junction-spanning reads between annotated splice
  sites.
- 'Read lengths other than the indexed ones are used: fortuna indexes are read-length-specific
  (built separately with -rl 75 and -rl 100), so off-target read lengths would require
  new indexes.'
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: fortuna
  role: 'Provides the pipeline under benchmark: processGTF.sh segmentation, --index
    per read length, --quant for signature counts and alt.tsv novel-event annotation;
    also the tool being scored'
  repo: https://github.com/canzarlab/fortuna
- name: kallisto
  role: Bundled pseudoaligner whose equivalence classes underlie fortuna's signature
    counts on the fragment index
  repo: https://github.com/pachterlab/kallisto
- name: htslib
  role: Bundled dependency for reading/writing high-throughput sequencing formats
    within fortuna
  repo: https://github.com/samtools/htslib
- name: STAR
  role: Competitor splice-aware aligner run in 1-pass and 2-pass modes on the same
    simulated FASTQs for recall/precision comparison
- name: Whippet
  role: Competitor quantification tool scored on the same simulated 75/100 bp data
- name: FluxSimulator
  role: Generates the 80M paired-end simulated reads from RefSeq 109 transcripts at
    75 bp and 100 bp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:97b54dbddcedd821e934ec3df412226abc5264ddf926b5f2131d6147bc5971c5
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# partial-annotation-benchmark-construction

## Summary

A benchmarking scheme for novel splice-junction detection in which a 'partial' ground-truth annotation is created by withholding transcripts carrying AStalavista-identified exon skipping / alternative donor / alternative acceptor events, so those events serve as held-out truth against which tools (fortuna, STAR 1/2-pass, Whippet) are scored for recall and precision.

## When to use

Use when you must quantify how accurately a splice-aware tool detects junctions between annotated splice sites that are absent from the reference annotation: simulate reads from full transcriptomes, derive a partial ground truth by removing held-out event transcripts, and score recall/precision per event type.

## When NOT to use

- The reference annotation already contains the junctions of interest, so there are no held-out novel events to score against — a partial-annotation design would measure nothing.
- You need full-assembly evaluation (e.g. novel splice-site discovery from unaligned reads), not recall/precision on junction-spanning reads between annotated splice sites.
- Read lengths other than the indexed ones are used: fortuna indexes are read-length-specific (built separately with -rl 75 and -rl 100), so off-target read lengths would require new indexes.

## Inputs

- RefSeq release 109 transcript annotations (GTF) for genome-wide simulation
- Simulated paired-end FASTQ files at 75 bp and 100 bp read length (~80M reads from FluxSimulator)
- Genome FASTA for index building
- AStalavista-derived event list serving as held-out truth (exon skipping, alternative donor, alternative acceptor)

## Outputs

- Partial ground-truth annotation (GTF with held-out-event transcripts removed)
- Per-tool recall and precision by splicing event type
- Hits-per-read statistics (expected ~1.02 / 1.04 / 1.16)
- Runtime comparison across fortuna, STAR 1-pass/2-pass, Whippet, and ASGAL
- fortuna outputs: cnt.tsv (signature counts), alt.tsv (novel event annotation), unaligned reads for optional refinement

## How to apply

Simulate ~80M paired-end reads (75 bp and 100 bp) with FluxSimulator from RefSeq release 109 transcripts, then derive the partial ground-truth annotation by removing transcripts whose AStalavista-identified exon skipping, alternative donor, or alternative acceptor events become the held-out truth set. Preprocess the partial GTF with fortuna's bin/processGTF.sh to obtain minimal disjoint subexon segments, build a fortuna --index per read length (-rl 75 and -rl 100) from the genome FASTA and segmented GTF, and run fortuna --quant (-miss 3, multi-threaded) to produce signature counts (cnt.tsv) and novel-event annotation (alt.tsv). Run STAR 1-pass, STAR 2-pass, and Whippet on the identical FASTQs as competitors. Score every tool against the withheld AStalavista events, computing recall and precision by event type, hits-per-read, and runtime; treat ASGAL's ~32 h genome-wide runtime and low recall as a documented reference point.

## Related tools

- **fortuna** (Provides the pipeline under benchmark: processGTF.sh segmentation, --index per read length, --quant for signature counts and alt.tsv novel-event annotation; also the tool being scored) — https://github.com/canzarlab/fortuna
- **kallisto** (Bundled pseudoaligner whose equivalence classes underlie fortuna's signature counts on the fragment index) — https://github.com/pachterlab/kallisto
- **htslib** (Bundled dependency for reading/writing high-throughput sequencing formats within fortuna) — https://github.com/samtools/htslib
- **STAR** (Competitor splice-aware aligner run in 1-pass and 2-pass modes on the same simulated FASTQs for recall/precision comparison)
- **Whippet** (Competitor quantification tool scored on the same simulated 75/100 bp data)
- **FluxSimulator** (Generates the 80M paired-end simulated reads from RefSeq 109 transcripts at 75 bp and 100 bp)

## Examples

```
bash bin/processGTF.sh /path/to/the/annotation.gtf && ./fortuna --index -rl 75 genome.fa annotation.rg.gtf
```

## Evaluation signals

- The rebuilt table must reproduce the Fig. 4 / Supplementary Table S1 pattern: fortuna achieves the highest recall and precision on junction-spanning reads across event types at both 75 bp and 100 bp, including on reads with mismatches.
- fortuna maps over 99% of simulated reads (mirrors the README example where it maps over 99% of the STAT1 sample).
- Hits-per-read values match the reported ~1.02 / 1.04 / 1.16 within rounding.
- Whippet sensitivity falls into the 68–71% band on reads containing mismatches as a sanity anchor for the weakest competitor.
- alt.tsv recovers withheld truth events with correct event-type labels (e.g. ES rows such as chr2:190984393-190986854 with >1000 supporting reads in the shipped example).

## Limitations

- The ground truth is partial by design: only AStalavista-classified exon skipping, alternative donor, and alternative acceptor events removed from the annotation can be scored, so events outside these classes are invisible to the benchmark.
- ASGAL exhibits very low recall and ~32 h genome-wide runtime, making it impractical as a genome-wide baseline.
- fortuna indexes are read-length-specific and require the segmented annotation.rg.gtf plus genome FASTA; a wrong read-length setting (e.g. -rl 100 on 75 bp data) invalidates the comparison.

## Evidence

- [results] fortuna achieved the highest recall and precision on junction-spanning reads with and without errors across the simulated 75 bp and 100 bp datasets: [evidence span withheld — anchor status: not_found]
- [results] Whippet sensitivity drop on mismatched reads: [evidence span withheld — anchor status: not_found]
- [methods] Preprocessing rationale for minimal disjoint segments: "A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites"
- [readme] Mapping rate expectation on example reads: "fortuna will map over 99% of those reads and find slightly over 1000 reads supporting a novel exon skipping at chr2:190984393-190986854."
- [abstract] Abstract claim supporting accuracy of mismatched-read mapping across novel junctions: "It mapped reads containing mismatches more accurately across novel junctions and found more reads supporting aberrant splicing events"
