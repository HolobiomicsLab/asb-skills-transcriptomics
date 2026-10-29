---
name: iqr-threshold-sensitivity-analysis
description: Use when benchmarking a rare-cell or doublet detection method whose outlier-calling
  threshold is adjustable (e.g. FiRE's default vs relaxed IQR threshold) and you need
  to show the comparison is not an artifact of a single cutoff choice.
when_to_use_negative:
- The method under comparison has no adjustable outlier threshold, so a single default
  run suffices
- Evaluating GapClust itself under alternative cutoffs — its candidate k is fixed
  by the rule skewness > 2, not an IQR-style threshold
edam_operation: http://edamontology.org/operation_3432
tools:
- name: FiRE
  role: Rare-cell finder evaluated twice per dataset — with the default threshold
    and with a relaxed IQR threshold — to test whether the comparison is threshold-sensitive
- name: GapClust
  role: Reference method whose F1 scores (with candidate k identified where distance-score
    distribution skewness > 2) are compared against the threshold-relaxed baselines
  repo: github.com/fabotao/GapClust
- name: RaceID
  role: Competitor rare-cell detection method run with published default settings
    on every benchmark dataset
- name: GiniClust
  role: Competitor rare-cell detection method run with published default settings
    on every benchmark dataset
- name: CellSIUS
  role: Competitor rare-cell detection method run with published default settings
    on every benchmark dataset
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:4584b8511134ace6e831a1e385b968b259c8122201be0bd0265b1ad0c4beea7f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellSIUS
  - FiRE
  - GapClust
  - GiniClust
  - RaceID
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# iqr-threshold-sensitivity-analysis

## Summary

A benchmarking sensitivity analysis that evaluates a rare-cell detection method (FiRE) under both its default outlier threshold and a relaxed IQR threshold, scoring each setting by precision, recall, and F1 against ground-truth rare-cell labels on subsampled 68k PBMC datasets.

## When to use

Use when benchmarking a rare-cell or doublet detection method whose outlier-calling threshold is adjustable (e.g. FiRE's default vs relaxed IQR threshold) and you need to show the comparison is not an artifact of a single cutoff choice.

## When NOT to use

- The method under comparison has no adjustable outlier threshold, so a single default run suffices
- Evaluating GapClust itself under alternative cutoffs — its candidate k is fixed by the rule skewness > 2, not an IQR-style threshold

## Inputs

- Subsampled 68k PBMC rare-cell benchmark datasets (2-100 planted CD14+ monocytes in 500 NK + 500 B cells)
- Subsampled 68k PBMC doublet benchmark datasets (2 monocytes at 0.08%-5% mixing proportions)
- Ground-truth cell-type labels for planted rare cells and doublets
- Method threshold settings: FiRE default threshold and relaxed IQR threshold

## Outputs

- Per-dataset precision, recall, and F1 scores per method per threshold setting
- Aggregated F1 results per method across the abundance gradient (benchmark_f1_results.csv)
- F1 plots reproducing Fig. 2b and Supplementary Figs. 5 and 8

## How to apply

Run the method under evaluation on every benchmark dataset twice: once with the published default threshold and once with a relaxed IQR threshold, so the baseline is not penalized for conservative cutoffs. Each dataset embeds 2-100 CD14+ monocytes into 500 NK plus 500 B cells (99 rare-cell datasets) or 2 monocytes at mixing proportions from 0.08% to 5% (140 doublet datasets). Score rare-cell calls against ground-truth labels with precision, recall, and F1, aggregating per method across the abundance gradient and per threshold setting. The reference conclusion in this paper is that GapClust, which flags candidate k when the skewness of the distance-score distribution exceeds 2, achieved higher F1 scores than GiniClust, RaceID, CellSIUS, and FiRE, including FiRE evaluated with relaxed IQR thresholds. Report results alongside competitor runs with published defaults (RaceID, GiniClust, CellSIUS) and verify against the paper's Fig. 2b and Supplementary Figs. 5 and 8.

## Related tools

- **FiRE** (Rare-cell finder evaluated twice per dataset — with the default threshold and with a relaxed IQR threshold — to test whether the comparison is threshold-sensitive)
- **GapClust** (Reference method whose F1 scores (with candidate k identified where distance-score distribution skewness > 2) are compared against the threshold-relaxed baselines) — github.com/fabotao/GapClust
- **RaceID** (Competitor rare-cell detection method run with published default settings on every benchmark dataset)
- **GiniClust** (Competitor rare-cell detection method run with published default settings on every benchmark dataset)
- **CellSIUS** (Competitor rare-cell detection method run with published default settings on every benchmark dataset)

## Evaluation signals

- Each method dataset pair must have results for both FiRE threshold settings (default and relaxed IQR), not only the default
- F1, precision, and recall must be computable per dataset and aggregatable per method across the 2-100 monocyte abundance gradient
- Plots should match Fig. 2b and Supplementary Figs. 5 and 8 of the GapClust paper
- Results table benchmark_f1_results.csv should contain per-method, per-threshold aggregated F1 values
- The relaxed-IQR setting should be confirmed to still not exceed GapClust's F1, per the published finding

## Limitations

- Relaxed thresholds can inflate recall but the paper found FiRE with relaxed IQR thresholds still did not surpass GapClust's F1 across all monocyte abundances
- No version numbers were reported for the compared tools (GiniClust, RaceID, CellSIUS, FiRE, EDGE), limiting reproducibility
- No parameter settings for GapClust runs (e.g. K, skewness threshold) are reported in the discussion section, so the exact reference configuration must be taken from the methods/supplement

## Evidence

- [other] Finding: GapClust beats all methods including relaxed-FiRE: [evidence span withheld — anchor status: not_found]
- [other] Workflow step: run FiRE under two thresholds: [evidence span withheld — anchor status: not_found]
- [readme] README: F1-score-based performance claim: "In terms of rare cell detection, GapClust offers best performance compared with other methods, quantified by F"
- [discussion] GapClust candidate k rule: "Candidate k can be identified if the skewness<k> > 2."
