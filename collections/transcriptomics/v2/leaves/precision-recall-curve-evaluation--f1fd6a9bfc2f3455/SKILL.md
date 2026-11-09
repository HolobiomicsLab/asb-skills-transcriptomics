---
name: precision-recall-curve-evaluation
description: Use when you have ranked enhancer–gene interaction scores (e.g., STARE_ABCpp
  gABC or regular ABC output) and binary ground-truth labels from CRISPRi perturbation
  datasets, and need to quantify or compare predictive accuracy via AUPRC/ROC AUC.
when_to_use_negative:
- No binary ground-truth labels (e.g., CRISPRi perturbation effects) exist for the
  predicted enhancer–gene pairs, so precision cannot be defined
- Candidate interaction sets differ between the compared methods — only shared candidate
  interactions (e.g., <= 96 kb for Enformer ISM) can be fairly ranked and compared
- The scored interactions come from the candidate-enhancer discovery stage rather
  than from STARE_ABCpp ABC/gABC scoring, making the CRISPRi-labeled evaluation mismatched
edam_operation: http://edamontology.org/operation_0249
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: pROC
  role: R package used to compute and compare the precision-recall and ROC curves
    and AUPRC/ROC AUC for ABC vs. gABC on every dataset/assay/contact combination
- name: STARE
  role: STARE_ABCpp produces the scored enhancer–gene interactions (regular ABC with
    -q False/-i 5_tss; gABC with -q True/-i all_tss, -d True, -w 5MB, -t 0.02) that
    are evaluated by the PR curves
  repo: https://github.com/schulzlab/STARE
- name: stare-abc
  role: Bioconda package providing the ABC-scoring part of STARE used to generate
    the interaction scores under evaluation
  repo: http://bioconda.github.io/recipes/stare-abc/README.html
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:081a76a07d08878f77c1312fa0c1622697921aae7963d95e4935d4eff9e47e64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - STARE
  - pROC
  - stare-abc
  edam_operation: http://edamontology.org/operation_0249
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# precision-recall-curve-evaluation

## Summary

Evaluates enhancer–gene interaction prediction scores (regular ABC vs. gABC vs. competing methods such as Enformer in-silico mutagenesis) by computing precision-recall curves and AUPRC, plus ROC AUC, with the pROC R package against CRISPRi perturbation ground truth.

## When to use

Use when you have ranked enhancer–gene interaction scores (e.g., STARE_ABCpp gABC or regular ABC output) and binary ground-truth labels from CRISPRi perturbation datasets, and need to quantify or compare predictive accuracy via AUPRC/ROC AUC.

## When NOT to use

- No binary ground-truth labels (e.g., CRISPRi perturbation effects) exist for the predicted enhancer–gene pairs, so precision cannot be defined
- Candidate interaction sets differ between the compared methods — only shared candidate interactions (e.g., <= 96 kb for Enformer ISM) can be fairly ranked and compared
- The scored interactions come from the candidate-enhancer discovery stage rather than from STARE_ABCpp ABC/gABC scoring, making the CRISPRi-labeled evaluation mismatched

## Inputs

- STARE_ABCpp-scored enhancer–gene interaction files (regular ABC and/or gABC scores) per CRISPRi dataset
- CRISPRi perturbation effect measurements from Gasperini 2019, Schraivogel 2020, and Fulco 2019 used to define positive/negative labels
- Alternative method scores on the same candidate set (e.g., Enformer in-silico mutagenesis scores for interactions <= 96 kb)

## Outputs

- Precision-recall curves (and ROC curves) per dataset/assay/contact combination
- AUPRC and ROC AUC values per dataset and condition
- Comparison tables: auprc_comparison.csv, gabc_ablation_auprc.csv, gabc_enformer_pr_roc_comparison.csv

## How to apply

Label the predicted enhancer–gene pairs against the experimentally measured CRISPRi perturbation effects of each validation dataset (Gasperini 2019, Schraivogel 2020, Fulco 2019) to define positives and negatives. Rank the candidate interactions by each scoring method — e.g., regular ABC (STARE_ABCpp with -q False, -i 5_tss) versus gABC (-q True, -i all_tss) — computed across activity-assay (-n) and contact-matrix (-f, e.g., Fulco average hg19 Hi-C matrix or the Gschwind et al. hg38 ENCODE matrix ENCFF134PUN) combinations. Compute precision-recall curves and ROC curves for every dataset/assay/contact combination with the pROC package. Tabulate AUPRC per dataset and condition into a comparison CSV (e.g., auprc_comparison.csv), and average gains across datasets to compare against the reported values (~0.107 total gABC gain, ~0.026 for adapted activity alone, ~0.053 for all-TSSs alone). For a competing method, rank only the shared candidate interactions (e.g., Enformer ISM interactions <= 96 kb) before curve computation.

## Related tools

- **pROC** (R package used to compute and compare the precision-recall and ROC curves and AUPRC/ROC AUC for ABC vs. gABC on every dataset/assay/contact combination)
- **STARE** (STARE_ABCpp produces the scored enhancer–gene interactions (regular ABC with -q False/-i 5_tss; gABC with -q True/-i all_tss, -d True, -w 5MB, -t 0.02) that are evaluated by the PR curves) — https://github.com/schulzlab/STARE
- **stare-abc** (Bioconda package providing the ABC-scoring part of STARE used to generate the interaction scores under evaluation) — http://bioconda.github.io/recipes/stare-abc/README.html

## Examples

```
R: library(pROC); auc(roc(response = crispri_labels, predictor = gabc_scores))
```

## Evaluation signals

- gABC should yield on average a 0.107 higher AUPRC than the regular ABC score across the three CRISPRi validation datasets
- In the ablation, adapted activity alone should average ~0.026 AUPRC gain and all annotated TSSs alone ~0.053 over the regular ABC baseline
- gABC should outperform regular ABC on every validation dataset (Gasperini, Schraivogel, Fulco)
- gABC should attain higher ROC AUC than Enformer in-silico-mutagenesis scores on all three datasets, with significantly higher areas under the ROC curves
- Output CSVs must contain per-dataset AUPRC values plus the per-adaptation averages matching reported values within the reproducibility tolerance of the run

## Limitations

- The evaluation is anchored to CRISPRi perturbation ground truth (Gasperini, Schraivogel, Fulco); accuracy metrics are only meaningful where such perturbation measurements exist
- Model accuracy depends on the choice of activity-assay (-n: DNase-seq, ATAC-seq, H3K27ac ChIP-seq) and contact-matrix (-f: Fulco average hg19 Hi-C or Gschwind et al. hg38 ENCODE matrix ENCFF134PUN) combinations
- The model is comparably accurate with only one assay to measure enhancer activity, but multi-assay combinations were used for the reported averages
- Enformer ISM comparison is restricted to interactions <= 96 kb, so shared-candidate filtering must be applied before curve comparison

## Evidence

- [supplementary] pROC: An open-source package for r and s+ to analyze and compare ROC curves: "pROC: An open-source package for r and s+ to analyze and compare ROC curves"
- [abstract] ABC score accuracy gains from adapted activity and contacts: "the ABC score can yield a higher accuracy by adapting the enhancer activity according to the number of contacts"
- [readme] STARE ABC scoring role: "score enhancer-gene interactions with the Activity-By-Contact (ABC)"
- [abstract] single-assay caveat: "Further, we show that the model is comparably accurate with only one assay to measure enhancer activity."
