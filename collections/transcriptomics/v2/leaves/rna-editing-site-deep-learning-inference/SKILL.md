---
name: rna-editing-site-deep-learning-inference
description: Use when you have tabix-indexed REDItools outTables of candidate A-to-G
  substitutions from RNAseq BAMs and need to distinguish genuine ADAR-mediated editing
  sites from SNPs or artifacts, especially when matched DNAseq data is unavailable.
when_to_use_negative:
- You need de novo genome-wide RNA editing calling from raw pileups without known
  candidate sites — REDItools v3 or REDItools v1 calling is the appropriate first
  step; REDInet only classifies pre-called A-to-G substitutions
- The 101-base window centered on the candidate site is not fully covered by RNAseq
  reads (unless using the imputation BETA feature); REDInet's accuracy depends on
  complete feature windows
- You need to detect non-A-to-G editing types (e.g. C-to-U) — REDInet is trained exclusively
  on A-to-I events from REDIportal
edam_operation: http://edamontology.org/operation_0487
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: REDInet
  role: Temporal convolutional network classifier that filters candidate sites, extracts
    101-nt feature windows, and outputs y_hat SNP/editing calls
  repo: https://github.com/BioinfoUNIBA/REDInet
- name: REDItools
  role: Generates the candidate A-to-G outTables from RNAseq BAMs at low-stringency
    settings required by REDInet
  repo: https://github.com/BioinfoUNIBA/REDItools
- name: REDItools3
  role: Faster alternative REDItools implementation (<60 min pileup vs ~9 h for v1)
    for producing RNAseq pileup data
  repo: https://github.com/BioinfoUNIBA/REDItools3
- name: pysam
  role: Python BAM/variant-file access library underpinning REDInet's interval extraction
  repo: https://github.com/pysam-developers/pysam
- name: RDDpred
  role: Comparison ML tool for distinguishing sequencing artifacts from genuine AG
    substitutions in benchmarking
  repo: https://github.com/vibbits/RDDpred
- name: RED-ML
  role: 'Comparison tool: logistic regression classifier over three broad feature
    classes, run on the same bona fide HEK293T site set'
  repo: https://github.com/BGIRED/RED-ML
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:2513e4ed04fadd25dcbbf280cec32b03f541e267d789de4d036faa035151c2f3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/BioinfoUNIBA/REDInet
  repo_url: https://github.com/BioinfoUNIBA/REDInet
  tools:
  - Python
  - RDDpred
  - RED-ML
  - REDInet
  - REDItools
  - REDItools3
  - pysam
  edam_operation: http://edamontology.org/operation_0487
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# rna-editing-site-deep-learning-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify A-to-G substitutions in RNAseq data as A-to-I RNA editing events or SNPs using REDInet, a temporal convolutional network (WaveNet-inspired) trained on ~13 million examples from 8404 GTEx RNAseq experiments and REDIportal, without requiring coupled WGS/WES data.

## When to use

Use when you have tabix-indexed REDItools outTables of candidate A-to-G substitutions from RNAseq BAMs and need to distinguish genuine ADAR-mediated editing sites from SNPs or artifacts, especially when matched DNAseq data is unavailable.

## When NOT to use

- You need de novo genome-wide RNA editing calling from raw pileups without known candidate sites — REDItools v3 or REDItools v1 calling is the appropriate first step; REDInet only classifies pre-called A-to-G substitutions
- The 101-base window centered on the candidate site is not fully covered by RNAseq reads (unless using the imputation BETA feature); REDInet's accuracy depends on complete feature windows
- You need to detect non-A-to-G editing types (e.g. C-to-U) — REDInet is trained exclusively on A-to-I events from REDIportal

## Inputs

- Sorted, stranded paired-end RNAseq BAM files (e.g. A549 samples downloaded from SRA)
- REDItools outTable files listing candidate A-to-G substitutions, bgzip-compressed and tabix-indexed (-s 1 -b 2 -e 2 -c R)
- Reference genome FASTA (GRCh37 or GRCh38 primary assembly from GENCODE release 46) with samtools faidx index
- Pretrained REDInet.h5 temporal convolutional network model (bundled in the REDInet package)

