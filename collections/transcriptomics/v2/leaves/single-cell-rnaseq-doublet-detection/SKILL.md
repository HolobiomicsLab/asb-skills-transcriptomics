---
name: single-cell-rnaseq-doublet-detection
description: Use when a scRNA-seq doublet analysis needs to boost Chord beyond its
  three built-in tools by appending scores from any external doublet detector to the
  saved chord() objects and re-running the ensemble.
when_to_use_negative:
- When the goal is a single-method doublet detection without ensemble aggregation
  — Chord's GBM integration adds runtime overhead and is not optimal in time efficiency
- When no prior chord() run has been completed, since the port requires the exported
  seu.robj, sce.robj, and score CSV files
- When experimental doublet labeling (cell hashing, MULTI-seq, demuxlet) is already
  available, which labels doublets directly rather than computationally
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_6170
tools:
- name: Chord
  role: Ensemble GBM framework whose expandable port (addmethods1/addmethods2 parameters)
    ingests external method scores for refitting
  repo: https://github.com/13308204545/Chord
- name: Scrublet
  role: External Python doublet detector scored over 'overkilled.robj' and 'seu.robj'
    to supply additional per-cell scores to the ensemble
  repo: https://github.com/AllonKleinLab/scrublet
- name: gbm
  role: R package providing the gradient boosting machine algorithm used to integrate
    built-in plus external method scores
- name: remotes
  role: R package used to install DoubletFinder and Chord from GitHub
- name: scds
  role: Built-in R doublet detection method (cxds/bcds) whose scores form part of
    the base ensemble before external augmentation
  repo: https://github.com/kostkalab/scds
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:8675c0f138820893729ecb259d256a39c8cc7c316b86f0a622973cb02e1b6062
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/13308204545/Chord
  tools:
  - Chord
  - R
  - Scrublet
  - gbm
  - remotes
  - scds
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_6170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# single-cell-rnaseq-doublet-detection

## Summary

Integrate external doublet detection methods (e.g. Python-based Scrublet or DoubletDetection) into Chord's ensemble GBM model through its expandable port, producing an enhanced Chord Plus (ChordP) version that refits the GBM over built-in plus external method scores.

## When to use

Use when a scRNA-seq doublet analysis needs to boost Chord beyond its three built-in tools by appending scores from any external doublet detector to the saved chord() objects and re-running the ensemble.

## When NOT to use

- When the goal is a single-method doublet detection without ensemble aggregation — Chord's GBM integration adds runtime overhead and is not optimal in time efficiency
- When no prior chord() run has been completed, since the port requires the exported seu.robj, sce.robj, and score CSV files
- When experimental doublet labeling (cell hashing, MULTI-seq, demuxlet) is already available, which labels doublets directly rather than computationally

## Inputs

- seu.robj — Seurat object exported by a prior chord() run
- sce.robj — SingleCellExperiment object exported by a prior chord() run
- overkilled.robj — filtered dataset for scoring with the external method
- simulated_data.scores.csv — per-cell score table for simulated data
- real_data.scores.csv — per-cell score table for real data
- Estimated doublet rate (e.g. ~0.9% per 1,000 cells for 10X data)

## Outputs

- Updated score CSVs with appended external-method score columns (same directory as the .robj files)
- outname2_doublets.csv — refined doublet barcodes after GBM refit
- ChordP — the enhanced Chord Plus model incorporating external method scores

## How to apply

After an initial chord() run, load the exported objects from 'seu.robj' and 'sce.robj' in the output directory. Evaluate the dataset 'overkilled.robj' with the external detector (e.g. Scrublet, which simulates doublets and computes a k-NN classifier doublet score between 0 and 1) and append the per-cell scores as new columns to 'simulated_data.scores.csv'; separately score the 'seu.robj' dataset and append its per-cell scores to the real-data score table. Then re-run chord(seu, sce, doubletrate=<~0.9% per 1,000 cells for 10X data>, overkill=T, addmethods1='real_data.scores.csv', addmethods2='simulated_data.scores.csv', outname=<new name>) so the GBM is refit over the union of built-in and external method scores, yielding refined doublet calls in '<outname>2_doublets.csv'. The rationale is Chord's modular architecture port, which lets better novel methods be selectively integrated to improve ensemble accuracy over any single method.

## Related tools

- **Chord** (Ensemble GBM framework whose expandable port (addmethods1/addmethods2 parameters) ingests external method scores for refitting) — https://github.com/13308204545/Chord
- **Scrublet** (External Python doublet detector scored over 'overkilled.robj' and 'seu.robj' to supply additional per-cell scores to the ensemble) — https://github.com/AllonKleinLab/scrublet
- **gbm** (R package providing the gradient boosting machine algorithm used to integrate built-in plus external method scores)
- **remotes** (R package used to install DoubletFinder and Chord from GitHub)
- **scds** (Built-in R doublet detection method (cxds/bcds) whose scores form part of the base ensemble before external augmentation) — https://github.com/kostkalab/scds

## Examples

```
load("seu.robj"); load("sce.robj"); chord(seu=seu, sce=sce, doubletrate=0.009, overkill=T, outname="chordp", addmethods1="real_data.scores.csv", addmethods2="simulated_data.scores.csv")
```

## Evaluation signals

- Validation: confirm that '<outname>2_doublets.csv' is produced after the re-run
- Confirm both score CSVs contain the appended external-method score columns merged in the same directory as the .robj files
- Score columns must be appended per-cell and aligned with the cells in seu.robj / overkilled.robj so the GBM can merge them
- Scrublet doublet scores should fall between 0 and 1; the threshold should ideally separate two peaks of a bimodal simulated doublet score histogram
- Predicted doublets should mostly co-localize when visualized in a UMAP or t-SNE embedding; if not, adjust the threshold or pre-processing parameters

## Limitations

- Chord requires integration of results from multiple methods, so it is not optimal in terms of time efficiency
- Scrublet may perform poorly on merged multi-sample datasets where cell type proportions are not representative of any single sample; run it on each sample separately
- The doublet score threshold may need manual adjustment if the simulated doublet score histogram is not cleanly bimodal
- Estimated doublet rate depends on cell count (10X guideline: ~0.9% per 1,000 cells); mis-estimation affects simulated training doublets

## Evidence

- [readme] Expandable port for boosting more methods (README): "Using any method to evaluate the dataset "overkilled.robj", adding the results of socres to "simulated_data.scores.csv"."
- [readme] Re-run chord with added method scores (README): "chord(seu = seu,sce=sce,doubletrat="estimated doubletrate 2",overkill=T,outname="the name you want 2",addmethods1 ="real_data.scores.csv",addmethods2 = "simulated_data.scores.csv" )"
- [abstract] Modular architecture port (abstract): "Chord was designed with a modular architecture port, which has high flexibility and adaptability to the incorporation of any new tools."
- [discussion] Selective integration of new methods (discussion): [evidence span withheld — anchor status: not_found]
- [discussion] GBM integration rationale (discussion): "To solve this problem, Chord integrates the results of different methods through the GBM algorithm."
- [readme] Doublet rate estimation (readme): "It depends on the number of cells in the sample. 10X can be referred：doubletrate = ~0.9% per 1,000 cells."
