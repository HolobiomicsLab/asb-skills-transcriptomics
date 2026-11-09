---
name: simulated-doublet-training-set-labeling
description: Use when fitting an ensemble doublet classifier (Chord GBM/AdaBoost)
  that requires labeled training data built from simulated doublets scored by existing
  detection tools rather than experimental ground truth.
when_to_use_negative:
- Ground-truth doublet labels are available experimentally (e.g., cell hashing, MULTI-seq,
  or demuxlet); simulated-doublet labels are unnecessary and the ensemble training
  approach adds cost without benefit.
- The goal is a single-method doublet call rather than an ensemble — scoring the simulated
  set for multiple predictors only pays off when training Chord's GBM/AdaBoost integration.
- 'Runtime is critical: Chord''s training-set scoring and GBM integration make it
  ''not optimal in terms of time efficiency'' compared to single tools.'
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3170
tools:
- name: Chord
  role: Orchestrates overkill simulation, scoring of the simulated training set, and
    GBM ensemble training
  repo: https://github.com/13308204545/Chord
- name: scds
  role: Provides cxds and bcds predictor scores on the simulated training set
  repo: https://github.com/kostkalab/scds
- name: DoubletFinder
  role: Provides artificial-nearest-neighbor (pANN) predictor scores with pN=0.25
    and tuned pK on the simulated training set
  repo: https://github.com/chris-mcginnis-ucsf/DoubletFinder
- name: gbm
  role: R package fitting the GBM classifier (1000 trees, shrinkage 0.01, cv.folds
    5) on the labeled training scores
- name: R
  role: Runtime environment for the Chord/scds/DoubletFinder/gbm pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:90a3e6f85c8f1ebc9857e2bd990bb4d4116fe0cbbe48285044c2be4c8f425bc4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/13308204545/Chord
  tools:
  - Chord
  - DoubletFinder
  - R
  - gbm
  - scds
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# simulated-doublet-training-set-labeling

## Summary

Assemble a supervised training set for Chord's second-stage GBM by scoring a simulated-doublet ('overkilled') dataset with DoubletFinder and scds (cxds, bcds), then labeling simulated doublets as positives and singlets as negatives so the GBM can learn to weight tool-predicted scores.

## When to use

Use when fitting an ensemble doublet classifier (Chord GBM/AdaBoost) that requires labeled training data built from simulated doublets scored by existing detection tools rather than experimental ground truth.

## When NOT to use

- Ground-truth doublet labels are available experimentally (e.g., cell hashing, MULTI-seq, or demuxlet); simulated-doublet labels are unnecessary and the ensemble training approach adds cost without benefit.
- The goal is a single-method doublet call rather than an ensemble — scoring the simulated set for multiple predictors only pays off when training Chord's GBM/AdaBoost integration.
- Runtime is critical: Chord's training-set scoring and GBM integration make it 'not optimal in terms of time efficiency' compared to single tools.

## Inputs

- overkilled.robj — SingleCellExperiment/Seurat-style object containing simulated doublets generated after the overkill filtering step
- seu.robj — input Seurat object (converted to SingleCellExperiment with creatSCE for scds compatibility)
- Parameters: cxds.ntop, cxds.binThresh, bcds.ntop, bcds.srat, dbf.PCs; dbf.pN fixed at 0.25 and dbf.pK tuned per dataset

## Outputs

- simulated_data.scores.csv — per-cell predictor scores from cxds, bcds, and DoubletFinder on the simulated training set
- Labeled training matrix with simulated doublets as positives and singlets as negatives, consumed by the GBM fit
- real_data.scores.csv — matching per-cell scores on the original dataset (produced downstream by DBboostPre)

## How to apply

Load the Seurat object (seu.robj) and the simulated-doublet training object (overkilled.robj) produced by the overkill stage, converting the Seurat object to a SingleCellExperiment via creatSCE for scds compatibility. Score all cells with scds using the cxds and bcds predictors (honoring cxds.ntop, cxds.binThresh, bcds.ntop, bcds.srat) and with DoubletFinder using artificial nearest neighbors with pN fixed at 0.25 (the pN-invariant default), dataset-tuned pK, and dbf.PCs principal components. Assemble the per-cell predictor score matrices and label simulated doublets as positives and singlets as negatives to form the training set. This labeled matrix is what the GBM classifier (R gbm package: 1000 trees, shrinkage = 0.01, cv.folds = 5) consumes as predictors; the same scoring recipe is applied to the original data so train and test predictor spaces match. Additional methods can be folded in by evaluating overkilled.robj and appending scores to simulated_data.scores.csv before re-running chord with addmethods2.

## Related tools

- **Chord** (Orchestrates overkill simulation, scoring of the simulated training set, and GBM ensemble training) — https://github.com/13308204545/Chord
- **scds** (Provides cxds and bcds predictor scores on the simulated training set) — https://github.com/kostkalab/scds
- **DoubletFinder** (Provides artificial-nearest-neighbor (pANN) predictor scores with pN=0.25 and tuned pK on the simulated training set) — https://github.com/chris-mcginnis-ucsf/DoubletFinder
- **gbm** (R package fitting the GBM classifier (1000 trees, shrinkage 0.01, cv.folds 5) on the labeled training scores)
- **R** (Runtime environment for the Chord/scds/DoubletFinder/gbm pipeline)

## Examples

```
chord(seu = seu, sce = sce, doubletrate = 0.09, overkill = TRUE, outname = "myrun")  # produces overkilled.robj, then scds(sce, cxds.ntop=500, bcds.ntop=500) and DoubletFinder scoring feed the GBM
```

## Evaluation signals

- real_data.scores.csv exists and contains exactly one doublet score per cell of the input Seurat object.
- simulated_data.scores.csv contains score columns for each integrated predictor (cxds, bcds, DoubletFinder) aligned to the cells of overkilled.robj.
- Label assignments partition the simulated set correctly: simulated doublets = positives, singlets = negatives, matching the overkill simulation provenance.
- GBM fit completes with the documented hyperparameters (1000 trees, shrinkage = 0.01, cv.folds = 5) without NA predictor scores.
- outname_doublets.csv barcodes correspond to cells whose final ensemble score exceeds the doublet-rate-derived threshold.

## Limitations

- Chord is not optimal in runtime because it must integrate results of multiple methods; refactoring the integrated methods was proposed but not implemented.
- Label quality depends on simulated doublets: only filtered doublets are removed first and simulations are generated from remaining cells, so heterotypic combinations not present in the residual population are unrepresented.
- pK for DoubletFinder has no default and must be adjusted per dataset (e.g., via mean-variance-normalized bimodality coefficient); a poorly tuned pK degrades training labels.
- The doublet rate must be estimated by the user (10X guidance: ~0.9% per 1,000 cells), and an incorrect rate biases the positive:negative balance.

## Evidence

- [methods] GBM training on tool scores: [evidence span withheld — anchor status: not_found]
- [methods] Score assembly and labeling: [evidence span withheld — anchor status: not_found]
- [discussion] Ensemble integration rationale: "To solve this problem, Chord integrates the results of different methods through the GBM algorithm."
- [discussion] Overkill simulation produces the training data: "Then, filtered doublets identified with various methods are removed, and doublets simulated based on the remaining cells are used to generate a"
- [readme] Augmenting training scores with new methods: "Using any method to evaluate the dataset "overkilled.robj", adding the results of socres to "simulated_data.scores.csv"."
- [readme] pN-invariant DoubletFinder default: "Default is set to 0.25, based on observation that DoubletFinder performance is largely pN-invariant (see McGinnis, Murrow and Gartner 2019, Cell Systems)."
