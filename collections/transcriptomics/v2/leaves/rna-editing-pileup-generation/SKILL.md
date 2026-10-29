---
name: rna-editing-pileup-generation
description: Use when REDInet inference needs tabix-indexed REDItools outTables from
  coordinate-sorted RNAseq BAMs, or when benchmarking REDItools v1 vs v3 pileup runtime
  on very deep HEK293T libraries.
when_to_use_negative:
- Input is already a tabix-indexed REDItools outTable — go directly to REDInet_Inference.py
  / REDInet_Inference_light_ver.py instead of re-running pileup
- The goal is classifying candidate A-to-G substitutions as RNA editing or artifacts
  — that is REDInet (TCN classifier) inference, not pileup generation
- The analysis requires WGS/WES-coupled empirical filtering or ML comparison via RED-ML/RDDpred
  guidelines rather than plain RNAseq pileup profiles
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: REDItools (v1)
  role: Reference implementation run via REDItoolDnaRna.py with low-stringency flags
    to produce complete outTables for REDInet; the slower (~9 h) benchmark arm
  repo: https://github.com/BioinfoUNIBA/REDItools
- name: REDItools3
  role: Re-implemented, faster pileup engine (<60 min on 277M-read BAMs) installed
    via 'pip install REDItools3' and run as 'python -m reditools analyze'
  repo: https://github.com/BioinfoUNIBA/REDItools3
- name: Samtools
  role: Generates the .fai index ('samtools faidx') required on the reference FASTA
    before pileup
- name: Tabix
  role: Indexes the bgzipped outTable (-s 1 -b 2 -e 2 -c R) so REDInet can consume
    it
- name: pysam
  role: Python SAM/BAM wrapper underlying the REDItools3 re-implementation (PySAM
    C libraries)
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:c736741d10ca6d33e456c1ba81384f04118d7207de0f0369dc008c81f1f610e0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/BioinfoUNIBA/REDInet
  repo_url: https://github.com/BioinfoUNIBA/REDItools
  tools:
  - Python
  - REDInet
  - REDItools (v1)
  - REDItools3
  - Samtools
  - Tabix
  - pysam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# rna-editing-pileup-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate complete genome-wide RNAseq pileup profiles (REDItools outTables) from coordinate-sorted RNAseq BAMs using REDItools v1 (REDItoolDnaRna.py) or the re-implemented REDItools3, choosing v3 when wall-clock time matters on deep libraries (~277M reads/sample).

## When to use

Use when REDInet inference needs tabix-indexed REDItools outTables from coordinate-sorted RNAseq BAMs, or when benchmarking REDItools v1 vs v3 pileup runtime on very deep HEK293T libraries.

## When NOT to use

- Input is already a tabix-indexed REDItools outTable — go directly to REDInet_Inference.py / REDInet_Inference_light_ver.py instead of re-running pileup
- The goal is classifying candidate A-to-G substitutions as RNA editing or artifacts — that is REDInet (TCN classifier) inference, not pileup generation
- The analysis requires WGS/WES-coupled empirical filtering or ML comparison via RED-ML/RDDpred guidelines rather than plain RNAseq pileup profiles

## Inputs

- Coordinate-sorted paired-end stranded RNAseq BAM (e.g. HEK293T KO samples SRR12492027/SRR12492028, ~277M aligned reads per BAM; 69.5M reads/sample for the public WT/ADAR KO comparison)
- Reference genome FASTA (GRCh37 or GRCh38 primary assembly) indexed with 'samtools faidx'
- Dedicated conda environments for REDItools v1 and REDItools3

## Outputs

- REDItools outTable per BAM (position-wise variant/invariant RNAseq sites with region, position, reference base, coverage, meanQ, BaseCount[A,C,G,T], AllSubs, Frequency)
- bgzip-compressed, tabix-indexed outTable (tabix -s 1 -b 2 -e 2 -c R) consumable by REDInet inference
- Benchmark table of recorded wall-clock runtimes (e.g. reditools_runtime_benchmark.tsv) with the v1/v3 runtime ratio (~10x)

## How to apply

Run REDItools on a coordinate-sorted paired-end RNAseq BAM against a samtools-faidx-indexed GRCh37/GRCh38 primary assembly FASTA using the low-stringency setting (-t 40 threads, -c 0,1, -m 0,255, -v 1, -q 0,30, -e, -n 0.0, -N 0.0, -u, -l, -p, -s 2, -g 2, -S) so that complete 101-base windows needed for REDInet predictions are retrieved. REDItools traverses the reference genome position by position, listing variant and invariant RNAseq-supported sites into an outTable. Compress the outTable with bgzip and index it with 'tabix -s 1 -b 2 -e 2 -c R' before REDInet consumption. For the v1-vs-v3 benchmark, run both tools in dedicated conda environments on the identical BAM (e.g. SRR12492027/SRR12492028 HEK293T KO data), record wall-clock time, and compute the runtime ratio: v3 completes in <60 minutes while v1 takes ~9 hours (~10x slower).

## Related tools

- **REDItools (v1)** (Reference implementation run via REDItoolDnaRna.py with low-stringency flags to produce complete outTables for REDInet; the slower (~9 h) benchmark arm) — https://github.com/BioinfoUNIBA/REDItools
- **REDItools3** (Re-implemented, faster pileup engine (<60 min on 277M-read BAMs) installed via 'pip install REDItools3' and run as 'python -m reditools analyze') — https://github.com/BioinfoUNIBA/REDItools3
- **Samtools** (Generates the .fai index ('samtools faidx') required on the reference FASTA before pileup)
- **Tabix** (Indexes the bgzipped outTable (-s 1 -b 2 -e 2 -c R) so REDInet can consume it)
- **pysam** (Python SAM/BAM wrapper underlying the REDItools3 re-implementation (PySAM C libraries)) — https://github.com/pysam-developers/pysam

## Examples

```
python REDItoolDnaRna.py -o Data -i SRR12492027.SRR12492028.Aligned.sortedByCoord.out.chr10.bam -f GRCh37.primary_assembly.genome.fa -t 40 -c 0,1 -m 0,255 -v 1 -q 0,30 -e -n 0.0 -N 0.0 -u -l -p -s 2 -g 2 -S
```

## Evaluation signals

- bgzip+tabix indexing succeeds without errors ('tabix -s 1 -b 2 -e 2 -c R outTable_*.gz'), confirming a valid, REDInet-consumable outTable
- Benchmark acceptance: REDItools3 finishes the full pileup in <60 minutes while REDItools v1 requires roughly 9 hours on the same 277M-read jobs (~10x ratio)
- outTable rows span variant AND invariant positions genome-wide (low-stringency run, -m 0,255, -n 0.0, -N 0.0), not only variant sites
- REDInet inference runs to completion on the indexed outTable with GRCh37/GRCh38 assembly specified; discarded/filtered-position files are produced as expected
- Equivalent settings are used for both tools so the runtime comparison is apples-to-apples

## Limitations

- REDInet's full-coverage 101-base window requirement means holes in pileup coverage force either site filtering or (BETA) imputation of missing values via the multiprocessing inference script
- Runtime scales with library depth: the ~10x v1/v3 gap was measured on ~277M-read HEK293T KO BAMs with 40 threads; smaller libraries will not show the same absolute times
- REDInet runs on tabix-indexed outTables only — unindexed or non-bgzip outTables cannot be consumed
- The paper's timing claims come from GRCh37-mapped HEK293T/A549 test cases; other genome builds or alignments may need re-benchmarking

## Evidence
