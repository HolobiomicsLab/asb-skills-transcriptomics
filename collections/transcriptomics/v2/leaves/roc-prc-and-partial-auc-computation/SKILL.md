---
name: roc-prc-and-partial-auc-computation
description: Use when benchmarking doublet detectors on experimentally annotated scRNA-seq
  ground truth and you need accuracy metrics (AUC, AUPRC, pAUC800/900/950/975) plus
  cross-dataset stability ranks.
when_to_use_negative:
- There are no experimentally annotated doublet labels for the dataset — the AUC/AUPRC/pAUC
  metrics require ground-truth doublet annotations and cannot be computed on unannotated
  data
- The goal is doublet removal itself (e.g., running chord(seu, doubletrate, overkill=TRUE,
  outname)) rather than evaluating detector accuracy
- Evaluating detectors on synthetic doublets only — the article's benchmark uses datasets
  with experimental annotations from real and synthetic data, and results may not
  transfer
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Chord
  role: Ensemble (GBM/AdaBoost) doublet detector under evaluation; its scores are
    compared against ground-truth annotations
  repo: github.com/13308204545/Chord
- name: ChordP
  role: Modular-port variant of Chord with additional method scores supplied via addmethods1/addmethods2,
    evaluated in the benchmark
  repo: github.com/13308204545/Chord
- name: DoubletFinder
  role: Standalone detector scored against annotations; run with pN=0.25 and per-dataset
    pK tuned via mean-variance-normalized bimodality coefficient
  repo: chris-mcginnis-ucsf/DoubletFinder
- name: scds
  role: Standalone SingleCellExperiment-based detector providing cxds and bcds scores
    for evaluation
  repo: kostkalab/scds
- name: Scrublet
  role: Standalone Python detector whose doublet scores are benchmarked against the
    ground-truth datasets
  repo: AllonKleinLab/scrublet
- name: Solo
  role: Standalone semi-supervised deep learning detector benchmarked in the comparison
  repo: calico/solo
- name: PRROC
  role: R package cited in the Chord repository for computing and visualizing precision-recall
    and ROC curves
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:2b7ae830950ff5da847ef5d94a9553746311a54e6e010010348797102126aad3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chord
  - ChordP
  - DoubletFinder
  - PRROC
  - R
  - Scrublet
  - Solo
  - adabag
  - gbm
  - remotes
  - scds
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# roc-prc-and-partial-auc-computation

## Summary

Score doublet-detection predictions against experimentally annotated ground-truth scRNA-seq datasets by computing AUC, AUPRC, and partial AUCs at fixed true-positive counts (800/900/950/975), then average ranks across datasets to assess both accuracy and stability.

## When to use

Use when benchmarking doublet detectors on experimentally annotated scRNA-seq ground truth and you need accuracy metrics (AUC, AUPRC, pAUC800/900/950/975) plus cross-dataset stability ranks.

## When NOT to use

- There are no experimentally annotated doublet labels for the dataset — the AUC/AUPRC/pAUC metrics require ground-truth doublet annotations and cannot be computed on unannotated data
- The goal is doublet removal itself (e.g., running chord(seu, doubletrate, overkill=TRUE, outname)) rather than evaluating detector accuracy
- Evaluating detectors on synthetic doublets only — the article's benchmark uses datasets with experimental annotations from real and synthetic data, and results may not transfer

## Inputs

- Per-cell doublet scores or binary doublet calls from each of the nine methods (Chord, ChordP, DoubletFinder, scds cxds, scds bcds, DoubletDetection, Scrublet, Solo, doubletCells)
- Experimentally annotated doublet labels for the seven ground-truth datasets (DM-A, DM-B, DM-C, HTO12, HTO8, DM-2.1, DM-2.2)
- True-positive count cutoffs at which partial AUC is computed: 800, 900, 950, 975

## Outputs

