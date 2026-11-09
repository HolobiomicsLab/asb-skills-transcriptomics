---
name: random-forest-probability-score-summarization
description: Use when you have per-cell scATOMIC prediction scores from run_scATOMIC()
  across the pan-cancer TME hierarchy and need to convert raw class probabilities
  into per-layer confidence cut-offs and 'unclassified' relabelling.
when_to_use_negative:
- 'The sample is normal (non-cancer) tissue: the default scATOMIC model assumes the
  presence of at least some cancer cells, so use the normal_tissue parameter in create_summary_matrix()
  instead'
- 'You are pooling multiple patients/biopsies: scATOMIC''s cancer signature scoring
  and confidence are confounded by inter-patient cancer cell effects, so run per individual
  patient sample'
- Input is an already-summarized feature table or aggregated scores rather than per-cell
  count matrices
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scATOMIC
  role: Pre-trained hierarchical random forest models; run_scATOMIC() produces the
    per-layer prediction scores being summarized
  repo: github.com/abelson-lab/scATOMIC
- name: cutoff.scATOMIC
  role: Fork of the cutoff package implementing the EM-finite-mixture method for bimodal
    cut-off determination
  repo: github.com/inofechm/cutoff.scATOMIC
- name: agrmt
  role: Provides the 'ajus' criterion used to assess the modality of each class's
    IGS distribution before choosing unimodal vs bimodal cut-off rules
- name: Rmagic
  role: Used by scATOMIC (via reticulate) to impute values in the count matrix before
    score computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:17522f6f2245e1a3296e4820043e836f1806c99a3f5d464c3a6834c7cbc3daa7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Rmagic
  - agrmt
  - cutoff.scATOMIC
  - scATOMIC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Random forest probability score summarization

## Summary

Summarizes scATOMIC's hierarchical random-forest prediction scores (intermediate group scores, IGS) per hierarchy layer and parent node, then derives per-class confidence cut-offs from the score distributions so low-confidence cells retain a broader parental label instead of a terminal one.

## When to use

Use when you have per-cell scATOMIC prediction scores from run_scATOMIC() across the pan-cancer TME hierarchy and need to convert raw class probabilities into per-layer confidence cut-offs and 'unclassified' relabelling.

## When NOT to use

- The sample is normal (non-cancer) tissue: the default scATOMIC model assumes the presence of at least some cancer cells, so use the normal_tissue parameter in create_summary_matrix() instead
- You are pooling multiple patients/biopsies: scATOMIC's cancer signature scoring and confidence are confounded by inter-patient cancer cell effects, so run per individual patient sample
- Input is an already-summarized feature table or aggregated scores rather than per-cell count matrices

## Inputs

- Gene-by-cell raw count matrix (sparse matrix recommended) from a single patient/biopsy sample, e.g. demo_lung_data
- Per-layer prediction score lists returned by run_scATOMIC()
- Per-class cell score distributions at each parent node (score_class()/add_class_per_layer() output)

## Outputs

- cutoff_table.csv: one row per parent-node class with label and automatically derived cut-off value in [0, 0.7]
- Per-cell labels with sub-threshold cells relabelled unclassified at the corresponding hierarchy layer
- Per-layer median score summary columns (median_score_class_layer_* as in create_summary_matrix())

## How to apply

Load the gene-by-cell count matrix (preferably sparse, e.g. the packaged demo_lung_data lung cancer sample) and run run_scATOMIC() to generate per-layer prediction score lists. For each hierarchy layer/parent node, extract per-class cell scores with score_class() and add_class_per_layer(), mirroring the median_score_class_layer_* columns of create_summary_matrix(). Assess the modality of each class's score distribution with the 'ajus' criterion from the agrmt package: unimodal distributions get a cut-off of mean minus 3 SD (unimodal_nsd = 3), bimodal distributions get second-component mean minus 2 SD (bimodal_nsd = 2, via the EM-finite-mixture method in cutoff.scATOMIC), and every cut-off is capped at a maximum of 0.7. Apply the thresholds with add_unclassified_automatic_threshold() so cells scoring below their class cut-off are relabelled unclassified at that layer, and write the per-parent-node table of class labels and cut-offs to cutoff_table.csv.

## Related tools

- **scATOMIC** (Pre-trained hierarchical random forest models; run_scATOMIC() produces the per-layer prediction scores being summarized) — github.com/abelson-lab/scATOMIC
- **cutoff.scATOMIC** (Fork of the cutoff package implementing the EM-finite-mixture method for bimodal cut-off determination) — github.com/inofechm/cutoff.scATOMIC
- **agrmt** (Provides the 'ajus' criterion used to assess the modality of each class's IGS distribution before choosing unimodal vs bimodal cut-off rules)
- **Rmagic** (Used by scATOMIC (via reticulate) to impute values in the count matrix before score computation)

## Examples

```
cutoffs <- get_auto_threshold(predictions, mc.cores, unimodal_nsd = 3, bimodal_nsd = 2); annotated <- add_unclassified_automatic_threshold(cutoffs)
```

## Evaluation signals

- cutoff_table.csv exists and contains one row per parent-node class with a cutoff field
- Every cutoff value is within [0, 0.7] (the 0.7 maximum cap is enforced)
- Applying the cut-offs via add_unclassified_automatic_threshold() converts sub-threshold cells to an unclassified label at the corresponding layer
- Each class's cut-off rule matches the ajus-assessed modality: mean - 3 SD for unimodal, second-component mean - 2 SD for bimodal distributions
- Extracted per-class median scores agree with the median_score_class_layer_* columns produced by create_summary_matrix()

## Limitations

- Cells whose IGS falls below the threshold stop descending the hierarchy and keep their broader parental classification rather than a terminal label — fine-grained subtypes are lost for ambiguous cells
- Inter-patient cancer cell specific effects interfere with scATOMIC's cancer signature scoring module and confidence in classifications, so the summarization must be done per biopsy
- Default model should not be used on normal tissue samples where cancer cells are absent

## Evidence

- [readme] run_scATOMIC produces per-layer scores: "To run scATOMIC we use the run_scATOMIC() function to generate a detailed annotation object."
- [readme] create_summary_matrix summarizes intermediate results: "After running scATOMIC we generate a summary of the intermediate results with create_summary_matrix()."
- [readme] cutoff.scATOMIC fits EM mixture model for bimodal cutoff: [evidence span withheld — anchor status: external_doc]
- [intro] hierarchical elimination reduces transcriptomic complexity: "We devise a structured scheme that uses hierarchically organized models and elimination processes, reducing the transcriptomic complexity of the TME multi-cellular system to improve cell"
- [readme] recommend running on individual patient samples: "Inter-patient cancer cell specific effects can interfere with scATOMICs cancer signature scoring module and confidence in classifications."
