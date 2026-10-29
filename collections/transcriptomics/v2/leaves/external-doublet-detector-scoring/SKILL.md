---
name: external-doublet-detector-scoring
description: Use when Chord's built-in tools (DoubletFinder, scds) are insufficient
  and you want to boost the GBM ensemble with an additional detector such as Scrublet
  or solo via the addmethods1/addmethods2 port.
when_to_use_negative:
- When Chord has not been run previously — the score CSVs and 'seu.robj'/'sce.robj'/'overkilled.robj'
  objects must already exist in the working directory.
- 'When you have not yet estimated a doublet rate for the dataset — the re-run requires
  doubletrate (10X: ~0.9% per 1,000 cells).'
- When only experimental labeling (cell hashing, MULTI-seq, demuxlet) is needed, since
  those approaches require special operations and additional costs and serve a different
  purpose than computational score boosting.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: Chord
  role: Ensemble framework whose modular port (addmethods1/addmethods2) ingests external-method
    score tables and refits the GBM to produce ChordP
  repo: https://github.com/13308204545/Chord
- name: Scrublet
  role: External Python doublet detector used to generate per-cell scores appended
    to the real-data and simulated-data score CSVs
  repo: https://github.com/AllonKleinLab/scrublet
- name: DoubletDetection
  role: External Python doublet detector integrated alongside Scrublet in the enhanced
    ChordP GBM model
- name: gbm
  role: R package used by chord() to fit the gradient boosting machine over built-in
    plus external method scores
- name: remotes
  role: Used to install DoubletFinder and Chord from GitHub
- name: scds
  role: Built-in method (cxds/bcds) whose scores are already in the score tables before
    external methods are added
  repo: https://github.com/kostkalab/scds
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:e41b5c670237f27a6f568b65aeef63a33fae49f847780efa766020003e871de7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/13308204545/Chord
  tools:
  - Chord
  - DoubletDetection
  - R
  - Scrublet
  - gbm
  - remotes
  - scds
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# external-doublet-detector-scoring

## Summary

Scores a scRNA-seq dataset with an external Python doublet detector (e.g., Scrublet) and appends the per-cell scores to Chord's 'real_data.scores.csv' and 'simulated_data.scores.csv' so the GBM ensemble can be refit over built-in plus external methods, producing an enhanced 'Chord Plus' (ChordP) model.

## When to use

Use when Chord's built-in tools (DoubletFinder, scds) are insufficient and you want to boost the GBM ensemble with an additional detector such as Scrublet or solo via the addmethods1/addmethods2 port.

## When NOT to use

- When Chord has not been run previously — the score CSVs and 'seu.robj'/'sce.robj'/'overkilled.robj' objects must already exist in the working directory.
- When you have not yet estimated a doublet rate for the dataset — the re-run requires doubletrate (10X: ~0.9% per 1,000 cells).
- When only experimental labeling (cell hashing, MULTI-seq, demuxlet) is needed, since those approaches require special operations and additional costs and serve a different purpose than computational score boosting.

## Inputs

- 'seu.robj' and 'sce.robj' saved by a prior chord() run (real-data Seurat and SingleCellExperiment objects)
- 'overkilled.robj' dataset containing simulated doublets for training
- raw UMI counts matrix (cells as rows, genes as columns) for the external detector (e.g., Scrublet)
- 'real_data.scores.csv' and 'simulated_data.scores.csv' score tables
- estimated doublet rate (10X: ~0.9% per 1,000 cells)

## Outputs

- Updated 'real_data.scores.csv' with appended external-method score columns
- Updated 'simulated_data.scores.csv' with appended external-method score columns
- Refined GBM ensemble (ChordP) model
- 'outname2_doublets.csv' listing the refined doublet barcodes

## How to apply

After an initial chord() run (which saves 'seu.robj', 'sce.robj', 'overkilled.robj', 'real_data.scores.csv', and 'simulated_data.scores.csv'), run the external detector on both the real dataset stored in 'seu.robj' and the simulated-doublet training set 'overkilled.robj', appending per-cell score columns to the respective score CSVs in the same directory. Then re-run chord(seu, sce, doubletrate, overkill=T, addmethods1='real_data.scores.csv', addmethods2='simulated_data.scores.csv', outname=<new name>) so the GBM refits over the combined built-in plus external method scores. For 10X data, estimate doubletrate as ~0.9% per 1,000 cells. The refined doublet barcodes are read from the new 'outname2_doublets.csv' output.