- AUC, AUPRC, and partial AUC (pAUC800, pAUC900, pAUC950, pAUC975) per method per dataset
- metrics_summary.csv assembling metrics across all 9 methods x 7 datasets
- Average rank across datasets per method (e.g., ChordP 2.285) for stability comparison against Table 1

## How to apply

Run each detector on the seven annotated datasets (DM-A, DM-B, DM-C, HTO12, HTO8, DM-2.1, DM-2.2), converting Seurat objects with creatSCE for SingleCellExperiment-based methods like scds (cxds/bcds with parameters cxds.ntop, cxds.binThresh, bcds.ntop, bcds.srat); for DoubletFinder fix pN at its default 0.25 and tune pK per dataset via the mean-variance-normalized bimodality coefficient. Score every method's continuous predictions against the experimental doublet annotations, computing AUC, AUPRC, and partial AUC at 800/900/950/975 true-positive counts — the partial AUCs evaluate the high-specificity region where actual doublet removal happens. Assemble all 9 methods x 7 datasets into metrics_summary.csv and compare against Table 1 reference values (e.g., ChordP average AUC 0.8132, Chord average AUC 0.801214286). Compute average rank across datasets as the stability signal, since no single method is a gold standard for every scenario.

## Related tools

- **Chord** (Ensemble (GBM/AdaBoost) doublet detector under evaluation; its scores are compared against ground-truth annotations) — github.com/13308204545/Chord
- **ChordP** (Modular-port variant of Chord with additional method scores supplied via addmethods1/addmethods2, evaluated in the benchmark) — github.com/13308204545/Chord
- **DoubletFinder** (Standalone detector scored against annotations; run with pN=0.25 and per-dataset pK tuned via mean-variance-normalized bimodality coefficient) — chris-mcginnis-ucsf/DoubletFinder
- **scds** (Standalone SingleCellExperiment-based detector providing cxds and bcds scores for evaluation) — kostkalab/scds
- **Scrublet** (Standalone Python detector whose doublet scores are benchmarked against the ground-truth datasets) — AllonKleinLab/scrublet
- **Solo** (Standalone semi-supervised deep learning detector benchmarked in the comparison) — calico/solo
- **PRROC** (R package cited in the Chord repository for computing and visualizing precision-recall and ROC curves)

## Evaluation signals

- ChordP average AUC across the seven datasets equals ~0.8132 and Chord average AUC ~0.801214286 when compared against Table 1 reference values
- ChordP shows the highest average rank across datasets (2.285), confirming stability as well as accuracy
- Partial AUCs are computed at exactly four fixed true-positive counts: 800, 900, 950, and 975
- metrics_summary.csv contains rows for all 9 methods x 7 datasets with no missing method-dataset pairs
- Per-dataset pK tuning for DoubletFinder via the mean-variance-normalized bimodality coefficient is documented and reproducible rather than using a fixed pK

## Limitations

- No single computational doublet detection method performs consistently well on all datasets, so single-dataset metrics can be misleading — stability must be judged by average rank across datasets
- Chord requires integrating results of multiple methods, so the overall pipeline is not optimal in terms of time efficiency
- The 'overkill' step improving other methods' performance is an unverified claim in the discussion
- Partial AUC values are sensitive to the chosen true-positive count cutoffs (800/900/950/975); results should be reported at all four cutoffs

## Evidence

- [results] Accuracy metrics supported by Table 1 benchmark: [evidence span withheld — anchor status: not_found]
- [abstract] Accuracy claim: "Chord had higher accuracy and stability than the individual approaches on different datasets containing real and synthetic data."
- [intro] No gold standard rationale: "no method can be considered as the gold standard for each scenario"
- [readme] Metric tooling: "PRROC: computing and visualizing precision-recall and receiver operating characteristic curves in R. Bioinformatics 31, 2595-2597."
- [discussion] Time efficiency caveat: "Chord requires the integration of the results of multiple methods, so it is not optimal in terms of time efficiency."
