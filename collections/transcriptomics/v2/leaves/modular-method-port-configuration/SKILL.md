---
name: modular-method-port-configuration
description: Use when running Chord/ChordP on a Seurat scRNA-seq object and you want
  to augment the built-in ensemble (scds, DoubletFinder) with scores from additional
  doublet detectors to improve accuracy and stability.
when_to_use_negative:
- When the initial chord() run already meets accuracy needs and no additional method
  scores are available — the base Chord run with overkill=TRUE suffices without the
  port.
- 'When time efficiency is the priority: Chord/ChordP requires integrating results
  of multiple methods and is not optimal in runtime.'
- When data are not droplet-based scRNA-seq (Chord targets doublet artefacts from
  microfluidic/droplet methods).
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3170
tools:
- name: Chord
  role: Ensemble framework whose modular port accepts addmethods1/addmethods2 score
    tables and integrates them via GBM/AdaBoost
  repo: github.com/13308204545/Chord
- name: scds
  role: Built-in method whose cxds/bcds scores feed the ensemble; convert Seurat to
    SCE with creatSCE
  repo: kostkalab/scds
- name: DoubletFinder
  role: Built-in method contributing pANN scores (pN=0.25, pK tuned per dataset via
    mean-variance-normalized bimodality coefficient)
  repo: chris-mcginnis-ucsf/DoubletFinder
- name: Scrublet
  role: Example external method whose scores can be added through the modular port
    (addmethods1/addmethods2)
  repo: AllonKleinLab/scrublet
- name: Solo
  role: Example external semi-supervised deep learning detector whose scores can be
    supplied via the modular port
  repo: calico/solo
- name: gbm
  role: R package providing the gradient boosting algorithm Chord uses to integrate
    method scores
- name: adabag
  role: R package providing the AdaBoost option (method='adaboost', mfinal=40) for
    score integration
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:8566fdfc06adf61ae2aa88eee5e2dbf643f9740a49121eac99eb8769d6158878
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chord
  - DoubletFinder
  - R
  - Scrublet
  - Solo
  - adabag
  - gbm
  - remotes
  - scds
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# modular-method-port-configuration

## Summary

Configures Chord's modular architecture port to incorporate additional doublet detection methods' scores, boosting the ensemble beyond its built-in methods via the GBM/AdaBoost integration. This is how new or better-performing tools are selectively integrated to improve accuracy.

## When to use

Use when running Chord/ChordP on a Seurat scRNA-seq object and you want to augment the built-in ensemble (scds, DoubletFinder) with scores from additional doublet detectors to improve accuracy and stability.

## When NOT to use

- When the initial chord() run already meets accuracy needs and no additional method scores are available — the base Chord run with overkill=TRUE suffices without the port.
- When time efficiency is the priority: Chord/ChordP requires integrating results of multiple methods and is not optimal in runtime.
- When data are not droplet-based scRNA-seq (Chord targets doublet artefacts from microfluidic/droplet methods).

## Inputs

- Seurat object (seu) and SingleCellExperiment object (sce, via creatSCE) from the initial Chord run
- Exported 'overkilled.robj' and 'seu.robj' R data files from the first chord() run
- Scores from an additional doublet detection method applied to both real and simulated data, as CSV ('real_data.scores.csv', 'simulated_data.scores.csv')
- Re-estimated doublet rate (~0.9% per 1,000 cells for 10X data)

## Outputs

- Refined doublet calls with barcodes in 'outname2_doublets.csv'
- Ensemble-integrated doublet scores combining built-in and added methods (ChordP-style result)

## How to apply