## Related tools

- **Chord** (Ensemble framework whose modular port (addmethods1/addmethods2) ingests external-method score tables and refits the GBM to produce ChordP) — https://github.com/13308204545/Chord
- **Scrublet** (External Python doublet detector used to generate per-cell scores appended to the real-data and simulated-data score CSVs) — https://github.com/AllonKleinLab/scrublet
- **DoubletDetection** (External Python doublet detector integrated alongside Scrublet in the enhanced ChordP GBM model)
- **gbm** (R package used by chord() to fit the gradient boosting machine over built-in plus external method scores)
- **remotes** (Used to install DoubletFinder and Chord from GitHub)
- **scds** (Built-in method (cxds/bcds) whose scores are already in the score tables before external methods are added) — https://github.com/kostkalab/scds

## Examples

```
scrub = scr.Scrublet(counts_matrix); doublet_scores, predicted_doublets = scrub.scrub_doublets()  # then append scores to real_data.scores.csv and simulated_data.scores.csv and re-run: chord(seu=seu, sce=sce, doubletrate=0.009, overkill=T, addmethods1='real_data.scores.csv', addmethods2='simulated_data.scores.csv', outname='chordp')
```

## Evaluation signals

- Both 'real_data.scores.csv' and 'simulated_data.scores.csv' contain the appended external-method score columns in the same directory as the .robj files.
- Re-running chord() with addmethods1/addmethods2 completes and produces the new 'outname2_doublets.csv' file.
- The per-cell score values from the external detector are in the expected continuous range (e.g., Scrublet doublet scores between 0 and 1).
- Barcodes in the appended score columns align 1:1 with the barcodes in 'seu.robj' / 'overkilled.robj' (no dropped or duplicated cells).
- The refined ChordP doublet calls in 'outname2_doublets.csv' are consistent with the estimated doublet rate (~0.9% per 1,000 cells for 10X).

## Limitations

- Chord is not optimal in runtime because it must integrate results of multiple methods; adding external detectors further increases compute time.
- Scrublet is designed to detect technical doublets from random co-encapsulation and may perform poorly on merged datasets where cell type proportions are not representative of any single sample — run it per sample.
- External detector score thresholds may need manual adjustment (e.g., check that Scrublet's threshold separates the bimodal simulated-doublet score histogram).
- The 'overkill' step improving other methods' performance is a proposed but unverified claim.

## Evidence

- [abstract] Chord was designed with a modular architecture port, which has high flexibility and adaptability to the incorporation of any new tools.: "Chord was designed with a modular architecture port, which has high flexibility and adaptability to the incorporation of any new tools."
- [readme] Using any method to evaluate the dataset 'overkilled.robj', adding the results of socres to 'simulated_data.scores.csv': "Using any method to evaluate the dataset "overkilled.robj", adding the results of socres to "simulated_data.scores.csv""
- [readme] scrub = scr.Scrublet(counts_matrix); doublet_scores, predicted_doublets = scrub.scrub_doublets(): "scrub = scr.Scrublet(counts_matrix)
doublet_scores, predicted_doublets = scrub.scrub_doublets()"
- [readme] 10X can be referred：doubletrate = ~0.9% per 1,000 cells: "A:It depends on the number of cells in the sample. 10X can be referred：doubletrate = ~0.9% per 1,000 cells."
- [methods] chord(seu = seu,sce=sce,doubletrat="estimated doubletrate 2",overkill=T,outname="the name you want 2",addmethods1 ="real_data.scores.csv",addmethods2 = "simulated_data.scores.csv" ): "chord(seu = seu,sce=sce,doubletrat="estimated doubletrate 2",overkill=T,outname="the name you want 2",addmethods1 ="real_data.scores.csv",addmethods2 = "simulated_data.scores.csv" )"
- [readme] Because Scrublet is designed to detect technical doublets formed by the random co-encapsulation of two cells, it may perform poorly on merged datasets: [evidence span withheld — anchor status: external_doc]
- [discussion] Chord requires the integration of the results of multiple methods, so it is not optimal in terms of time efficiency: "Chord requires the integration of the results of multiple methods, so it is not optimal in terms of time efficiency."
