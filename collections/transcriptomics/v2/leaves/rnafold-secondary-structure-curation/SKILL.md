---
name: rnafold-secondary-structure-curation
description: Use when candidate dsRNA/editing-enriched regions with genomic coordinates
  need structural validation or classification (e.g., stem >= 200 bp, <= 20% mismatches,
  adjusted MFE <= -0.35) before use as training labels or for final dsRNA annotation.
when_to_use_negative:
- Input is raw PacBio BAM reads rather than candidate regions with coordinates — extract_whole.py
  and model_predict.py must run first to define candidates
- Regions already pass structural filters and only need downstream model training
  or prediction
- Analysis targets something other than double-stranded RNA secondary structure (e.g.,
  protein-level or expression quantification tasks)
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0099
tools:
- name: ViennaRNA RNAfold
  role: Folds the genomic sequence of each candidate region to assess secondary structure
    and compute minimum free energy for filtering
- name: Python
  role: Scripting the candidate-region sequence extraction, RNAfold invocation, and
    threshold filtering into dsRID_train.tsv
  repo: https://github.com/gxiaolab/dsRID
- name: dsRID
  role: Pipeline whose candidate regions and training set (dsRID_train.tsv) are produced
    and curated by this RNAfold step
  repo: https://github.com/gxiaolab/dsRID
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:218a02425bec9cd4f733c9dcd5355c1d4e56f52d15f078559c28b025cff7ecef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - Python
  - ViennaRNA RNAfold
  - dsRID
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# rnafold-secondary-structure-curation

## Summary

Structurally curate candidate dsRNA regions by folding their genomic sequences with ViennaRNA RNAfold and filtering on stem length, mismatch rate, and adjusted minimum free energy (MFE). In dsRID this step both validates the EER-derived positive training set and classifies novel long dsRNAs vs. structured RNAs among genome-wide candidates.

## When to use

Use when candidate dsRNA/editing-enriched regions with genomic coordinates need structural validation or classification (e.g., stem >= 200 bp, <= 20% mismatches, adjusted MFE <= -0.35) before use as training labels or for final dsRNA annotation.

## When NOT to use

- Input is raw PacBio BAM reads rather than candidate regions with coordinates — extract_whole.py and model_predict.py must run first to define candidates
- Regions already pass structural filters and only need downstream model training or prediction
- Analysis targets something other than double-stranded RNA secondary structure (e.g., protein-level or expression quantification tasks)

## Inputs

- Genomic coordinates (BED/TSV) of candidate dsRNA or editing-enriched regions
- Reference genome sequences from which candidate region sequences are extracted
- REDIportal A-to-I RNA editing site annotations (for EER-based positive-set construction)

## Outputs

- dsRID_train.tsv — tab-separated table of positive dsRNA regions with genomic coordinate columns, each passing stem >= 200 bp, mismatch <= 20%, adjusted MFE <= -0.35
- Structural classification of genome-wide candidate dsRNA regions into novel long dsRNAs and structured RNAs (contributing to data/dsRID_novel_dsRNA.bed)

## How to apply

Extract the genomic sequence for each candidate region (e.g., EER regions defined by sliding a 50-bp window with >= 3 A-to-I editing sites from REDIportal and merging windows within 1 kb), then fold each sequence with RNAfold. Retain a region as a positive dsRNA only if it forms a double-stranded stem of at least 200 bp with no more than 20% mismatches within the stem and an adjusted minimum free energy (MFE) of at most -0.35. Write surviving regions with their genomic coordinates to dsRID_train.tsv as the positive training set. In the genome-wide application, apply the same RNAfold curation to dsRID candidate regions (predicted probability > 0.5, non-overlapping with EER dsRNAs) to classify them into novel long dsRNAs (32,391) versus structured RNAs (49,875). Validation is count-based: the candidate, novel-long-dsRNA, and structured-RNA totals must match the paper's reported values (82,266 / 32,391 / 49,875, Fig. 3A).

## Related tools

- **ViennaRNA RNAfold** (Folds the genomic sequence of each candidate region to assess secondary structure and compute minimum free energy for filtering)
- **Python** (Scripting the candidate-region sequence extraction, RNAfold invocation, and threshold filtering into dsRID_train.tsv) — https://github.com/gxiaolab/dsRID
- **dsRID** (Pipeline whose candidate regions and training set (dsRID_train.tsv) are produced and curated by this RNAfold step) — https://github.com/gxiaolab/dsRID

## Examples

```
RNAfold --noPS < candidate_region.fa
```

## Evaluation signals

- Every region in dsRID_train.tsv satisfies stem >= 200 bp, mismatch <= 20%, and adjusted MFE <= -0.35
- dsRID_train.tsv exists as a tab-separated table with genomic coordinate columns
- Genome-wide classification totals match the paper: 82,266 candidate regions / 32,391 novel long dsRNAs / 49,875 structured RNAs (Fig. 3A)
- Novel long dsRNAs exclude any windows overlapping EER dsRNAs and have predicted probability > 0.5 from the random-forest model

## Limitations

- Thresholds (200 bp stem, 20% mismatches, adjusted MFE <= -0.35) were calibrated for the dsRID EER training workflow; other structural criteria would change the positive set
- No specific RNAfold or ViennaRNA version is stated in the article or README, so exact MFE values may vary across tool versions
- Classification depends on upstream candidate quality: windows must come from 2500-nt scans with >= 6 supporting reads including >= 1 skipping read and model probability > 0.5

## Evidence

- [methods] retained as positive dsRNAs only if they had at least 200 bp stem length, up to 20% mismatches, and an adjusted MFE ≤ −0.35: [evidence span withheld — anchor status: not_found]
- [results] 32,391 were categorized as novel long dsRNAs based on RNAfold, and 49,875 were denoted as structured RNAs: [evidence span withheld — anchor status: not_found]
- [readme] dsRID_whole will contain column of predicted probability to be a candidate dsRNA region from the random forest model trained on dsRID_data.tsv: "dsRID_whole will contain column of predicted probability to be a candidate dsRNA region from the random forest model trained on dsRID_data.tsv."