After an initial chord() run with overkill=TRUE (which evaluates the data with built-in methods, removes filtered doublets, and simulates doublets from remaining cells to generate the training set), evaluate the two exported datasets 'overkilled.robj' and 'seu.robj' with any additional method (e.g. Scrublet, Solo). Write the real-data scores to 'real_data.scores.csv' and simulated-data scores to 'simulated_data.scores.csv' in the same directory. Then re-run chord() supplying addmethods1='real_data.scores.csv' and addmethods2='simulated_data.scores.csv' along with the re-estimated doubletrate, so the GBM/AdaBoost learner (method='gbm' or 'adaboost', mfinal=40 for adaboost) integrates the new scores into refined doublet calls. Retrieve the final doublet barcodes from the new 'outname2_doublets.csv'. In the benchmark, this port produced ChordP, which achieved the highest average AUC (0.8132) and best average rank (2.285) across seven ground-truth datasets.

## Related tools

- **Chord** (Ensemble framework whose modular port accepts addmethods1/addmethods2 score tables and integrates them via GBM/AdaBoost) — github.com/13308204545/Chord
- **scds** (Built-in method whose cxds/bcds scores feed the ensemble; convert Seurat to SCE with creatSCE) — kostkalab/scds
- **DoubletFinder** (Built-in method contributing pANN scores (pN=0.25, pK tuned per dataset via mean-variance-normalized bimodality coefficient)) — chris-mcginnis-ucsf/DoubletFinder
- **Scrublet** (Example external method whose scores can be added through the modular port (addmethods1/addmethods2)) — AllonKleinLab/scrublet
- **Solo** (Example external semi-supervised deep learning detector whose scores can be supplied via the modular port) — calico/solo
- **gbm** (R package providing the gradient boosting algorithm Chord uses to integrate method scores)
- **adabag** (R package providing the AdaBoost option (method='adaboost', mfinal=40) for score integration)

## Examples

```
load("seu.robj"); load("sce.robj"); chord(seu=seu, sce=sce, doubletrate=0.009, overkill=T, outname="chordP", addmethods1="real_data.scores.csv", addmethods2="simulated_data.scores.csv")
```

## Evaluation signals

- Re-run produces 'outname2_doublets.csv' containing doublet barcodes in the working directory
- Added score tables merge correctly: 'simulated_data.scores.csv' and 'real_data.scores.csv' present in the same directory as 'seu.robj'/'sce.robj' before the second chord() call
- Against experimental doublet annotations, the ported ensemble (ChordP) should yield higher AUC/pAUC metrics than base Chord (benchmark: ChordP average AUC 0.8132 vs Chord 0.801214286 across 7 datasets)
- Stability check: ChordP attains the highest average rank across datasets (2.285 in the benchmark) rather than excelling on only some datasets
- overkilllist/overkill settings verified: filtered doublets removed before simulated doublets are generated from remaining cells

## Limitations

- Chord is not optimal in time efficiency because it must integrate results of multiple methods; refactoring was proposed but not implemented.
- Performance of individual methods varies greatly across datasets, so added methods must be chosen/validated per dataset rather than assumed universally better.
- The doublet rate must be estimated from cell count (~0.9% per 1,000 cells for 10X); mis-estimation propagates into simulation and boosting.
- The claimed benefit of the overkill step for other methods' performance was not tested in the paper.

## Evidence

- [abstract] Chord was designed with a modular architecture port, which has high flexibility and adaptability to the incorporation of any new tools.: "Chord was designed with a modular architecture port, which has high flexibility and adaptability to the incorporation of any new tools."
- [methods] Re-run with added method scores: "chord(seu = seu,sce=sce,doubletrat="estimated doubletrate 2",overkill=T,outname="the name you want 2",addmethods1 ="real_data.scores.csv",addmethods2 = "simulated_data.scores.csv" )"
- [readme] Boost additional methods workflow: "1.Using any method to evaluate the dataset "overkilled.robj", adding the results of socres to "simulated_data.scores.csv"."
- [discussion] Overkill rationale: "Then, filtered doublets identified with various methods are removed, and doublets simulated based on the remaining cells are used to generate a"
- [discussion] Selectively integrate better novel methods: [evidence span withheld — anchor status: not_found]