## Outputs

- Per-site predictions in *.predictions.tsv with y_hat calls (0 = SNP, 1 = editing site)
- <outTable name>_discarded.txt files recording positions filtered out at the coverage/substitution-rate/G-count step

## How to apply

First generate candidate sites by running REDItools REDItoolDnaRna.py on sorted RNAseq BAMs at low stringency (e.g. -q 0,30 -c 0,1 -v 1 -n 0.0 -N 0.0) so that complete 101-nt intervals are retrievable, then compress each outTable with bgzip and index with 'tabix -s 1 -b 2 -e 2 -c R'. Launch REDInet_Inference.py (multiprocessing, supports missing-value imputation via the reference genome) or REDInet_Inference_light_ver.py (HTC-friendly) against the indexed outTables with the appropriate assembly (GRCh37 or GRCh38) and default thresholds: minimum coverage 50 reads, minimum 3 G substitutions, minimum A-to-G frequency 0.01. REDInet filters positions in three steps (coverage/AG-rate/G-count filtering, missing-nucleotide filtering over the 101-base window, then classification), encodes each 101-base window as an 8 x 101 feature matrix of RNAseq nucleotide frequencies, and scores it with the pretrained REDInet.h5 TCN model. Collect per-site y_hat calls from the *.predictions.tsv output where 0 = SNP and 1 = editing site. Validate performance against an independent ground truth (e.g. ADAR WT vs silenced/knockout contrasts), expecting >95-96% accuracy as reported on A549 and HEK293T benchmark sets.

## Related tools

- **REDInet** (Temporal convolutional network classifier that filters candidate sites, extracts 101-nt feature windows, and outputs y_hat SNP/editing calls) — https://github.com/BioinfoUNIBA/REDInet
- **REDItools** (Generates the candidate A-to-G outTables from RNAseq BAMs at low-stringency settings required by REDInet) — https://github.com/BioinfoUNIBA/REDItools
- **REDItools3** (Faster alternative REDItools implementation (<60 min pileup vs ~9 h for v1) for producing RNAseq pileup data) — https://github.com/BioinfoUNIBA/REDItools3
- **pysam** (Python BAM/variant-file access library underpinning REDInet's interval extraction) — https://github.com/pysam-developers/pysam
- **RDDpred** (Comparison ML tool for distinguishing sequencing artifacts from genuine AG substitutions in benchmarking) — https://github.com/vibbits/RDDpred
- **RED-ML** (Comparison tool: logistic regression classifier over three broad feature classes, run on the same bona fide HEK293T site set) — https://github.com/BGIRED/RED-ML

## Examples

```
python3 REDInet_Inference.py --I <outTable_folder> --O <output_folder> --A GRCh37 --C 50 --M 3 --N <outTable_names>
```

## Evaluation signals

- On the A549 ground truth set (>13,000 sites, WT vs ADAR-silenced), mean classification accuracy must reach the reported >96% (Fig. 2b, Table S2)
- On HEK293T KO-WT and KO-OE ground truths, expect accuracy >95%, specificity >93%, precision >96%
- Discarded-site files (<outTable name>_discarded.txt) should be empty if no position fails the minimum coverage 50 / 3 G / 0.01 frequency thresholds
- Biological invariant: fraction of predicted positives in Alu adenosines should rise from ~5% in ADAR KO to ~95% in ADAR overexpression (OE) HEK293T samples
- Training/validation/test sanity check: the pretrained model should show >99% accuracy without under- or over-fitting

## Limitations

- Complete REDInet architecture description and feature rescaling details are deferred to Supplementary Material not included in the main text
- Missing-nucleotide imputation using the reference genome is a BETA feature in the multiprocessing version
- Performance evaluation metric formulas are only in the Supplementary Methods; ground truth datasets are referenced as being in 'our GitHub repository' without a specific URL
- REDInet was trained on GTEx/REDIportal data; benchmarks note comparisons to EditPredict (0.78), RED-ML, and RDDpred trained on DARNED/RADAR sites, so cross-tool score scales are not directly comparable

## Evidence
