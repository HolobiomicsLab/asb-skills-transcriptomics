---
name: discordant-sample-characterization
description: Use when scATOMIC malignant/non-malignant calls for external tumour biopsies
  must be cross-checked against CopyKAT aneuploid/diploid ploidy status and discordant
  biopsies need causal characterization.
when_to_use_negative:
- Sample is normal (non-cancer) tissue — the default scATOMIC model assumes the presence
  of at least some cancer cells and 'should not be used in normal tissue applications'
  (use the normal_tissue parameter instead)
- Analysis pools cells across multiple patients — the authors strongly recommend running
  scATOMIC only on individual patient/biopsy samples because inter-patient cancer
  cell effects interfere with the cancer signature scoring
- Cancer type is already known and CNV-based reconciliation is unnecessary — use the
  known_cancer_type parameter rather than relying on discordance analysis
tools:
- name: scATOMIC
  role: Generates per-cell malignant/non-malignant predictions via run_scATOMIC()
    on the pan-cancer hierarchy that are reconciled against CNV ploidy
  repo: https://github.com/abelson-lab/scATOMIC
- name: copykat
  role: Performs CNV inference (copy_kat_no_heatmap) on raw counts to assign aneuploid/diploid
    ploidy status used as the comparison standard
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:6b576aa928cff38d563e128a4d3b77ab3145f07800fad0869d38e3c88ff618a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abelson-lab/scATOMIC
  tools:
  - R
  - Rmagic
  - copykat
  - scATOMIC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# discordant-sample-characterization

## Summary

Reconciles scATOMIC's malignant/non-malignant per-cell predictions against CopyKAT-inferred CNV ploidy (aneuploid vs diploid) per biopsy, computes per-sample agreement rates, and annotates discordant samples with biologically plausible causes such as low CNV burden, sub-clonal CNV heterogeneity, near-diploid cancer cells, or too few reference (normal) cells for CopyKAT.

## When to use

Use when scATOMIC malignant/non-malignant calls for external tumour biopsies must be cross-checked against CopyKAT aneuploid/diploid ploidy status and discordant biopsies need causal characterization.

## When NOT to use

- Sample is normal (non-cancer) tissue — the default scATOMIC model assumes the presence of at least some cancer cells and 'should not be used in normal tissue applications' (use the normal_tissue parameter instead)
- Analysis pools cells across multiple patients — the authors strongly recommend running scATOMIC only on individual patient/biopsy samples because inter-patient cancer cell effects interfere with the cancer signature scoring
- Cancer type is already known and CNV-based reconciliation is unnecessary — use the known_cancer_type parameter rather than relying on discordance analysis

## Inputs

- Raw gene-by-cell count matrices of external tumour biopsies (converted to sparse matrices)
- Cell-level quality metadata for filtering (percent mitochondrial reads, number of unique expressed features)
- scATOMIC per-cell malignant/non-malignant predictions from run_scATOMIC()
- CopyKAT aneuploid/diploid ploidy assignments from create_summary_matrix(use_CNVs = TRUE)

## Outputs

- Per-biopsy agreement rate between scATOMIC malignant calls and CopyKAT ploidy status
- Discordance annotations per biopsy (low CNV burden, sub-clonal CNV heterogeneity, near-diploid cancer cells, few reference/control cells)
- cnv_agreement_rates.csv (per-biopsy agreement and discordance annotations)
- Cohort-level summary statistics (median agreement, Q1 threshold)

## How to apply

Run scATOMIC's run_scATOMIC() on filtered sparse gene-by-cell count matrices (drop cells with >25% mitochondrial reads or <500 unique expressed features), then call create_summary_matrix() with use_CNVs = TRUE and modify_results = TRUE, which invokes CopyKAT CNV inference (copy_kat_no_heatmap) on the raw counts; use mc.cores for multi-core parallelism. For each biopsy, reconcile scATOMIC's malignant/non-malignant calls with CopyKAT's aneuploid/diploid assignments and compute the per-biopsy agreement rate. Summarize agreement across the cohort (expected median ~85.9%, discordant first-quartile threshold Q1 ≤ 63.2%). For biopsies below the discordant threshold, annotate the plausible CNV-related failure causes identified in the paper: tumours with low CNV levels, intra-tumoural malignant subclones with variable CNV burdens, near-diploid cancer cells, and few normal cells serving as CopyKAT controls. Save per-biopsy agreement and discordance annotations to cnv_agreement_rates.csv.

## Related tools

- **scATOMIC** (Generates per-cell malignant/non-malignant predictions via run_scATOMIC() on the pan-cancer hierarchy that are reconciled against CNV ploidy) — https://github.com/abelson-lab/scATOMIC
- **copykat** (Performs CNV inference (copy_kat_no_heatmap) on raw counts to assign aneuploid/diploid ploidy status used as the comparison standard)

## Examples

```
summary_results <- create_summary_matrix(sparse_count_mat, use_CNVs = TRUE, modify_results = TRUE, mc.cores = 8)
```

## Evaluation signals

- cnv_agreement_rates.csv exists and contains one agreement-rate row per biopsy with discordance annotations for samples below threshold
- Cohort-level median scATOMIC–CopyKAT agreement reproduces the reported 85.9%
- The first-quartile (Q1) agreement threshold reproduces the reported 63.2% discordant cutoff
- Every discordant biopsy is annotated with at least one plausible CNV-related cause (low CNV level, sub-clonal CNV burden, near-diploid profile, or few normal control cells)
- All per-biopsy agreement rates fall in [0, 1] and match a recomputation from the cell-level malignant and ploidy calls

## Limitations

- CNV inference may fall short in samples with cancer sub-clones of variable CNV burdens, near-diploid cancer cells, or few normal cells to serve as CopyKAT controls, causing false discordance
- Solely relying on inferred CNVs to annotate malignant cells yields false negatives in cancers with minimal genomic structural variation or nearly diploid genomes
- A degree of discordancy still exists between scATOMIC and CNV inference, which is why the authors recommend using scATOMIC in conjunction with CNV inference rather than either alone

## Evidence

- [discussion] in samples with cancer sub-clones defined by variable CNV burdens, cancer cells with near diploid genomic profiles, or few normal cells to serve as controls, CNV inference may fall short: "in samples with cancer sub-clones defined by variable CNV burdens, cancer cells with near diploid genomic profiles, or few normal cells to serve as controls, CNV inference may fall short"
- [discussion] using scATOMIC in conjunction with CNV inference to annotate cancer cells and their type is recommended: "using scATOMIC in conjunction with CNV inference to annotate cancer cells and their type is recommended"
- [intro] Solely relying on the presence of inferred CNVs to annotate malignant cells may lead to false negatives or undefined cells in cancers with minimal genomic structural variation: "Solely relying on the presence of inferred CNVs to annotate malignant cells may lead to false negatives or undefined cells in cancers with minimal genomic structural variation"
- [methods] In this mode we run the create_summary_matrix() function with use_CNVs = TRUE.: "In this mode we run the create_summary_matrix() function with use_CNVs = TRUE."
