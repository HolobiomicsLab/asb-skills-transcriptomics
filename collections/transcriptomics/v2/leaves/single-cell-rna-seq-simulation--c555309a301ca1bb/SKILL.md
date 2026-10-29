---
name: single-cell-rna-seq-simulation
description: Use when benchmarking how many DE genes are needed for a method (e.g.
  FiRE) to detect a rare cell subpopulation (~5% of cells) in simulated scRNA-seq
  data, or when building AUC-ROC sensitivity curves over DE-gene counts.
when_to_use_negative:
- The goal is clustering or typing all major cell populations rather than detecting
  a rare minor subpopulation.
- Input is real single-cell data with known labels; the skill specifically targets
  splatter-simulated two-group data for controlled sensitivity sweeps.
- The rare population differs from the major population by only 1-19 DE genes and
  detection is expected to fail — FiRE struggled below ~20 DE genes.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: FiRE
  role: Assigns a rareness score to every simulated cell profile; scores are converted
    to AUC-ROC against the rare-group labels
  repo: github.com/princethewinner/FiRE
- name: Python
  role: Runs the FiRE module (installed via './INSTALL --py'), drives the simulate-score-repeat
    loop, and computes AUC-ROC averages
- name: splatter
  role: R package used to simulate two-group single-cell expression data with a 5%
    rare population and controlled DE-gene counts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:1a833f8c6b4dc60c12a5a5961bb831ed883a1e34892e7321f41478c4ec8fa63f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - Python
  - splatter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# single-cell-rna-seq-simulation

## Summary

Sensitivity-evaluation skill for rare-cell detection: simulate two-group single-cell expression data with the splatter R package (a 5% rare population among 500 cells x 5000 genes), vary the number of differentially expressed (DE) genes distinguishing the minor group, and score each simulation with FiRE to build an AUC-ROC versus DE-gene-count curve.

## When to use

Use when benchmarking how many DE genes are needed for a method (e.g. FiRE) to detect a rare cell subpopulation (~5% of cells) in simulated scRNA-seq data, or when building AUC-ROC sensitivity curves over DE-gene counts.

## When NOT to use

- The goal is clustering or typing all major cell populations rather than detecting a rare minor subpopulation.
- Input is real single-cell data with known labels; the skill specifically targets splatter-simulated two-group data for controlled sensitivity sweeps.
- The rare population differs from the major population by only 1-19 DE genes and detection is expected to fail — FiRE struggled below ~20 DE genes.

## Inputs

- splatter simulation parameters (batchCells=500, nGenes=5000, group.prob=c(0.95, 0.05))
- DE-gene definitions with effect-size parameters de.facLoc=0.4 and de.facScale=0.8
- list of DE-gene counts to sweep (1 to 150)
- FiRE model parameters (L=100, M=50, H=1017881, seed=5489)

## Outputs

- FiRE rareness score per cell for every simulated dataset (higher scores = rare cells)
- per-repetition AUC-ROC values with the rare group as positives
- mean AUC-ROC versus DE-gene-count curve written to auc_roc_curve.csv

## How to apply

Install the FiRE Python module from the repository with './INSTALL --py' (which runs 'python setup.py install --record fireInstall.log'), then generate two-group splatter simulations with batchCells=500, nGenes=5000, and group.prob=c(0.95, 0.05) so the rare population is 5% of cells. For each DE-gene count from 1 to 150, replace that many non-DE genes with pre-identified DE genes using de.facLoc=0.4 and de.facScale=0.8. Score every cell with a FiRE model (e.g. FiRE.FiRE(L=100, M=50, H=1017881, seed=5489)) where higher FiRE scores represent rare cells, repeating the simulate-and-score procedure 1000 times per DE-gene count. Compute AUC-ROC per repetition treating the rare group as positives, average over the 1000 repetitions, and write mean AUC-ROC versus DE-gene count to auc_roc_curve.csv.

## Related tools

- **FiRE** (Assigns a rareness score to every simulated cell profile; scores are converted to AUC-ROC against the rare-group labels) — github.com/princethewinner/FiRE
- **Python** (Runs the FiRE module (installed via './INSTALL --py'), drives the simulate-score-repeat loop, and computes AUC-ROC averages)
- **splatter** (R package used to simulate two-group single-cell expression data with a 5% rare population and controlled DE-gene counts)

## Examples

```
model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489, verbose=0); model.fit(preprocessedData); score = np.array(model.score(preprocessedData))
```

## Evaluation signals

- auc_roc_curve.csv contains mean AUC-ROC values for every DE-gene count from 1 to 150, averaged over 1000 repetitions per count
- The AUC-ROC curve shows low values for few DE genes and a sharp improvement once 20 or more DE genes are introduced, matching Fig. 4
- The simulated matrix has dimensions 500 cells x 5000 genes with a 95%/5% group split as specified by group.prob=c(0.95, 0.05)
- FiRE scores are higher for the rare group than the abundant group in high-DE-count simulations ('Higher values of FiRE score represent rare cells')
- Reproducibility check: rerunning the pipeline with the same FiRE seed (5489) yields identical scores for a fixed simulation

## Limitations

- FiRE struggled to detect the minor cell population with only a few DE genes; robust detection required 20 or more DE genes
- FiRE does not discriminate between an outlier and cells representing minor cell types, so detected 'rare' cells may be technical outliers
- The FiRE Python package requires specific dependencies (g++ >= 4.8.4, boost >= 1.54.0, cython >= 0.23.4, numpy, pandas, statsmodels) and was tested on Ubuntu 14.04/16.04

## Evidence

- [methods] Simulate two-group single-cell expression data with the splatter R package using batchCells=500, nGenes=5000, group.prob=c(0.95, 0.05): [evidence span withheld — anchor status: not_found]
- [results] Sharp improvement at >=20 DE genes: [evidence span withheld — anchor status: not_found]
- [readme] FiRE score orientation: [evidence span withheld — anchor status: external_doc]
- [readme] FiRE model construction: [evidence span withheld — anchor status: external_doc]
- [discussion] Outlier vs minor cell type caveat: "FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types."
