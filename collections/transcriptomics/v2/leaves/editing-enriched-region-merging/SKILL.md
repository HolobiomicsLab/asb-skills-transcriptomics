---
name: editing-enriched-region-merging
description: Use when A-to-I RNA editing site annotations (e.g., REDIportal) must
  be converted into candidate dsRNA training regions via windowed enrichment detection,
  spatial merging, and RNAfold secondary-structure filtering.
when_to_use_negative:
- You already have experimentally verified dsRNA annotations and do not need to infer
  positives from RNA editing enrichment
- Your editing evidence is not A-to-I editing from REDIportal — the window-count thresholds
  (≥3 sites per 50-bp window) were defined for REDIportal A-to-I annotations
- You need genome-wide dsRNA prediction from long-read RNA-seq reads rather than construction
  of a positive training set (use the full dsRID prediction workflow with extract_whole.py
  and model_predict.py)
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: REDIportal
  role: Source of A-to-I RNA editing site annotations used as the editing evidence
    for windowed enrichment detection
- name: ViennaRNA RNAfold
  role: Folds extracted candidate region sequences to assess secondary structure and
    compute MFE-based filtering metrics
- name: dsRID
  role: Consumes the resulting dsRID_train.tsv positives (concatenated with dsRID_null
    negatives) as dsRID_data.tsv for random forest training
  repo: https://github.com/gxiaolab/dsRID
- name: Python
  role: Scripting language (Python 3.x) for implementing the windowing, merging, and
    filtering steps
  repo: https://github.com/gxiaolab/dsRID
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:23e59cbfcdecb37bd4069a6513dee074f36020de46196ee7da85d64d7d7448b5
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# editing-enriched-region-merging

## Summary

Reconstructs the EER-based positive dsRNA annotation pipeline used to train dsRID: A-to-I RNA editing sites from REDIportal are windowed over the genome, merged into candidate editing-enriched regions, and retained as positive dsRNAs only after RNAfold structural curation.

## When to use

Use when A-to-I RNA editing site annotations (e.g., REDIportal) must be converted into candidate dsRNA training regions via windowed enrichment detection, spatial merging, and RNAfold secondary-structure filtering.

## When NOT to use

- You already have experimentally verified dsRNA annotations and do not need to infer positives from RNA editing enrichment
- Your editing evidence is not A-to-I editing from REDIportal — the window-count thresholds (≥3 sites per 50-bp window) were defined for REDIportal A-to-I annotations
- You need genome-wide dsRNA prediction from long-read RNA-seq reads rather than construction of a positive training set (use the full dsRID prediction workflow with extract_whole.py and model_predict.py)

## Inputs

- REDIportal A-to-I RNA editing site annotations
- Reference genome sequence for extracting candidate region sequences

## Outputs

- dsRID_train.tsv — tab-separated positive training set with genomic coordinate columns for each retained editing-enriched region
- Candidate editing-enriched regions with RNAfold structural metrics (stem length, mismatch fraction, adjusted MFE)

## How to apply

Download REDIportal A-to-I RNA editing site annotations as the source of editing evidence, then slide a 50-bp window across the genome and retain every window containing at least 3 editing sites. Merge qualifying windows that lie within 1 kb of each other into candidate editing-enriched regions, since nearby editing clusters reflect the hyperediting characteristic of long dsRNA stems. Extract the genomic sequence of each candidate region and fold it with ViennaRNA RNAfold to assess secondary structure. Filter candidates to those with a double-stranded stem of at least 200 bp, no more than 20% mismatches within the stem, and an adjusted minimum free energy (MFE) of at most -0.35; write the surviving regions with their genomic coordinates as the positive training set to dsRID_train.tsv. These thresholds ensure only structurally plausible, editing-enriched duplexes enter model training, distinguishing positives from the dsRID_null negative set used to build dsRID_data.tsv.

## Related tools

- **REDIportal** (Source of A-to-I RNA editing site annotations used as the editing evidence for windowed enrichment detection)
- **ViennaRNA RNAfold** (Folds extracted candidate region sequences to assess secondary structure and compute MFE-based filtering metrics)
- **dsRID** (Consumes the resulting dsRID_train.tsv positives (concatenated with dsRID_null negatives) as dsRID_data.tsv for random forest training) — https://github.com/gxiaolab/dsRID
- **Python** (Scripting language (Python 3.x) for implementing the windowing, merging, and filtering steps) — https://github.com/gxiaolab/dsRID

## Examples

```
bash dsRID.sh <input_file> <output_dir>
```

## Evaluation signals

- dsRID_train.tsv exists as a tab-separated table with genomic coordinate columns
- Every retained region satisfies stem >= 200 bp
- Every retained region has mismatch <= 20% within the stem
- Every retained region has adjusted MFE <= -0.35
- No region originated from a 50-bp window containing fewer than 3 editing sites, and all merged windows were within 1 kb of each other

## Limitations

- No version of dsRID or specific package versions are stated in the repository (required packages are only listed in requirements.txt)
- The thresholds (50-bp window, >=3 editing sites, 1-kb merge distance, 200-bp stem, 20% mismatches, adjusted MFE <= -0.35) are specific to this paper's positive-set construction and may need re-tuning for other editing databases or genomes
- Positives are inferred from editing enrichment plus RNAfold curation, not directly validated as structured dsRNAs, so false positives from unstructured editing-dense loci are possible

## Evidence

- [other] Sliding window editing enrichment with 50-bp window and >=3 sites: [evidence span withheld — anchor status: not_found]
- [other] Merging of qualifying windows within 1 kb: [evidence span withheld — anchor status: not_found]
- [other] Structural filter thresholds for positive dsRNA retention: [evidence span withheld — anchor status: not_found]
- [readme] Role of dsRID_train.tsv positives in the dsRID training data: "dsRID_data.tsv is a concatenated file of dsRID_train and dsRID_null each represents positive and negative dsRNA regions."
