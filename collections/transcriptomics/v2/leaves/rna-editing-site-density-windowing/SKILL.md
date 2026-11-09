---
name: rna-editing-site-density-windowing
description: Use when constructing a positive training set of double-stranded RNA
  regions from A-to-I RNA editing evidence — i.e., when converting REDIportal editing-site
  annotations into structurally verified dsRNA coordinates for model training.
when_to_use_negative:
- When RNA editing calls are the analysis goal themselves — this skill uses editing
  sites only as a proxy signal to define dsRNA training regions, not to call or validate
  editing sites
- When you want dsRID's dsRNA predictions on new sequencing data rather than training-set
  construction — use the dsRID pipeline (dsRID.sh, extract_whole.py, model_predict.py)
  directly instead
- 'When short-read RNA-seq is the only input: this skill''s positive set is tied to
  the dsRID design that trains on PacBio long-read RNA-seq data'
edam_topics:
- http://edamontology.org/topic_0097
- http://edamontology.org/topic_0099
tools:
- name: REDIportal
  role: Source of A-to-I RNA editing site annotations used as the editing-enrichment
    evidence for sliding-window candidate definition
- name: ViennaRNA RNAfold
  role: Folds extracted candidate region sequences to compute secondary structure,
    enabling the stem >= 200 bp, mismatch <= 20%, and adjusted MFE <= -0.35 filters
- name: dsRID
  role: Hosts the training workflow whose positive set (dsRID_train.tsv) this skill
    produces; consumed by dsRID.sh training
  repo: https://github.com/gxiaolab/dsRID
- name: Python
  role: Implementation language for the windowing, merging, sequence extraction, and
    filtering steps
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:c125b250cbd1b32c38a95226629c1c9eb15fdf0b1cae6dc1f2781768e820cfda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - Python
  - REDIportal
  - ViennaRNA RNAfold
  - dsRID
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0097
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# rna-editing-site-density-windowing

## Summary

Builds the positive dsRNA training set for dsRID by sliding a 50-bp window over REDIportal A-to-I RNA editing sites, merging site-dense windows within 1 kb into candidate regions, and curating them with ViennaRNA RNAfold using stem length, mismatch, and adjusted MFE thresholds.

## When to use

Use when constructing a positive training set of double-stranded RNA regions from A-to-I RNA editing evidence — i.e., when converting REDIportal editing-site annotations into structurally verified dsRNA coordinates for model training.

## When NOT to use

- When RNA editing calls are the analysis goal themselves — this skill uses editing sites only as a proxy signal to define dsRNA training regions, not to call or validate editing sites
- When you want dsRID's dsRNA predictions on new sequencing data rather than training-set construction — use the dsRID pipeline (dsRID.sh, extract_whole.py, model_predict.py) directly instead
- When short-read RNA-seq is the only input: this skill's positive set is tied to the dsRID design that trains on PacBio long-read RNA-seq data

## Inputs

- REDIportal A-to-I RNA editing site annotations (genomic coordinates of editing sites)
- Reference genome sequence for extracting candidate region sequences for RNAfold folding

## Outputs

- dsRID_train.tsv — tab-separated table of positive dsRNA regions with genomic coordinate columns, each satisfying stem >= 200 bp, mismatch <= 20%, and adjusted MFE <= -0.35

## How to apply

Download REDIportal A-to-I editing site annotations as the source of editing evidence, then slide a 50-bp window across the genome and retain every window containing at least 3 editing sites. Merge qualifying windows that lie within 1 kb of each other into candidate editing-enriched regions, since dsRNA stems typically span multiple short editing-dense windows. Extract each candidate region's genomic sequence and fold it with ViennaRNA RNAfold to assess secondary structure. Retain candidates only if they have a double-stranded stem of at least 200 bp, no more than 20% mismatches within the stem, and an adjusted minimum free energy (MFE) of at most -0.35; write surviving regions with genomic coordinates to dsRID_train.tsv as the positive training set.

## Related tools

- **REDIportal** (Source of A-to-I RNA editing site annotations used as the editing-enrichment evidence for sliding-window candidate definition)
- **ViennaRNA RNAfold** (Folds extracted candidate region sequences to compute secondary structure, enabling the stem >= 200 bp, mismatch <= 20%, and adjusted MFE <= -0.35 filters)
- **dsRID** (Hosts the training workflow whose positive set (dsRID_train.tsv) this skill produces; consumed by dsRID.sh training) — https://github.com/gxiaolab/dsRID
- **Python** (Implementation language for the windowing, merging, sequence extraction, and filtering steps)

## Examples

```
bash dsRID.sh <input_file> <output_dir>
```

## Evaluation signals

- dsRID_train.tsv exists as a tab-separated table with genomic coordinate columns
- Every retained region has a double-stranded stem of at least 200 bp
- Every retained region has no more than 20% mismatches within its stem
- Every retained region has an adjusted MFE of at most -0.35
- Each retained region can be traced back to at least one qualifying 50-bp window with >= 3 editing sites, with merged windows spaced within 1 kb

## Limitations

- Only A-to-I RNA editing sites from REDIportal are used as the positive signal, so dsRNA regions lacking editing enrichment will not enter the positive set
- The stem >= 200 bp, mismatch <= 20%, and adjusted MFE <= -0.35 thresholds are fixed; candidate dsRNAs with weaker structure or more mismatches are excluded by design
- No specific dsRID or package versions are stated in the methods (requirements listed only in requirements.txt), so environment reproducibility is limited

## Evidence

- [other] Candidate editing-enriched regions were structurally verified with RNAfold and retained as positive dsRNAs only if they had at least 200 bp stem length, up to 20% mismatches, and an adjusted MFE ≤ −0.35.: [evidence span withheld — anchor status: not_found]
- [readme] Positive set output is dsRID_train.tsv, produced by the dsRID.sh training process.: "Run the dsRID script to include the training process with the following command: bash dsRID.sh <input_file> <output_dir>"
- [readme] dsRID_train.tsv is confirmed as the positive dsRNA region file within the dsRID output schema.: "dsRID_data.tsv is a concatenated file of dsRID_train and dsRID_null each represents positive and negative dsRNA regions."
- [abstract] dsRID itself is a machine-learning method predicting dsRNA regions from long-read RNA-seq, contextualizing why the positive set is needed.: "we report dsRID (double-stranded RNA identifier), a machine-learning-based method to predict dsRNA regions in silico"
