---
name: doublet-score-integration-across-methods
description: Use when a Seurat/SCE dataset has already been run once through chord()
  and you want to boost the ensemble with additional doublet detectors' per-cell score
  columns.
when_to_use_negative:
- No prior chord() run exists — the .robj exported objects and overkill/simulated-data
  score files are prerequisites for the addmethods1/addmethods2 port
- The goal is a single-method doublet call rather than an ensemble — running one external
  detector directly is faster, and Chord is known to be suboptimal in runtime because
  it integrates multiple methods
- Data comes from multiple samples merged into one matrix — Scrublet should be run
  per sample separately, and merged datasets with unrepresentative cell-type proportions
  degrade performance
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: Chord
  role: Ensemble framework whose GBM re-fit ingests external method scores via addmethods1/addmethods2
  repo: https://github.com/13308204545/Chord
- name: Scrublet
  role: External Python doublet detector used to produce appended per-cell score columns
  repo: https://github.com/AllonKleinLab/scrublet
- name: gbm
  role: Boosting algorithm that integrates built-in and added methods' scores into
    the final doublet call
- name: remotes
  role: Used to install DoubletFinder and other GitHub-hosted dependencies
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:53064d848de0f57eaa5ad8b763b8697cb7b910ef6b7093a621cf1078a8e20ac0
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# doublet-score-integration-across-methods

## Summary

Extends the Chord ensemble doublet detector beyond its three built-in methods by scoring real and simulated datasets with external tools (e.g., Scrublet, DoubletDetection) and re-fitting the GBM over the union of built-in plus appended per-cell scores.

## When to use

Use when a Seurat/SCE dataset has already been run once through chord() and you want to boost the ensemble with additional doublet detectors' per-cell score columns.

## When NOT to use

- No prior chord() run exists — the .robj exported objects and overkill/simulated-data score files are prerequisites for the addmethods1/addmethods2 port
- The goal is a single-method doublet call rather than an ensemble — running one external detector directly is faster, and Chord is known to be suboptimal in runtime because it integrates multiple methods
- Data comes from multiple samples merged into one matrix — Scrublet should be run per sample separately, and merged datasets with unrepresentative cell-type proportions degrade performance

## Inputs

- seu.robj (Seurat object exported by a prior chord() run)
- sce.robj (SingleCellExperiment object exported by a prior chord() run)
- overkilled.robj (overkilled dataset from the prior run)
- real_data.scores.csv and simulated_data.scores.csv (per-cell score tables)
- Estimated doublet rate (~0.9% per 1,000 cells for 10X)

## Outputs

- Updated real_data.scores.csv and simulated_data.scores.csv with appended external-method score columns
- outname2_doublets.csv containing refined doublet barcodes from the re-fit GBM ensemble

## How to apply

After an initial chord() run exports 'seu.robj' and 'sce.robj', load these objects and score both the real dataset ('seu.robj') and the overkilled/simulated dataset ('overkilled.robj') with an external detector such as Scrublet from AllonKleinLab/scrublet. Append the external per-cell scores to 'simulated_data.scores.csv' (from overkilled.robj and seu.robj evaluations) and 'real_data.scores.csv', keeping the files in the same directory as the .robj objects. Then re-run chord(seu, sce, doubletrate=<estimated rate, ~0.9% per 1,000 cells for 10X>, overkill=T, addmethods1='real_data.scores.csv', addmethods2='simulated_data.scores.csv', outname=<new name>) so the GBM is refit over built-in plus external method scores. Rationale: Chord's modular architecture port is explicitly designed so that better novel methods can be selectively integrated to improve accuracy, and ensemble integration via GBM addresses the instability of any single method.

## Related tools

- **Chord** (Ensemble framework whose GBM re-fit ingests external method scores via addmethods1/addmethods2) — https://github.com/13308204545/Chord
- **Scrublet** (External Python doublet detector used to produce appended per-cell score columns) — https://github.com/AllonKleinLab/scrublet
- **gbm** (Boosting algorithm that integrates built-in and added methods' scores into the final doublet call)
- **remotes** (Used to install DoubletFinder and other GitHub-hosted dependencies)

## Examples

```
load("seu.robj"); load("sce.robj"); chord(seu=seu, sce=sce, doubletrate=0.009, overkill=T, outname="chordp", addmethods1="real_data.scores.csv", addmethods2="simulated_data.scores.csv")
```

## Evaluation signals

- Re-run completes and 'outname2_doublets.csv' is produced (the refined doublet barcodes file)
- Both score CSVs contain the appended external-method score columns merged in the same directory as the .robj files
- Appended score columns align row-for-row with cells in seu.robj / overkilled.robj so the GBM can consume them
- External scores are on a comparable per-cell scale (e.g., Scrublet's continuous doublet_score between 0 and 1) suitable for GBM integration
- Refined calls can be compared to the initial outname_doublets.csv to confirm the added methods changed the ensemble output

## Limitations

- Chord is not optimal in time efficiency because it must integrate results of multiple methods; refactoring the integrated methods was proposed but not implemented
- Performance of the underlying detectors varies greatly across datasets; no single method is a gold standard for every scenario
- The claim that the 'overkill' step improves other methods' performance was not tested in this work
- When scoring with Scrublet on multi-sample data, per-sample runs are required; merged datasets may perform poorly

## Evidence

- [methods] Modular port for adding methods: "Using any method to evaluate the dataset "overkilled.robj", adding the results of socres to "simulated_data.scores.csv"."
- [readme] Re-run chord with added scores: "chord(seu = seu,sce=sce,doubletrat="estimated doubletrate 2",overkill=T,outname="the name you want 2",addmethods1 ="real_data.scores.csv",addmethods2 = "simulated_data.scores.csv" )"
- [discussion] Ensemble integration rationale: "To solve this problem, Chord integrates the results of different methods through the GBM algorithm."
- [abstract] Modular architecture for new tools: "Chord was designed with a modular architecture port, which has high flexibility and adaptability to the incorporation of any new tools."
- [discussion] Runtime limitation: "Chord requires the integration of the results of multiple methods, so it is not optimal in terms of time efficiency."
