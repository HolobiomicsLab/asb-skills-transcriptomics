---
name: majority-vote-sample-level-label-aggregation
description: Use when you have per-cell classification predictions across a sample
  and need a single sample-level call (e.g. predicted primary origin of a metastatic
  biopsy) from the majority cancer-cell annotation.
when_to_use_negative:
- Samples expected to be normal/non-cancerous tissue — the default scATOMIC model
  assumes presence of at least some cancer cells and should not be used on normal
  tissue
- Cases where a per-cell heterogeneous mixture must be preserved (e.g. TME composition
  analysis) rather than collapsed to a single sample-level label
- When the cancer type is already known — create_summary_matrix() has a known_cancer_type
  parameter instead of majority-vote inference
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scATOMIC
  role: Per-cell hierarchical pan-cancer classifier whose terminal cancer-type layer
    predictions are majority-voted per sample
  repo: https://github.com/abelson-lab/scATOMIC
- name: cutoff.scATOMIC
  role: Forked cutoff package used by scATOMIC to compute automatic confidence cutoffs
    that separate confidently classified cells before majority aggregation
  repo: https://github.com/inofechm/cutoff.scATOMIC
- name: Rmagic
  role: MAGIC imputation of expression values prior to scATOMIC classification feeding
    the aggregated labels
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:30dcc9cfab9c6151a06e9a97c845e5eba5d0a5e2cb0e5c3758e68d44918ff7da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abelson-lab/scATOMIC
  tools:
  - R
  - Rmagic
  - cutoff.scATOMIC
  - devtools
  - reticulate
  - scATOMIC
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# majority-vote-sample-level-label-aggregation

## Summary

Derives a sample-level label by taking the majority annotation among cells classified as a class of interest (e.g. cancer cells) after running a per-cell classifier like scATOMIC, used to predict the primary tissue of origin of each metastatic biopsy.

## When to use

Use when you have per-cell classification predictions across a sample and need a single sample-level call (e.g. predicted primary origin of a metastatic biopsy) from the majority cancer-cell annotation.

## When NOT to use

- Samples expected to be normal/non-cancerous tissue — the default scATOMIC model assumes presence of at least some cancer cells and should not be used on normal tissue
- Cases where a per-cell heterogeneous mixture must be preserved (e.g. TME composition analysis) rather than collapsed to a single sample-level label
- When the cancer type is already known — create_summary_matrix() has a known_cancer_type parameter instead of majority-vote inference

## Inputs

- Per-cell classification results from run_scATOMIC() for each of 62 metastatic biopsies (Supplementary Data 6 gene-by-cell sparse count matrices)
- Known primary tumour type annotations per sample (Supplementary Data 6)
- create_summary_matrix() parameters: use_CNVs = F, modify_results = TRUE, min_prop = 0.5

## Outputs

- One predicted primary-origin label per sample (majority cancer-cell annotation at the terminal cancer-type hierarchy layer)
- Per-sample tally of correct vs incorrect origin calls and samples abstained to intermediate (unresolved) classes

## How to apply

Run the per-cell classifier (scATOMIC via run_scATOMIC()) on each biopsy individually — README guidance recommends running on individual patient/biopsy samples because inter-patient cancer-cell-specific effects interfere with cancer signature scoring — then aggregate intermediate layer predictions with create_summary_matrix() using modify_results = TRUE to enforce a single cancer type per sample and min_prop = 0.5. For each sample, derive the predicted primary origin as the majority cancer-cell annotation among cells assigned to the terminal cancer-type layer of the hierarchy. Tally correct calls against known primary tumour types, and record samples where the classifier abstained to an intermediate (non-terminal) class as unresolved rather than forcing a terminal label.

## Related tools

- **scATOMIC** (Per-cell hierarchical pan-cancer classifier whose terminal cancer-type layer predictions are majority-voted per sample) — https://github.com/abelson-lab/scATOMIC
- **cutoff.scATOMIC** (Forked cutoff package used by scATOMIC to compute automatic confidence cutoffs that separate confidently classified cells before majority aggregation) — https://github.com/inofechm/cutoff.scATOMIC
- **Rmagic** (MAGIC imputation of expression values prior to scATOMIC classification feeding the aggregated labels)

## Examples

```
sm <- create_summary_matrix(query_mat, models = scATOMIC_models, use_CNVs = F, modify_results = TRUE, min_prop = 0.5)
```

## Evaluation signals

- Majority-vote origin calls must reproduce the reported benchmark: 52 of 62 metastatic biopsies (83.9%) correct primary-origin predictions across breast, kidney, lung, ovarian, and skin cancers
- 3 additional samples should resolve as abstentions to the correct intermediate (non-terminal) class rather than terminal calls
- create_summary_matrix() with modify_results = TRUE must enforce exactly one cancer type per sample
- Every sample must receive either a terminal cancer-type label or an explicit intermediate-class abstention — no sample should silently lack a classification

## Limitations

- Additional work is required to evaluate the limits of single-cell transcriptomics to predict cancer origin, specifically in cancers of unknown primary and mucinous ovarian carcinoma
- Some non-validated cancer classes (e.g. bile duct, endometrial, gastric) may be misclassified as other related cancers, corrupting majority-vote sample calls
- The default scATOMIC model assumes the presence of at least some cancer cells in a biopsy and should not be used on normal tissue samples

## Evidence

- [methods] modify_results / min_prop aggregation and per-sample labelling: "After running scATOMIC we generate a summary of the intermediate results with create_summary_matrix()."
- [abstract] ScATOMIC abstains from terminal classification yet focuses on the correct intermediate class: [evidence span withheld — anchor status: not_found]
- [readme] Individual-biopsy running rationale: "We strongly recommend running scATOMIC only on individual patient/biopsy samples."
- [readme] Default model assumes cancer cells present: "The default scATOMIC model assumes the presence of at least some cancer cells in a biopsy and should not be used in normal tissue applications."
- [discussion] Limitation on cancers of unknown primary: "Additional work is required to evaluate the limits of single-cell transcriptomics to predict cancer origin, specifically in cancers of unknown primary"
