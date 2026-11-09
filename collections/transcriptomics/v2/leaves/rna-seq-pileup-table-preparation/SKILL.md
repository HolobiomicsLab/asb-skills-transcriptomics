---
name: rna-seq-pileup-table-preparation
description: Use when sorted RNAseq BAM files must be converted into position-by-position
  variant/invariant nucleotide frequency tables (REDItools outTables, bgzip+tabix
  indexed) to feed REDInet or other RNA editing classifiers.
when_to_use_negative:
- Sites are already provided as a REDItools outTable or a curated ground-truth site
  list — skip pileup generation and run REDInet inference directly on the tabix-indexed
  table
- Analysis targets non-A-to-G events or non-human species — REDItoolDnaRna.py low-stringency
  settings and REDInet's A-to-G filtering (MinAG, MinAGfreq) are tuned for human A-to-I
  editing detection
- Only variant sites without flanking nucleotide frequencies are needed — the low-stringency
  settings deliberately retrieve complete 101-base intervals and are unnecessary overhead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: REDItools
  role: Generates the DNA/RNA pileup outTables from BAM files via REDItoolDnaRna.py
    with low-stringency settings before REDInet inference
  repo: https://github.com/BioinfoUNIBA/REDItools
- name: REDItools3
  role: Faster re-implementation of REDItools for pileup profiling of massive RNAseq
    data (used instead of REDItools v1, which was ~10x slower at ~9 hours)
  repo: https://github.com/BioinfoUNIBA/REDItools3
- name: REDInet
  role: Consumes the tabix-indexed outTables; classifies A-to-G substitutions in 101-base
    windows as A-to-I RNA editing events
  repo: https://github.com/BioinfoUNIBA/REDInet
- name: pysam
  role: Python BAM/SAM interface underlying the REDItools/REDInet pileup computation
  repo: https://github.com/pysam-developers/pysam
- name: samtools
  role: Installed in the REDInet conda environment; faidx-indexes the reference genomes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:d540a8ba565c7789851228925a5168b9a8cca61640fabcc34e0ab2daa6b87286
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
  - EditPredict
  - Python
  - RDDpred
  - RED-ML
  - REDInet
  - REDItools
  - REDItools3
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# rna-seq-pileup-table-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates RNAseq pileup outTables from sorted BAM files using REDItools (v1/v3) under low-stringency settings so that complete 101-base windows are retrievable for REDInet predictions, then bgzip-compresses and tabix-indexes the outTables for downstream inference.

## When to use

Use when sorted RNAseq BAM files must be converted into position-by-position variant/invariant nucleotide frequency tables (REDItools outTables, bgzip+tabix indexed) to feed REDInet or other RNA editing classifiers.

## When NOT to use

- Sites are already provided as a REDItools outTable or a curated ground-truth site list — skip pileup generation and run REDInet inference directly on the tabix-indexed table
- Analysis targets non-A-to-G events or non-human species — REDItoolDnaRna.py low-stringency settings and REDInet's A-to-G filtering (MinAG, MinAGfreq) are tuned for human A-to-I editing detection
- Only variant sites without flanking nucleotide frequencies are needed — the low-stringency settings deliberately retrieve complete 101-base intervals and are unnecessary overhead

## Inputs

- Sorted, coordinate-aligned RNAseq BAM file(s) (e.g. Aligned.sortedByCoord.out.bam)
- Reference genome FASTA (GRCh37/GRCh38 primary assembly, samtools faidx-indexed)
- Installed REDItools (v1 or REDItools3) in a dedicated conda environment

## Outputs

- REDItools outTable(s) listing variant/invariant sites with per-position nucleotide counts
- bgzip-compressed, tabix-indexed outTable.gz files (tabix -s 1 -b 2 -e 2 -c R)
- REDInet inference outputs: filtered position classifications, <.gz name>_discarded.txt for positions filtered on MinCov/MinAG/MinAGfreq

## How to apply

Run REDItools REDItoolDnaRna.py on each BAM against the GRCh37 (or GRCh38) reference FASTA using the low-stringency parameters published in the REDInet README (e.g. -t 40, -c 0,1, -m 0,255, -v 1, -q 0,30, -e, -n 0.0, -N 0.0, -u, -l, -p, -s 2, -g 2, -S); the rationale is to retrieve complete intervals needed for REDInet's 101-base windows rather than only candidate sites. Compress each resulting outTable with bgzip and index with tabix using '-s 1 -b 2 -e 2 -c R' (chromosome, position-start, position-end, reference-base columns). Run REDInet in dedicated conda environments (REDInet env with python 3.9.0, pysam 0.22.1, tensorflow[and-cuda] 2.14.0; a separate env for REDItools). REDInet inference then filters positions on minimum coverage (MinCov, e.g. 50), minimum A-to-G count (MinAG, e.g. 3), and minimum substitution rate (MinAGfreq, e.g. 0.01).

## Related tools

- **REDItools** (Generates the DNA/RNA pileup outTables from BAM files via REDItoolDnaRna.py with low-stringency settings before REDInet inference) — https://github.com/BioinfoUNIBA/REDItools
- **REDItools3** (Faster re-implementation of REDItools for pileup profiling of massive RNAseq data (used instead of REDItools v1, which was ~10x slower at ~9 hours)) — https://github.com/BioinfoUNIBA/REDItools3
- **REDInet** (Consumes the tabix-indexed outTables; classifies A-to-G substitutions in 101-base windows as A-to-I RNA editing events) — https://github.com/BioinfoUNIBA/REDInet
- **pysam** (Python BAM/SAM interface underlying the REDItools/REDInet pileup computation) — https://github.com/pysam-developers/pysam
- **samtools** (Installed in the REDInet conda environment; faidx-indexes the reference genomes)

## Examples

```
bgzip outTable_SRR12492045.SRR12492046; tabix -s 1 -b 2 -e 2 -c R outTable_SRR12492045.SRR12492046.gz
```

## Evaluation signals

- Each outTable.gz exists alongside its .tbi index built with 'tabix -s 1 -b 2 -e 2 -c R'
- REDInet's first-step filter file <.gz name>_discarded.txt is non-empty only if positions failed MinCov (e.g. 50), MinAG (e.g. 3) or MinAGfreq (e.g. 0.01) thresholds — positions passing should appear in prediction outputs
- A-to-G calls should satisfy 'at least 3 Gs or showed a substitution rate >=1%' as in the article's ground-truth protocol
- Runtime sanity: REDItools v3 completes a pileup profile in <60 minutes; REDItools v1 takes ~9 hours — much longer runtimes suggest misconfiguration
- The 101-base window centered on each candidate position must be fully covered by RNAseq reads for REDInet prediction to proceed

## Limitations

- REDInet requires tabix-indexed REDItools output tables; uncompressed or differently indexed outTables are not usable
- Missing nucleotides within the 101-base window cause position filtering, though the multiprocessing REDInet_Inference.py offers genome-build-specific imputation as a BETA feature
- Ran in the article with REDItools v1 settings; REDItools v1 is ~10x slower than REDItools v3 (~9 hours vs <60 minutes) for full pileup profiles
- Two distinct conda environments are recommended (one for REDItools, one for REDInet with python 3.9.0 / tensorflow 2.14.0); mixing installs can break dependencies

## Evidence
