---
name: auc-roc-monte-carlo-estimation
description: Use when quantifying how sensitivity of a rareness scorer (e.g. FiRE
  scores) varies with the number of DE genes distinguishing a ~5% minor subpopulation,
  requiring mean AUC-ROC over many simulated replicates.
when_to_use_negative:
- Input is real experimental data with known rare-cell labels — evaluate with a single
  AUC-ROC, not Monte Carlo splatter simulation
- The rare population is far larger than 5% of cells or the groups are binary-predicted
  (RaceID/GiniClust), where AUC-ROC over continuous scores is not applicable
- You need runtime/speed benchmarks of FiRE rather than detection sensitivity versus
  DE-gene count
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: FiRE
  role: Assigns a continuous rareness score to every simulated cell so AUC-ROC can
    be computed with the rare group as positives
  repo: github.com/princethewinner/FiRE
- name: Python
  role: Hosts the FiRE module (installed via './INSTALL --py') used to fit the model
    and compute scores inside the Monte Carlo loop
  repo: github.com/princethewinner/FiRE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:dbcc5b588e77d0945ba5b56ca2dc2d8ad181a2b0100dfea4a2882b053fb8b7df
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# auc-roc-monte-carlo-estimation

## Summary

Estimate detection sensitivity of a rare-cell scorer by averaging AUC-ROC over 1000 independent Monte Carlo repetitions of a splatter-simulated two-group scRNA-seq experiment (500 cells, 5000 genes, 5% rare subpopulation), sweeping the number of differentially expressed genes from 1 to 150.

## When to use

Use when quantifying how sensitivity of a rareness scorer (e.g. FiRE scores) varies with the number of DE genes distinguishing a ~5% minor subpopulation, requiring mean AUC-ROC over many simulated replicates.

## When NOT to use

- Input is real experimental data with known rare-cell labels — evaluate with a single AUC-ROC, not Monte Carlo splatter simulation
- The rare population is far larger than 5% of cells or the groups are binary-predicted (RaceID/GiniClust), where AUC-ROC over continuous scores is not applicable
- You need runtime/speed benchmarks of FiRE rather than detection sensitivity versus DE-gene count

## Inputs

- splatter simulation specification: batchCells=500, nGenes=5000, group.prob=c(0.95, 0.05)
- DE gene configuration: de.facLoc=0.4, de.facScale=0.8, DE-gene counts swept 1–150
- preprocessed expression matrix (cells x genes, log2-transformed variable genes) for FiRE scoring

## Outputs

- auc_roc_curve.csv: mean AUC-ROC versus number of DE genes (averaged over 1000 repetitions per count)
- per-repetition AUC-ROC values for each DE-gene count (1000 replicates each)

## How to apply

Simulate two-group expression data with the splatter R package using batchCells=500, nGenes=5000 and group.prob=c(0.95, 0.05), so the rare group is 5% of cells; pre-identify the differentially expressed genes with de.facLoc=0.4 and de.facScale=0.8. For each DE-gene count from 1 to 150, replace that many non-DE genes with the DE genes to control the separation strength between groups. Score every cell with the FiRE Python module (fit on the samples-by-features matrix, then call score), treating the rare group as positives; FiRE's continuous rareness scores — unlike the binary predictions of RaceID or GiniClust — support direct AUC-ROC computation. Repeat the simulation-and-scoring procedure 1000 times per DE-gene count, compute AUC-ROC per repetition, and average. Write the mean AUC-ROC versus DE-gene-count curve to auc_roc_curve.csv and inspect for the sharp improvement at >=20 DE genes reported in Fig. 4.

## Related tools

- **FiRE** (Assigns a continuous rareness score to every simulated cell so AUC-ROC can be computed with the rare group as positives) — github.com/princethewinner/FiRE
- **Python** (Hosts the FiRE module (installed via './INSTALL --py') used to fit the model and compute scores inside the Monte Carlo loop) — github.com/princethewinner/FiRE

## Examples

```
model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489, verbose=0); model.fit(preprocessedData); score = np.array(model.score(preprocessedData))
```

## Evaluation signals

- auc_roc_curve.csv contains 150 DE-gene counts (1–150), each with an AUC-ROC averaged over 1000 repetitions
- Mean AUC-ROC is near chance (~0.5) for very few DE genes (1–few) and improves sharply once 20 or more DE genes are introduced
- FiRE scores are continuous (np.array[nCells]) with higher values marking rare cells, so ROC thresholds can be swept
- Each simulation retains the 95%/5% group split (batchCells=500, group.prob=c(0.95, 0.05)) so the positive class stays rare
- Repetition counts and splatter parameters (de.facLoc=0.4, de.facScale=0.8) match the Fig. 4 protocol

## Limitations

- FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types, which can contaminate simulated rare-group positives
- FiRE struggled to detect the minor population when only a few DE genes distinguish it — sensitivity is poor at low DE-gene counts
- Performance depends on preprocessing (ranger_preprocess selects 1000 variable genes by default), which may drop genuinely DE genes from the scored feature set

## Evidence

- [readme] score is a continuous rareness measure supporting AUC-ROC: [evidence span withheld — anchor status: external_doc]
- [intro] evaluation on simulated datasets grounds the Monte Carlo AUC-ROC design: "We evaluated FiRE on a number of real and simulated datasets."
- [discussion] FiRE scores (unlike RaceID/GiniClust binary calls) enable ROC analysis: "While RaceID15 and GiniClust16 offer binary predictions, FiRE gives a rareness score to every individual expression profile."
- [readme] concrete FiRE model instantiation used for scoring simulated cells: [evidence span withheld — anchor status: external_doc]
