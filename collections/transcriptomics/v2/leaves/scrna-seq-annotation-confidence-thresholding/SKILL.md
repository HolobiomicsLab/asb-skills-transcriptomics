---
name: scrna-seq-annotation-confidence-thresholding
description: Use when annotating scRNA-seq cells with a hierarchical classifier (e.g.
  scATOMIC run_scATOMIC() output) and you need per-class confidence cut-offs derived
  automatically from each class's score distribution rather than fixed thresholds.
when_to_use_negative:
- The sample is normal tissue without cancer cells — scATOMIC's default model assumes
  at least some cancer cells; use the normal_tissue parameter path instead
- You want a single global confidence threshold — this skill derives per-class, per-node
  thresholds from each class's own score distribution
- Input is batch-level pooled multi-patient data — scATOMIC's cancer signature scoring
  is confounded by inter-patient effects and should be run per patient/biopsy
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scATOMIC
  role: Hierarchical pan-cancer classifier whose IGS distributions are thresholded;
    provides run_scATOMIC(), score_class(), add_class_per_layer(), create_summary_matrix(),
    get_auto_threshold(), add_unclassified_automatic_threshold()
  repo: github.com/abelson-lab/scATOMIC
- name: agrmt
  role: Provides the 'ajus' criterion used to assess modality of each class's score
    distribution
- name: cutoff.scATOMIC
  role: Fork of the cutoff package; fits a finite mixture model (EM algorithm) to
    bimodal IGS distributions for cut-off determination
  repo: github.com/inofechm/cutoff.scATOMIC
- name: cutoff
  role: Original package implementing the mixture-model cut-off method (Trang et al.
    2015) that cutoff.scATOMIC forks
  repo: github.com/choisy/cutoff
- name: R
  role: Implementation language (v4.3.2 tested in methods)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:8830c10c5e32ce87930dd8e69f31571625725058547681fb5692172105d5f5be
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - agrmt
  - cutoff
  - cutoff.scATOMIC
  - scATOMIC
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# scrna-seq-annotation-confidence-thresholding

## Summary

Reconstructs scATOMIC's automatic per-node derivation of confidence cut-offs on intermediate group scores (IGS), so that cells scoring below a data-driven threshold retain their broader parental label (or become unclassified) instead of a terminal annotation.

## When to use

Use when annotating scRNA-seq cells with a hierarchical classifier (e.g. scATOMIC run_scATOMIC() output) and you need per-class confidence cut-offs derived automatically from each class's score distribution rather than fixed thresholds.

## When NOT to use

- The sample is normal tissue without cancer cells — scATOMIC's default model assumes at least some cancer cells; use the normal_tissue parameter path instead
- You want a single global confidence threshold — this skill derives per-class, per-node thresholds from each class's own score distribution
- Input is batch-level pooled multi-patient data — scATOMIC's cancer signature scoring is confounded by inter-patient effects and should be run per patient/biopsy

## Inputs

- Sparse gene-by-cell count matrix from a single patient/biopsy sample (e.g. Seurat object counts slot or as(..., 'sparseMatrix'))
- Per-layer prediction score lists from run_scATOMIC()
- Per-class IGS distributions extracted per parent node

## Outputs

- Per-class confidence cut-off values (capped at 0.7) for each parent node
- cutoff_table.csv with one row per parent-node class label and its cut-off
- Re-labelled annotation layers where sub-threshold cells become unclassified/retain parental labels

## How to apply

Run run_scATOMIC() on a sparse gene-by-cell count matrix (e.g. the packaged demo_lung_data lung cancer sample) to generate per-layer prediction scores. For each hierarchy layer/parent node, extract per-class cell scores with score_class() and add_class_per_layer(), mirroring the median_score_class_layer_* columns of create_summary_matrix(). Assess the modality of each class's IGS distribution with the 'ajus' criterion from the agrmt package. Then compute each class's cut-off with automatic_threshold(): mean minus 3 SD (unimodal_nsd = 3) for unimodal distributions, or second-component mean minus 2 SD (bimodal_nsd = 2) for bimodal distributions, capping any cut-off at a maximum of 0.7; batch all layers with get_auto_threshold(predictions, mc.cores, unimodal_nsd = 3, bimodal_nsd = 2). Apply the cut-offs with add_unclassified_automatic_threshold() so cells scoring below their class cut-off are relabelled unclassified at that layer, and write the per-parent-node table of class labels and cut-off values to cutoff_table.csv. The bimodal branch relies on cutoff.scATOMIC, a fork of choisy/cutoff that fits a finite mixture model via the EM algorithm to separate the two modes.

## Related tools

- **scATOMIC** (Hierarchical pan-cancer classifier whose IGS distributions are thresholded; provides run_scATOMIC(), score_class(), add_class_per_layer(), create_summary_matrix(), get_auto_threshold(), add_unclassified_automatic_threshold()) — github.com/abelson-lab/scATOMIC
- **agrmt** (Provides the 'ajus' criterion used to assess modality of each class's score distribution)
- **cutoff.scATOMIC** (Fork of the cutoff package; fits a finite mixture model (EM algorithm) to bimodal IGS distributions for cut-off determination) — github.com/inofechm/cutoff.scATOMIC
- **cutoff** (Original package implementing the mixture-model cut-off method (Trang et al. 2015) that cutoff.scATOMIC forks) — github.com/choisy/cutoff
- **R** (Implementation language (v4.3.2 tested in methods))

## Examples

```
thresholds <- get_auto_threshold(predictions, mc.cores = 8, unimodal_nsd = 3, bimodal_nsd = 2); predictions <- add_unclassified_automatic_threshold(predictions, thresholds)
```

## Evaluation signals

- cutoff_table.csv exists and contains one row per parent-node class with a cutoff field
- Every cut-off value is within [0, 0.7] (0.7 cap respected)
- Modality assignment matches the agrmt 'ajus' criterion: unimodal classes get mean − 3 SD, bimodal classes get second-component mean − 2 SD
- Applying the cut-offs converts sub-threshold cells to an unclassified label at the corresponding layer, retaining broader parental classification
- Per-layer cut-off columns align with the median_score_class_layer_* structure of create_summary_matrix()

## Limitations

- Inter-patient cancer cell-specific effects can interfere with scATOMIC's cancer signature scoring module and confidence in classifications, so it should be run only on individual patient/biopsy samples
- The default scATOMIC model assumes the presence of at least some cancer cells and should not be used on normal tissue samples (use the normal_tissue parameter in v2)
- The bimodal cut-off branch depends on an EM-fitted finite mixture model, which presumes a genuinely two-mode score distribution

## Evidence

- [readme] README cutoff.scATOMIC modality/mixture basis: [evidence span withheld — anchor status: external_doc]
- [readme] README cutoff.scATOMIC fork rationale: [evidence span withheld — anchor status: external_doc]
- [methods] Article methods run_scATOMIC: "To run scATOMIC we use the run_scATOMIC() function to generate a detailed annotation object."
- [methods] Article methods summary matrix: "After running scATOMIC we generate a summary of the intermediate results with create_summary_matrix()."
- [readme] README per-patient applicability: "Inter-patient cancer cell specific effects can interfere with scATOMICs cancer signature scoring module and confidence in classifications. We strongly recommend running scATOMIC only on individual"
