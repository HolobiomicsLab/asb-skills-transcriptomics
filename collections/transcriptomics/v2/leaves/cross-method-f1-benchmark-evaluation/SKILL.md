---
name: cross-method-f1-benchmark-evaluation
description: Use when comparing rare-cell detection algorithms on scRNA-seq mixtures
  with a known rare population, so that continuous rareness scores and binary cluster
  calls can be converted to a common F1-score comparison.
when_to_use_negative:
- The rare population ground-truth labels are unavailable, since F1 cannot be computed
  without a reference minor population
- Only one method is being applied with no cross-method comparison intended (single-method
  rare-cell discovery, not benchmarking)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FiRE
  role: Assigns a continuous rareness score to every expression profile; scored cells
    are dichotomized via the IQR rule for F1 comparison
  repo: github.com/princethewinner/FiRE
- name: LOF
  role: Alternative density-based scorer producing continuous rareness scores dichotomized
    by the same IQR threshold rule
- name: RaceID
  role: R method producing binary rare-cell predictions compared directly by F1
- name: GiniClust
  role: R method producing binary rare-cell predictions compared directly by F1
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:a2d3c66364763d6ccb947c74096f20065df1f4301a5a31b7086331dce3cebdcf
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cross-method-f1-benchmark-evaluation

## Summary

Benchmark rare-cell detection methods (FiRE, LOF, RaceID, GiniClust) on the GSE94820 293T-Jurkat dilution simulation by dichotomizing all predictions and comparing F1 scores against the ground-truth Jurkat minor population at 0.5%, 2.5% and 5% dilution levels.

## When to use

Use when comparing rare-cell detection algorithms on scRNA-seq mixtures with a known rare population, so that continuous rareness scores and binary cluster calls can be converted to a common F1-score comparison.

## When NOT to use

- The rare population ground-truth labels are unavailable, since F1 cannot be computed without a reference minor population
- Only one method is being applied with no cross-method comparison intended (single-method rare-cell discovery, not benchmarking)

## Inputs

- 293T-Jurkat scRNA-seq mixture from GEO accession GSE94820 (~3200 cells, cells x genes matrix)
- Ground-truth cell labels distinguishing abundant (293T) vs rare (Jurkat) populations
- Continuous rareness scores from FiRE and LOF for every cell
- Binary rare-cell predictions from RaceID and GiniClust

## Outputs

- F1 score per method (FiRE, LOF, RaceID, GiniClust) at each dilution level (0.5%, 2.5%, 5%)
- Fig. 3a-style comparison table of F1 values and method ranking

## How to apply

Download the ~3200-cell 293T-Jurkat scRNA-seq mixture from GEO accession GSE94820 and bioinformatically dilute the annotated Jurkat population to 0.5%, 2.5% and 5% within the 293T background. Run FiRE (installed via './INSTALL --py') and LOF to assign a continuous rareness score to every expression profile, and run RaceID and GiniClust (R installations) to obtain binary rare-cell predictions. Make the four methods comparable by dichotomizing FiRE and LOF scores with the interquartile-range rule (threshold = Q3 + 1.5*IQR of the score distribution, i.e. score >= q3 + 1.5*stats.iqr(score)), as in the FiRE demo. Compute the F1 score of each method at each dilution level against the ground-truth Jurkat labels (label '1' = abundant, '2' = rare) and assemble the Fig. 3a comparison table. The rationale is that only a common threshold rule on continuous scores plus a common metric lets binary predictors (RaceID, GiniClust) be ranked against scoring methods like FiRE.

## Related tools

- **FiRE** (Assigns a continuous rareness score to every expression profile; scored cells are dichotomized via the IQR rule for F1 comparison) — github.com/princethewinner/FiRE
- **LOF** (Alternative density-based scorer producing continuous rareness scores dichotomized by the same IQR threshold rule)
- **RaceID** (R method producing binary rare-cell predictions compared directly by F1)
- **GiniClust** (R method producing binary rare-cell predictions compared directly by F1)

## Examples

```
python example/jurkat_simulation.py; model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489, verbose=0); score = np.array(model.score(preprocessedData)); th = np.percentile(score, 75) + 1.5*stats.iqr(score)
```

## Evaluation signals

- Output table contains F1 values for all four methods (FiRE, LOF, RaceID, GiniClust) at each of the 0.5%, 2.5% and 5% dilutions
- Reported method ranking is consistent with the Fig. 3a reference: FiRE achieves the highest F1 on every test case
- FiRE/LOF binary calls use the exact IQR rule (score >= Q3 + 1.5*IQR) so thresholding is identical across dilution levels
- Dilution mixtures reproduce the intended minor-population fractions (0.5%, 2.5%, 5%) within the 293T background
- All four methods are evaluated on the identical cell set with the same ground-truth Jurkat labels

## Limitations

- RaceID and GiniClust natively offer only binary predictions, so a threshold rule must be imposed on FiRE/LOF scores to make results comparable
- FiRE does not discriminate between an outlier and cells representing minor cell types, which can affect F1 against the Jurkat reference
- Loading the full ~3200-cell jurkat simulation data may require large amounts of RAM; a preprocessed smaller variant is provided

## Evidence

- [discussion] binary-vs-scored-method comparison: "While RaceID15 and GiniClust16 offer binary predictions, FiRE gives a rareness score to every individual expression profile."
- [readme] IQR dichotomization rule: [evidence span withheld — anchor status: external_doc]
- [readme] FiRE score semantics: [evidence span withheld — anchor status: external_doc]
- [abstract] FiRE scoring capability: "an algorithm that, in a matter of seconds, assigns a rareness score to every individual expression profile under study"
- [readme] benchmark invocation: [evidence span withheld — anchor status: external_doc]
