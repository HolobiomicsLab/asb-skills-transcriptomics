---
name: dsrna-positive-training-set-construction
description: Use when building a labeled positive training set of endogenous dsRNA
  regions from A-to-I RNA editing site annotations, and when structural curation via
  RNAfold secondary-structure filters is needed before model training.
when_to_use_negative:
- When constructing the negative (null) dsRNA training set — this skill only defines
  dsRID_train.tsv positives; dsRID_null.tsv comes from a separate step
- When the input already contains curated dsRNA predictions (e.g. data/dsRID_novel_dsRNA.bed
  from a completed dsRID run) rather than raw editing evidence
- When no A-to-I editing annotation source comparable to REDIportal exists for the
  organism, since editing enrichment is the sole positive-signal basis
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: REDIportal
  role: Source of A-to-I RNA editing site annotations used as editing evidence for
    candidate window selection
- name: ViennaRNA RNAfold
  role: Folds extracted candidate region sequences to assess secondary structure and
    derive stem length, mismatch fraction, and adjusted MFE for filtering
- name: dsRID
  role: Pipeline script (dsRID.sh) that runs the training process and emits dsRID_train.tsv
    alongside the negative set and prediction outputs
  repo: https://github.com/gxiaolab/dsRID
- name: Python
  role: Language for the dsRID pipeline scripts (Python 3.x or higher required)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:8e776bd613f95e35e9a4e23f2cceb1baa4208587ccdda4389f3e95478d80e391
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - Python
  - REDIportal
  - ViennaRNA RNAfold
  - dsRID
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# dsrna-positive-training-set-construction

## Summary

Defines the positive training regions (dsRID_train.tsv) for the dsRID random-forest model by mining REDIportal A-to-I RNA editing enrichment as evidence of dsRNA structure, then curating candidates with ViennaRNA RNAfold using stem-length, mismatch, and adjusted-MFE thresholds.

## When to use

Use when building a labeled positive training set of endogenous dsRNA regions from A-to-I RNA editing site annotations, and when structural curation via RNAfold secondary-structure filters is needed before model training.

## When NOT to use

- When constructing the negative (null) dsRNA training set — this skill only defines dsRID_train.tsv positives; dsRID_null.tsv comes from a separate step
- When the input already contains curated dsRNA predictions (e.g. data/dsRID_novel_dsRNA.bed from a completed dsRID run) rather than raw editing evidence
- When no A-to-I editing annotation source comparable to REDIportal exists for the organism, since editing enrichment is the sole positive-signal basis

## Inputs

- REDIportal A-to-I RNA editing site annotations (genomic editing site coordinates)
- Reference genome sequence for extracting candidate region sequences
- PacBio long-read RNA-seq BAM file (context for the dsRID training run)

## Outputs

- dsRID_train.tsv — tab-separated table of positive dsRNA regions with genomic coordinate columns
- dsRID_data.tsv — concatenated positive (dsRID_train) and negative (dsRID_null) regions used for model training

## How to apply

Download REDIportal A-to-I editing site annotations as the source of editing evidence, since clustered A-to-I editing implies double-stranded RNA substrate for ADAR. Slide a 50-bp window across the genome and keep every window with at least 3 editing sites, then merge qualifying windows within 1 kb of each other into candidate editing-enriched regions. Extract each candidate's genomic sequence and fold it with RNAfold; retain candidates only if they have a double-stranded stem of at least 200 bp, no more than 20% mismatches within the stem, and an adjusted minimum free energy (MFE) of at most -0.35. Write surviving regions with their genomic coordinates to dsRID_train.tsv, which becomes the positive half of dsRID_data.tsv (concatenated with the negative set dsRID_null.tsv) used to train the random-forest model in dsRID.

## Related tools

- **REDIportal** (Source of A-to-I RNA editing site annotations used as editing evidence for candidate window selection)
- **ViennaRNA RNAfold** (Folds extracted candidate region sequences to assess secondary structure and derive stem length, mismatch fraction, and adjusted MFE for filtering)
- **dsRID** (Pipeline script (dsRID.sh) that runs the training process and emits dsRID_train.tsv alongside the negative set and prediction outputs) — https://github.com/gxiaolab/dsRID
- **Python** (Language for the dsRID pipeline scripts (Python 3.x or higher required))

## Examples

```
bash dsRID.sh <input_file> <output_dir>
```

## Evaluation signals

- dsRID_train.tsv exists as a tab-separated table with genomic coordinate columns
- Every retained region has a double-stranded stem of at least 200 bp
- Every retained region has no more than 20% mismatches within the stem
- Every retained region has an adjusted MFE of at most -0.35
- All candidate regions derive from merged windows containing at least 3 editing sites in a 50-bp window, merged within 1 kb

## Limitations

- Positive labels depend on REDIportal A-to-I editing coverage; regions lacking detected editing will be missed even if structurally dsRNA
- The 50-bp window / ≥3 editing sites / 1-kb merge and the 200-bp stem, 20% mismatch, −0.35 adjusted MFE thresholds are specific to this pipeline and may not transfer to other organisms or editing densities
- No dsRID or package versions are specified (requirements.txt only), so reproducibility of exact thresholds depends on the repository state at time of use

## Evidence

- [results] Candidate editing-enriched regions were structurally verified with RNAfold and retained as positive dsRNAs only if they had at least 200 bp stem length, up to 20% mismatches, and an adjusted MFE ≤ −0.35.: [evidence span withheld — anchor status: not_found]
- [methods] 50-bp window with at least 3 editing sites: [evidence span withheld — anchor status: not_found]
- [methods] Merge qualifying windows within 1 kb: [evidence span withheld — anchor status: not_found]
- [readme] dsRID_train.tsv as the positive set file: "dsRID_data.tsv is a concatenated file of dsRID_train and dsRID_null each represents positive and negative dsRNA regions."
