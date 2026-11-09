---
name: cancer-primary-origin-prediction-from-metastatic-scrnaseq
description: Use when scRNA-seq count matrices from metastatic tumour biopsies (e.g.
  breast, kidney, lung, ovarian, skin cancer metastases) lack a known primary and
  the goal is to infer each sample's cancer tissue of origin at the cancer-cell level.
when_to_use_negative:
- The sample is normal/non-cancerous tissue — the default scATOMIC model assumes at
  least some cancer cells are present; use the normal_tissue parameter instead
- The primary cancer type is already known — use the known_cancer_type parameter in
  create_summary_matrix rather than origin prediction
- Cells have already been QC-filtered by different thresholds (e.g. no cells with
  pct_mt ≥ 25 or nFeatureRNA ≤ 500 remain), making the mitochondrial/feature-count
  filtering step redundant
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_2640
tools:
- name: scATOMIC
  role: Hierarchical random-forest pan-cancer classifier run per biopsy via run_scATOMIC()
    to assign cancer tissue-of-origin labels
  repo: https://github.com/abelson-lab/scATOMIC
- name: Rmagic
  role: MAGIC imputation used by default inside run_scATOMIC() before classification
  repo: https://github.com/abelson-lab/scATOMIC
- name: cutoff.scATOMIC
  role: Forked cutoff package implementing EM mixture-model confidence cutoffs used
    for scATOMIC's abstention decisions
  repo: https://github.com/inofechm/cutoff.scATOMIC
- name: R
  role: Runtime environment (v4.3.2 used in the paper) for the scATOMIC pipeline and
    majority-vote aggregation
- name: reticulate
  role: Bridges Rmagic to the Python magic-impute package; install.magic() if pymagic
    is unavailable
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:4c1c87a7bf67d385f57b63b7f27bbbd6b5589f400202b19226d0f331af8eb792
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# cancer-primary-origin-prediction-from-metastatic-scrnaseq

## Summary

Predict the primary tissue of origin of cancer cells in metastatic biopsies by running scATOMIC's hierarchical random-forest pan-cancer classifier per sample and taking the majority cancer-cell annotation at the terminal cancer-type layer. Unlike CNV-based malignant-cell identification, this yields a tissue-of-origin call even for unknown primaries.

## When to use

Use when scRNA-seq count matrices from metastatic tumour biopsies (e.g. breast, kidney, lung, ovarian, skin cancer metastases) lack a known primary and the goal is to infer each sample's cancer tissue of origin at the cancer-cell level.

## When NOT to use

- The sample is normal/non-cancerous tissue — the default scATOMIC model assumes at least some cancer cells are present; use the normal_tissue parameter instead
- The primary cancer type is already known — use the known_cancer_type parameter in create_summary_matrix rather than origin prediction
- Cells have already been QC-filtered by different thresholds (e.g. no cells with pct_mt ≥ 25 or nFeatureRNA ≤ 500 remain), making the mitochondrial/feature-count filtering step redundant

## Inputs

- Gene-by-cell sparse count matrices for metastatic biopsies (e.g. Supplementary Data 6)
- Known primary tumour type labels per sample (for validation only)
- scATOMIC pre-trained pan-cancer reference models (19 common cancers, hierarchical random forests)

## Outputs

- Per-sample predicted primary tissue of origin (majority cancer-cell annotation)
- scATOMIC_pred final classification label per cell
- Layer-wise (intermediate and terminal) scATOMIC predictions for every cell
- List of samples abstained to non-terminal intermediate classes

## How to apply

Load the gene-by-cell sparse count matrices for each metastatic biopsy (e.g. Supplementary Data 6) and filter low-quality cells, removing cells with >25% mitochondrial reads (pct_mt < 25) or fewer than 500 expressed features (nFeatureRNA > 500). Run scATOMIC individually on each biopsy with run_scATOMIC() using default settings — MAGIC imputation via Rmagic, hierarchical random forest models, and automatic confidence cutoffs — because inter-patient cancer-cell effects interfere with the cancer signature scoring module, so per-sample classification is mandatory. Aggregate intermediate predictions with create_summary_matrix(use_CNVs = F, modify_results = TRUE, min_prop = 0.5) to enforce a single cancer type per sample and obtain the final scATOMIC_pred label plus layer-wise predictions. Derive each sample's predicted primary origin as the majority cancer-cell annotation among cells assigned to the terminal cancer-type layer, recording samples that abstain to a non-terminal intermediate class (e.g. focusing on the correct intermediate class) as unresolved. Validate by tallying correct origin calls against known primary tumour types, expecting ~84% accuracy (52/62 in the published benchmark).

## Related tools

- **scATOMIC** (Hierarchical random-forest pan-cancer classifier run per biopsy via run_scATOMIC() to assign cancer tissue-of-origin labels) — https://github.com/abelson-lab/scATOMIC
- **Rmagic** (MAGIC imputation used by default inside run_scATOMIC() before classification) — https://github.com/abelson-lab/scATOMIC
- **cutoff.scATOMIC** (Forked cutoff package implementing EM mixture-model confidence cutoffs used for scATOMIC's abstention decisions) — https://github.com/inofechm/cutoff.scATOMIC
- **R** (Runtime environment (v4.3.2 used in the paper) for the scATOMIC pipeline and majority-vote aggregation)
- **reticulate** (Bridges Rmagic to the Python magic-impute package; install.magic() if pymagic is unavailable)

## Examples

```
scATOMIC_pred <- create_summary_matrix(run_scATOMIC(sparse_count_mat), use_CNVs = F, modify_results = TRUE, min_prop = 0.5)
```

## Evaluation signals

- Majority-vote origin calls across the 62 metastatic biopsies reproduce 52/62 (83.9%) correct primary-origin predictions across breast, kidney, lung, ovarian, and skin cancers
- Samples where scATOMIC abstains from terminal classification (3 additional samples) should focus on the correct intermediate class rather than an arbitrary cancer type
- create_summary_matrix with min_prop = 0.5 and modify_results = TRUE yields exactly one cancer type per sample (no mixed terminal cancer labels)
- Per-cell QC confirms all cells satisfy pct_mt < 25 and nFeatureRNA > 500 before classification
- Predicted origins match the known primary tumour types in Supplementary Data 6 for the majority of samples

## Limitations

- Additional work is required to evaluate limits of single-cell transcriptomics for predicting cancer origin, specifically in cancers of unknown primary and mucinous ovarian carcinoma
- Unvalidated cancer classes (e.g. bile duct, bone, endometrial, esophageal, gallbladder, gastric) may be misclassified as other related cancers since they have not been tested on external datasets
- A degree of discordancy exists between scATOMIC and CNV inference; using both together is recommended, and CNV inference may fall short with sub-clones of variable CNV burden, near-diploid cancer cells, or few normal control cells

## Evidence

- [abstract] Metastatic origin prediction benchmark: [evidence span withheld — anchor status: not_found]
- [intro] CNV-based malignant-cell ID cannot give origin: "Solely relying on the presence of inferred CNVs to annotate malignant cells may lead to false negatives or undefined cells in cancers with minimal genomic structural variation or nearly diploid"
- [methods] QC filtering thresholds: "We recommend to remove any cells with more than 25% of reads mapping to mitochondrial genes or those with fewer than 500"
- [methods] Summary aggregation step: "After running scATOMIC we generate a summary of the intermediate results with create_summary_matrix()."
- [readme] Per-sample classification requirement: "We strongly recommend running scATOMIC only on individual patient/biopsy samples."
