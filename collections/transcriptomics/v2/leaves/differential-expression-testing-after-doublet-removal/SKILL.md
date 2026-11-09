---
name: differential-expression-testing-after-doublet-removal
description: Use when a doublet-contaminated scRNA-seq dataset with known or expected
  DEG labels is available and you need to quantify whether doublet removal (by Chord,
  DoubletFinder, scds, etc.) improves differential expression results.
when_to_use_negative:
- The dataset has no ground-truth or reference DEG labels, so TPR/TNR/accuracy cannot
  be computed against a clean-data reference.
- The input is a bulk RNA-seq or otherwise non-single-cell dataset where doublet artefacts
  do not apply.
- You only need doublet calls themselves rather than downstream DEG accuracy assessment
  — use the doublet detection skill instead.
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Chord
  role: Ensemble doublet-detection method whose filtered dataset (doublet barcodes
    from outname_doublets.csv) is one of the DEG-accuracy conditions being evaluated
  repo: https://github.com/13308204545/Chord
- name: DoubletFinder
  role: Individual competing doublet-detection method run with tuned pK; its filtered
    dataset provides a comparison DEG-accuracy condition
  repo: https://github.com/chris-mcginnis-ucsf/DoubletFinder
- name: scds
  role: Provides cxds/bcds doublet scores used to rank and remove the top 40% of cells
    for comparison DEG-accuracy conditions
  repo: https://github.com/kostkalab/scds
- name: MAST
  role: Statistical framework used, alongside Wilcoxon rank-sum, for differential
    expression testing on each filtered dataset
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:c52c0bd2cf465838eccf928a1ddbb445cbeb1db7f79f94e7926c082d9ed9221f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/13308204545/Chord
  tools:
  - Chord
  - DoubletFinder
  - MAST
  - R
  - adabag
  - gbm
  - remotes
  - scds
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# differential-expression-testing-after-doublet-removal

## Summary

Evaluates whether removing predicted doublets improves DEG detection accuracy by rerunning differential expression testing (Wilcoxon rank-sum and MAST) on doublet-filtered single-cell datasets and scoring TPR, TNR, and accuracy against known DEG labels.

## When to use

Use when a doublet-contaminated scRNA-seq dataset with known or expected DEG labels is available and you need to quantify whether doublet removal (by Chord, DoubletFinder, scds, etc.) improves differential expression results.

## When NOT to use

- The dataset has no ground-truth or reference DEG labels, so TPR/TNR/accuracy cannot be computed against a clean-data reference.
- The input is a bulk RNA-seq or otherwise non-single-cell dataset where doublet artefacts do not apply.
- You only need doublet calls themselves rather than downstream DEG accuracy assessment — use the doublet detection skill instead.

## Inputs

- Seurat object or SingleCellExperiment of the doublet-contaminated dataset (e.g. scDesign-simulated DataDEG with 40% labelled doublets)
- Doublet barcodes per method, e.g. outname_doublets.csv from Chord or ranked doublet scores from DoubletFinder/scds
- Known simulated-DEG labels for the dataset (ground truth)
- Clean (no-doublet) reference DEG results

## Outputs

- DEG calls per filtered dataset from Wilcoxon rank-sum and MAST tests
- Table of TPR, TNR, and accuracy of DEG calls per doublet-removal method relative to clean-data reference

## How to apply

First filter predicted doublets from the single-cell dataset — e.g. remove the barcodes listed in Chord's outname_doublets.csv, or remove the top 40% of cells ranked by each individual method's doublet score (DoubletFinder with tuned pK, scds cxds/bcds). For every filtered dataset, rerun differential expression testing with Wilcoxon rank-sum and MAST. Compute TPR, TNR, and accuracy of the DEG calls against known simulated-DEG labels, comparing each filtered result to the clean (no-doublet) reference results. On the synthetic DataDEG dataset (1,667 cells, 40% doublets, scDesign-simulated), the expected pattern is that DEG results after doublet removal are better than the contaminated data, with Chord-filtered data yielding TPR/TNR/accuracy closest to the clean-data reference.

## Related tools

- **Chord** (Ensemble doublet-detection method whose filtered dataset (doublet barcodes from outname_doublets.csv) is one of the DEG-accuracy conditions being evaluated) — https://github.com/13308204545/Chord
- **DoubletFinder** (Individual competing doublet-detection method run with tuned pK; its filtered dataset provides a comparison DEG-accuracy condition) — https://github.com/chris-mcginnis-ucsf/DoubletFinder
- **scds** (Provides cxds/bcds doublet scores used to rank and remove the top 40% of cells for comparison DEG-accuracy conditions) — https://github.com/kostkalab/scds
- **MAST** (Statistical framework used, alongside Wilcoxon rank-sum, for differential expression testing on each filtered dataset)

## Examples

```
R: chord(seu=seu,doubletrate=0.09,overkill=T,outname="DataDEG")  # then remove barcodes from DataDEG_doublets.csv and rerun DE testing with Wilcoxon/MAST
```

## Evaluation signals

- DEG results after doublet removal should be better than the results on the contaminated (unfiltered) data
- Chord-filtered dataset should yield TPR/TNR/accuracy values closest to the clean-data (no-doublet) reference results
- Chord, DoubletFinder and DoubletDetection filtered results should cluster closer to clean-data results than weaker competing methods
- TPR, TNR, and accuracy must be computed per method against the known simulated-DEG labels for both Wilcoxon rank-sum and MAST tests

## Limitations

- Heterotypic doublets have the most impact on downstream analyses such as differential expression; homotypic doublet contamination may be less detectable by this evaluation
- Other pre-processing steps, such as ambient mRNA removal for droplets, should be considered to improve the accuracy of downstream analysis but are not incorporated in this evaluation
- Performance of individual doublet-detection methods varies greatly across datasets, so DEG-accuracy improvements after filtering are dataset-dependent

## Evidence

- [methods] DEG accuracy after filtering: [evidence span withheld — anchor status: not_found]
- [results] Expected result pattern: [evidence span withheld — anchor status: not_found]
- [intro] Motivation for DEG impact: "heterotypic doublets are considered to have more impact on downstream analyses, including dimensionality reduction, cell clustering, differential expression, and cell developmental trajectories"
- [abstract] Chord accuracy claim: "Chord had higher accuracy and stability than the individual approaches on different datasets containing real and synthetic data."
- [readme] Output barcodes file: "The doublets' barcodes are in the file "outname_doublets.csv""
