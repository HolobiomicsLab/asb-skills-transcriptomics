---
name: simulated-cell-mixture-dilution
description: Use when a labeled two-population scRNA-seq mixture is available and
  you need a controlled simulation of rare-population abundance (e.g. 0.5–5%) to benchmark
  rare-cell detection tools with F1 scores.
when_to_use_negative:
- The dataset has more than two labeled cell types or no ground-truth rare-population
  annotation, so controlled dilution and F1 computation are not possible.
- The goal is detecting novel, unannotated rare cell types in real tissue data rather
  than benchmarking known-population recovery (FiRE itself does not discriminate outliers
  from minor cell types).
- Input is a small mixture where the rare population fraction is already fixed and
  dilution simulation would leave too few rare cells for stable F1 estimates.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: FiRE
  role: Assigns a continuous rareness score to every cell; scores are dichotomized
    by the IQR rule for F1 comparison
  repo: github.com/princethewinner/FiRE
- name: LOF
  role: Baseline continuous outlier/rareness scorer compared against FiRE at each
    dilution
- name: RaceID
  role: R-based baseline producing binary rare-cell predictions on each simulated
    mixture
- name: GiniClust
  role: R-based baseline producing binary rare-cell predictions on each simulated
    mixture
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:d43dc81e814ebe4e00f9e48c96b5ef0ff1419495bfda8e96b24aa6414cbf6a3a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - GiniClust
  - LOF
  - R
  - RaceID
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# simulated-cell-mixture-dilution

## Summary

Benchmark rare-cell detection methods by bioinformatically diluting an annotated rare cell population (e.g. Jurkat) into an abundant background (e.g. 293T) at controlled fractions (0.5%, 2.5%, 5%) and comparing method performance via F1 score against the ground-truth rare labels, as done for FiRE vs LOF, RaceID and GiniClust.

## When to use

Use when a labeled two-population scRNA-seq mixture is available and you need a controlled simulation of rare-population abundance (e.g. 0.5–5%) to benchmark rare-cell detection tools with F1 scores.

## When NOT to use

- The dataset has more than two labeled cell types or no ground-truth rare-population annotation, so controlled dilution and F1 computation are not possible.
- The goal is detecting novel, unannotated rare cell types in real tissue data rather than benchmarking known-population recovery (FiRE itself does not discriminate outliers from minor cell types).
- Input is a small mixture where the rare population fraction is already fixed and dilution simulation would leave too few rare cells for stable F1 estimates.

## Inputs

- 293T-Jurkat scRNA-seq mixture count matrix (~3200 cells) from GEO accession GSE94820 with per-cell identity labels (abundant vs rare)
- Dilution fractions to simulate (e.g. 0.5%, 2.5%, 5%)
- FiRE rareness scores (continuous, higher = rarer) and LOF scores per cell
- Binary rare-cell calls from RaceID and GiniClust

## Outputs

- F1 score per method (FiRE, LOF, RaceID, GiniClust) at each Jurkat dilution level (Fig. 3a reproduction table)
- Binary rare/non-rare prediction vector per method per dilution
- Ranking of methods by F1 against ground-truth Jurkat labels

## How to apply

Download the ~3200-cell 293T-Jurkat mixture (GEO GSE94820) with annotated identities. Subsample the annotated Jurkat population to 0.5%, 2.5% and 5% within the 293T background to build three simulated mixtures. Run score-based methods (FiRE via './INSTALL --py', LOF) and binary-prediction methods (RaceID, GiniClust via './INSTALL --R') on each mixture; dichotomize FiRE/LOF continuous rareness scores with the IQR rule (threshold = Q3 + 1.5*IQR of the score) so all methods yield binary rare-cell calls. Compute F1 scores per method per dilution with respect to the Jurkat ground truth and assemble a comparison table mirroring Fig. 3a; FiRE achieved the highest F1 on every tested dilution.

## Related tools

- **FiRE** (Assigns a continuous rareness score to every cell; scores are dichotomized by the IQR rule for F1 comparison) — github.com/princethewinner/FiRE
- **LOF** (Baseline continuous outlier/rareness scorer compared against FiRE at each dilution)
- **RaceID** (R-based baseline producing binary rare-cell predictions on each simulated mixture)
- **GiniClust** (R-based baseline producing binary rare-cell predictions on each simulated mixture)

## Examples

```
python example/jurkat_simulation.py  # or small demo: python example/jurkat_simulation_small.py; then th = np.percentile(score,75) + 1.5*stats.iqr(score)
```

## Evaluation signals

- Output table contains F1 values for all four methods (FiRE, LOF, RaceID, GiniClust) across the 0.5%, 2.5% and 5% dilutions
- Reported method ranking is consistent with the Fig. 3a reference: FiRE achieves the highest F1 at every dilution level
- FiRE/LOF binary calls derived via the IQR rule (threshold = Q3 + 1.5*IQR) and all four methods compared on identical ground-truth Jurkat labels
- Simulated mixtures contain the intended Jurkat fractions (0.5%, 2.5%, 5%) within the 293T background, verifiable from the label vector counts
- FiRE run reproduces via the demo pattern: python example/jurkat_simulation.py on GSE94820-derived data

## Limitations

- FiRE does not discriminate between an outlier and cells representing minor cell types, so dilution-simulation F1 reflects minor-population recovery, not outlier rejection.
- RaceID and GiniClust give binary predictions while FiRE and LOF give scores, requiring a thresholding rule (IQR) that introduces a comparability choice.
- Doublet rate may vary between 1% and 30%, a range that does not necessarily comply with the notion of cell rareness; simulated rareness fractions may not map onto doublet-detection benchmarks.

## Evidence

- [results] FiRE outperformed at all dilutions: [evidence span withheld — anchor status: not_found]
- [readme] IQR thresholding in README: [evidence span withheld — anchor status: external_doc]
- [readme] Threshold formula: "q3 = np.percentile(score, 75)
iqr = stats.iqr(score)
th = q3 + 1.5*iqr"
- [discussion] Binary vs score methods: "While RaceID15 and GiniClust16 offer binary predictions, FiRE gives a rareness score to every individual expression profile."
- [readme] Label semantics: [evidence span withheld — anchor status: external_doc]
