---
name: gbm-ensemble-refitting
description: Use when a prior chord() run's saved objects (seu.robj, sce.robj, overkilled.robj)
  exist and additional external doublet detector scores should be integrated into
  a refit GBM ensemble for improved doublet calls.
when_to_use_negative:
- No prior chord() run artifacts exist — the overkill/simulation step and saved .robj
  objects must be generated first via a standard chord() call
- The goal is only the standard three built-in methods with no external scores to
  add — a plain chord(seu, doubletrate, overkill=T, outname=...) call suffices
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: Chord
  role: Provides the expandable port and chord() function whose GBM ensemble is refit
    with addmethods1/addmethods2 score tables
  repo: github.com/13308204545/Chord
- name: gbm
  role: R package implementing the gradient boosting machine used to integrate built-in
    and external method scores as predictors
- name: Scrublet
  role: External Python doublet detector scored on both overkilled.robj and seu.robj
    to append per-cell scores for the refit
  repo: AllonKleinLab/scrublet
- name: DoubletFinder
  role: Built-in method whose scores serve as predictors alongside appended external
    scores in the refit GBM
  repo: chris-mcginnis-ucsf/DoubletFinder
- name: scds
  role: Built-in method providing cxds and bcds predictors whose scores are combined
    with external scores in the refit
  repo: kostkalab/scds
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:5b8ec76aad10d8520671071b46555a76693427dc0775ff14eeba79c4a3406dff
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chord
  - DoubletFinder
  - R
  - Scrublet
  - gbm
  - remotes
  - scds
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gbm-ensemble-refitting

## Summary

Refits Chord's gradient boosting machine (GBM) classifier over the built-in doublet method scores (DoubletFinder, cxds, bcds) plus externally appended method scores (e.g. Scrublet, DoubletDetection), extending the ensemble through Chord's modular port to produce refined doublet calls.

## When to use

Use when a prior chord() run's saved objects (seu.robj, sce.robj, overkilled.robj) exist and additional external doublet detector scores should be integrated into a refit GBM ensemble for improved doublet calls.

## When NOT to use

- No prior chord() run artifacts exist — the overkill/simulation step and saved .robj objects must be generated first via a standard chord() call
- The goal is only the standard three built-in methods with no external scores to add — a plain chord(seu, doubletrate, overkill=T, outname=...) call suffices

## Inputs

- seu.robj (Seurat object saved from a prior chord() run)
- sce.robj (SingleCellExperiment saved from a prior chord() run)
- overkilled.robj (simulated-doublet training object from the overkill step)
- simulated_data.scores.csv (appended external-method scores for simulated data)
- real_data.scores.csv (appended external-method scores for the original data)
- estimated doublet rate (e.g. ~0.9% per 1,000 cells for 10X)

## Outputs

- <outname>2_doublets.csv (refined doublet barcodes from the refit GBM ensemble)
- real_data.scores.csv and simulated_data.scores.csv containing merged external-method score columns

## How to apply

Load the previously saved 'seu.robj' and 'sce.robj' objects exported by the prior chord() run, and score both 'overkilled.robj' (simulated doublets) and 'seu.robj' with the external detector (e.g. Scrublet from AllonKleinLab/scrublet), appending per-cell scores to 'simulated_data.scores.csv' and the real-data score table. Then re-run chord(seu, sce, doubletrate=..., overkill=T, addmethods1='real_data.scores.csv', addmethods2='simulated_data.scores.csv', outname=<new name>) so the GBM (R gbm package) is refit over built-in plus external method scores, treating simulated doublets as positives and singlets as negatives. For 10X data, the doublet rate can be estimated as ~0.9% per 1,000 cells. Read the refined doublet barcodes from '<outname>2_doublets.csv'; the rationale is that selectively integrating better novel methods as they emerge improves Chord's accuracy.

## Related tools

- **Chord** (Provides the expandable port and chord() function whose GBM ensemble is refit with addmethods1/addmethods2 score tables) — github.com/13308204545/Chord
- **gbm** (R package implementing the gradient boosting machine used to integrate built-in and external method scores as predictors)
- **Scrublet** (External Python doublet detector scored on both overkilled.robj and seu.robj to append per-cell scores for the refit) — AllonKleinLab/scrublet
- **DoubletFinder** (Built-in method whose scores serve as predictors alongside appended external scores in the refit GBM) — chris-mcginnis-ucsf/DoubletFinder
- **scds** (Built-in method providing cxds and bcds predictors whose scores are combined with external scores in the refit) — kostkalab/scds

## Examples

```
load("seu.robj"); load("sce.robj"); chord(seu = seu, sce = sce, doubletrate = "estimated doubletrate 2", overkill = T, outname = "the name you want 2", addmethods1 = "real_data.scores.csv", addmethods2 = "simulated_data.scores.csv")
```

## Evaluation signals

- '<outname>2_doublets.csv' is produced in the same directory as the .robj files
- Both real_data.scores.csv and simulated_data.scores.csv contain the appended external-method score columns merged in
- real_data.scores.csv contains one doublet score per cell of the input Seurat object
- The refit chord() call completes without error and the GBM training set includes the new score columns as predictors

## Limitations

- Chord is not optimal in terms of time efficiency because it must integrate the results of multiple methods; refactoring the integrated methods was proposed but not implemented
- The doublet rate must be user-estimated (e.g. ~0.9% per 1,000 cells for 10X), and an incorrect estimate propagates into simulation and refit
- External methods run on merged multi-sample data may perform poorly (Scrublet is designed for single samples and may underperform when cell type proportions are not representative of any single sample)

## Evidence

- [abstract] Chord was designed with a modular architecture port, which has high flexibility and adaptability to the incorporation of any new tools.: "Chord was designed with a modular architecture port, which has high flexibility and adaptability to the incorporation of any new tools."
- [methods] Re-run chord with added method scores to produce refined doublet calls: "chord(seu = seu,sce=sce,doubletrat="estimated doubletrate 2",overkill=T,outname="the name you want 2",addmethods1 ="real_data.scores.csv",addmethods2 = "simulated_data.scores.csv" )"
- [readme] Boost additional methods by evaluating the 'overkilled.robj' dataset: "1.Using any method to evaluate the dataset "overkilled.robj", adding the results of socres to "simulated_data.scores.csv"."
- [discussion] Chord requires the integration of the results of multiple methods, so it is not optimal in terms of time efficiency: "Chord requires the integration of the results of multiple methods, so it is not optimal in terms of time efficiency."
- [discussion] Selectively integrate better novel methods as they emerge to improve Chord's accuracy: [evidence span withheld — anchor status: not_found]
