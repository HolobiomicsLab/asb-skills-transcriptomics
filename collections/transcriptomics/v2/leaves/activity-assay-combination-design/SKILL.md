---
name: activity-assay-combination-design
description: Use when deciding which chromatin-assay combination to feed as enhancer
  activity into ABC scoring, e.g. if only ATAC-seq (no DNase-seq + H3K27ac ChIP-seq
  pair) is available for the cell type of interest.
when_to_use_negative:
- Both DNase-seq and H3K27ac ChIP-seq are available and the goal is the original,
  fully specified ABC activity set-up rather than testing single-assay substitutes
- No CRISPR perturbation gold-standard enhancer-gene labels exist for the cell type,
  so AUPRC comparison of assay combinations cannot be validated
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: STARE
  role: Framework executing the ABC scoring of enhancer-gene interactions for each
    activity-column setup
  repo: https://github.com/schulzlab/STARE
- name: stare-abc
  role: Bioconda package providing the STARE_ABCpp binary called with -b, -n, -a,
    -o, -w 5MB, -f, -k, -t 0.02, -q True, -i all_tss
- name: pROC
  role: R package used to compute the AUPRC of each activity setup against the CRISPR
    perturbation gold-standard labels
- name: ABC-Enhancer-Gene-Prediction
  role: Original ABC model whose combined DNase-seq + H3K27ac ChIP-seq activity set-up
    serves as the baseline for the comparison
  repo: https://github.com/broadinstitute/ABC-Enhancer-Gene-Prediction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:48eccf17e6c020505a3ec3e168976b3c49a0e20d27d47e5ec83d354834c1508e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - ABC-Enhancer-Gene-Prediction
  - STARE
  - pROC
  - stare-abc
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# activity-assay-combination-design

## Summary

Designs a comparison of enhancer-activity assay combinations for ABC-based enhancer-gene assignment, testing whether a single assay (DNase-seq, H3K27ac ChIP-seq, ATAC-seq, or their geometric mean) matches the original DNase-seq + H3K27ac ChIP-seq requirement.

## When to use

Use when deciding which chromatin-assay combination to feed as enhancer activity into ABC scoring, e.g. if only ATAC-seq (no DNase-seq + H3K27ac ChIP-seq pair) is available for the cell type of interest.

## When NOT to use

- Both DNase-seq and H3K27ac ChIP-seq are available and the goal is the original, fully specified ABC activity set-up rather than testing single-assay substitutes
- No CRISPR perturbation gold-standard enhancer-gene labels exist for the cell type, so AUPRC comparison of assay combinations cannot be validated

## Inputs

- Candidate-enhancer bed files with separate activity columns (DNase-seq counts, H3K27ac ChIP-seq counts, geometric mean, ATAC-seq counts; all values ≥ 0)
- Gene annotation in GTF format (e.g. Gencode)
- Normalized chromatin contact data (.hic-derived binned matrices, e.g. Fulco et al. 2019 average hg19 Hi-C matrix or ENCFF134PUN averaged hg38 matrix) or -f false for the distance-based contact estimate c = max{d, 5000}^-1
- CRISPR perturbation gold-standard enhancer-gene labels (e.g. Gasperini dataset) for validation

## Outputs

- STARE_ABCpp-scored enhancer-gene interactions for each of the four activity setups and the combined two-assay baseline
- Per-setup AUPRC values computed with pROC against the CRISPR gold-standard labels
- auprc_comparison.csv assembling single-assay and combined-baseline AUPRCs

## How to apply

Build candidate-enhancer bed files in which separate columns hold DNase-seq read counts, H3K27ac ChIP-seq read counts, their geometric mean, and ATAC-seq read counts, with all activity values ≥ 0. For each of the four single-assay setups plus the combined DNase-seq + H3K27ac baseline, call STARE_ABCpp with -b <bed_file>, -n <activity_column>, -a <gtf_annotation>, -o <output_path>, window -w 5MB, contact data -f <contact_data_dir> with -k <bin_size> (or -f false for the distance-based estimate c = max{d, 5000}^-1), cutoff -t 0.02, keeping the gABC defaults -q True (adapted activity) and -i all_tss. Compute the AUPRC of each setup's scored interactions against the CRISPR perturbation gold-standard enhancer-gene labels using pROC in R, then assemble the four single-assay AUPRCs alongside the combined baseline AUPRC into auprc_comparison.csv. The rationale is that the adapted ABC model's accuracy with a single assay is comparable to the two-assay combination, so a single accessible-chromatin assay suffices when the paired assays are unavailable.

## Related tools

- **STARE** (Framework executing the ABC scoring of enhancer-gene interactions for each activity-column setup) — https://github.com/schulzlab/STARE
- **stare-abc** (Bioconda package providing the STARE_ABCpp binary called with -b, -n, -a, -o, -w 5MB, -f, -k, -t 0.02, -q True, -i all_tss)
- **pROC** (R package used to compute the AUPRC of each activity setup against the CRISPR perturbation gold-standard labels)
- **ABC-Enhancer-Gene-Prediction** (Original ABC model whose combined DNase-seq + H3K27ac ChIP-seq activity set-up serves as the baseline for the comparison) — https://github.com/broadinstitute/ABC-Enhancer-Gene-Prediction

## Examples

```
STARE_ABCpp -b enhancers.bed -n atac_count -a gencode.gtf -o out_atac/ -w 5MB -f contact_dir/ -k 5000 -t 0.02 -q True -i all_tss
```

## Evaluation signals

- At least one single-assay AUPRC is comparable to the combined DNase-seq + H3K27ac ChIP-seq baseline AUPRC
- ATAC-seq alone improves AUPRC relative to the combined baseline on the Gasperini dataset
- All activity values in the candidate-enhancer bed columns are ≥ 0
- STARE_ABCpp was run with the gABC defaults -q True (adapted activity) and -i all_tss and cutoff -t 0.02 in every setup, so the setups differ only in the -n activity column
- auprc_comparison.csv contains exactly the four single-assay AUPRCs plus the combined two-assay baseline AUPRC

## Limitations

- The comparison relies on CRISPR perturbation gold-standard datasets; performance conclusions (e.g. ATAC-seq surpassing the combined setup) are dataset-specific, observed on the Gasperini dataset
- Without real contact data (-f false), STARE_ABCpp falls back to the coarse distance-based contact estimate c = max{d, 5000}^-1, which may reduce interaction-scoring accuracy
- Candidate interactions are restricted to the -w 5MB window around the TSS and scored at cutoff -t 0.02, so weaker or more distant interactions are excluded from the AUPRC evaluation

## Evidence

- [abstract] single-assay comparability finding: "Further, we show that the model is comparably accurate with only one assay to measure enhancer activity."
- [abstract] adapted ABC accuracy rationale: [evidence span withheld — anchor status: not_found]
- [other] per-setup STARE_ABCpp invocation parameters: [evidence span withheld — anchor status: not_found]
- [other] distance-based contact fallback: [evidence span withheld — anchor status: not_found]
- [readme] STARE ABC scoring role: "score enhancer-gene interactions with the Activity-By-Contact (ABC)"
