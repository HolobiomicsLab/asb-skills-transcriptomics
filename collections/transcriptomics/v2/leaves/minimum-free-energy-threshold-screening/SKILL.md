---
name: minimum-free-energy-threshold-screening
description: Use when a set of candidate genomic regions (e.g., editing-enriched windows)
  must be filtered for bona fide double-stranded RNA secondary structure before use
  as training positives or confident annotations.
when_to_use_negative:
- Candidates are already validated dsRNAs (e.g., taken from a curated database) and
  only need format conversion or downstream modeling
- The goal is predicting dsRNA from sequencing read features alone (dsRID's random-forest
  prediction path on BAM-derived features), not validating editing-enriched candidates
  by structure
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0099
tools:
- name: ViennaRNA RNAfold
  role: Folds each candidate region's genomic sequence to compute the secondary structure
    from which stem length, mismatches, and adjusted MFE are derived
- name: dsRID
  role: Pipeline in which the MFE/structure filter defines the positive training regions
    (dsRID_train.tsv)
  repo: https://github.com/gxiaolab/dsRID
- name: Python 3.x
  role: Implements the sliding-window scan, candidate merging, RNAfold invocation,
    and threshold filtering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:0b8b0341f2dee5183cf2d8e9b84502fe25ef92558e8c6eace9897610a8478949
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - Python
  - Python 3.x
  - ViennaRNA RNAfold
  - dsRID
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# minimum-free-energy-threshold-screening

## Summary

Screens candidate RNA regions for double-stranded character by folding extracted genomic sequence with ViennaRNA RNAfold and retaining only regions whose adjusted minimum free energy (MFE) is at most -0.35, alongside stem-length and mismatch thresholds. In dsRID this converts REDIportal-derived editing-enriched candidates into a validated positive dsRNA training set.

## When to use

Use when a set of candidate genomic regions (e.g., editing-enriched windows) must be filtered for bona fide double-stranded RNA secondary structure before use as training positives or confident annotations.

## When NOT to use

- Candidates are already validated dsRNAs (e.g., taken from a curated database) and only need format conversion or downstream modeling
- The goal is predicting dsRNA from sequencing read features alone (dsRID's random-forest prediction path on BAM-derived features), not validating editing-enriched candidates by structure

## Inputs

- Candidate editing-enriched genomic regions (merged 50-bp windows containing >=3 REDIportal A-to-I editing sites, merged within 1 kb)
- Reference genome sequence from which candidate region sequences are extracted

## Outputs

- Filtered positive dsRNA training set with genomic coordinates (dsRID_train.tsv, tab-separated)
- Per-region RNAfold secondary-structure metrics: stem length, mismatch fraction, adjusted MFE

## How to apply

Extract the genomic sequence of each candidate region and fold it with ViennaRNA RNAfold to assess secondary structure. Compute the double-stranded stem length, the fraction of mismatches within the stem, and an adjusted minimum free energy (MFE) normalized for region length. Retain a candidate as a positive dsRNA only if it has a stem of at least 200 bp, no more than 20% mismatches within the stem, and an adjusted MFE of at most -0.35. Write surviving regions with their genomic coordinates as the positive training set (in dsRID this is dsRID_train.tsv, a tab-separated table with genomic coordinate columns). The rationale is that A-to-I editing sites cluster in structured dsRNA substrates, so structural folding criteria independent of the editing signal validate that candidates are true dsRNAs rather than coincidental editing clusters.

## Related tools

- **ViennaRNA RNAfold** (Folds each candidate region's genomic sequence to compute the secondary structure from which stem length, mismatches, and adjusted MFE are derived)
- **dsRID** (Pipeline in which the MFE/structure filter defines the positive training regions (dsRID_train.tsv)) — https://github.com/gxiaolab/dsRID
- **Python 3.x** (Implements the sliding-window scan, candidate merging, RNAfold invocation, and threshold filtering)

## Examples

```
RNAfold < candidate_region.fa  # then filter: stem_len >= 200 and mismatch_pct <= 20 and adjusted_MFE <= -0.35 -> dsRID_train.tsv
```

## Evaluation signals

- dsRID_train.tsv exists as a tab-separated table with genomic coordinate columns
- Every retained region satisfies stem >= 200 bp
- Every retained region has mismatch fraction <= 20% within the stem
- Every retained region has adjusted MFE <= -0.35
- No candidate lacking at least 3 A-to-I editing sites in a 50-bp window (pre-merge) survives into the training set

## Limitations

- No specific version of dsRID or its required packages is stated in the methods; only that requirements are listed in requirements.txt
- Thresholds (200 bp stem, 20% mismatches, adjusted MFE <= -0.35) were calibrated on PacBio long-read RNA-seq from Alzheimer's disease brain and may not transfer to other tissues or editing regimes
- Regions lacking RNA editing signal (no >=3 editing sites per 50-bp window) are never considered, so the filter is scoped to editing-enriched candidates rather than a genome-wide structural scan

## Evidence

- [other] Candidate editing-enriched regions were structurally verified with RNAfold and retained as positive dsRNAs only if they had at least 200 bp stem length, up to 20% mismatches, and an adjusted MFE ≤ −0.35.: [evidence span withheld — anchor status: not_found]
- [other] Extract the genomic sequence of each candidate region and fold it with RNAfold to assess secondary structure.: [evidence span withheld — anchor status: not_found]
- [other] Filter candidates to those with a double-stranded stem of at least 200 bp, no more than 20% mismatches within the stem, and an adjusted minimum free energy (MFE) of at most -0.35.: [evidence span withheld — anchor status: not_found]
- [other] Slide a 50-bp window across the genome and retain every window containing at least 3 editing sites.: [evidence span withheld — anchor status: not_found]
- [readme] Output will be the list of following files under /output_dir/ dsRID_train.tsv: "Output will be the list of following files under /output_dir/"
