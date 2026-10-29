---
name: benchmark-comparison-against-published-metrics
description: Use when a paper reports headline metrics (e.g. MCC/F1 on named test
  datasets) and you have access to the model or code deposit, so you can recompute
  metrics and diff them against the published benchmark values.
when_to_use_negative:
- No model artifact or deposit exists and only a web server is available (e.g. LncCat's
  web server returns results only by email, so exact reproduction is not possible)
- The target paper omits key settings (e.g. CatBoost/BERT versions, hyperparameters),
  making a faithful re-run and honest comparison infeasible
- Your inputs are new or out-of-distribution transcripts rather than the paper's named
  benchmark test sets — use model evaluation, not published-value comparison
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: LncCat
  role: 'Model under benchmark: its reported MCC/F1 values on integrated and sORF
    test sets are the reference numbers to reproduce and compare against'
  repo: https://github.com/a525076133/LncCat
- name: CatBoost
  role: Gradient-boosting classifier used to retrain/refit the LncCat model whose
    test metrics are recomputed
  repo: https://github.com/a525076133/LncCat
- name: RNAsamba
  role: Competing lncRNA predictor whose published benchmark scores are compared against
    LncCat's in the metric comparison
  repo: http://github.com/apcamargo/RNAsamba
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:c2d2cfee405024d0b49ad71995a5a6456fe35124b6df5014a6361560233879c0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/a525076133/LncCat
  repo_url: https://github.com/a525076133/LncCat
  tools:
  - CatBoost
  - LncCat
  - RNAsamba
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# benchmark-comparison-against-published-metrics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce a trained model's evaluation on benchmark test sets (long-ORF and sORF) and quantitatively compare computed metrics (PRE, SEN, SPE, ACC, F1, MCC, AUC) against the values published in the source paper, e.g. LncCat's integrated MCC of 0.9236 and sORF-integrated MCC of 0.8346.

## When to use

Use when a paper reports headline metrics (e.g. MCC/F1 on named test datasets) and you have access to the model or code deposit, so you can recompute metrics and diff them against the published benchmark values.

## When NOT to use

- No model artifact or deposit exists and only a web server is available (e.g. LncCat's web server returns results only by email, so exact reproduction is not possible)
- The target paper omits key settings (e.g. CatBoost/BERT versions, hyperparameters), making a faithful re-run and honest comparison infeasible
- Your inputs are new or out-of-distribution transcripts rather than the paper's named benchmark test sets — use model evaluation, not published-value comparison

## Inputs

- Trained model artifact or training script from the paper's code deposit (e.g. https://github.com/a525076133/LncCat)
- Precomputed feature matrix for the five-species training set (human, mouse, zebrafish, wheat, chicken)
- Integrated long-ORF test dataset and sORF-integrated test dataset with lncRNA/PCT labels
- Published benchmark metric values from the paper's tables (e.g. Table 14 MCC 0.9236; Table 15 MCC 0.8346)

## Outputs

- integrated_model_evaluation.csv with PRE, SEN, SPE, ACC, F1, MCC, and AUC per test set
- Recomputed-vs-published metric comparison table with pass/fail per reported value

## How to apply

Load the deposited model and precomputed feature matrices (e.g. the five-species codon-related, GC-related, sequence-related, peptide-related, and ORF-attention features from the LncCat GitHub deposit), then apply the trained CatBoost classifier to the specified test sets (integrated long-ORF test data and sORF-integrated test data). Fix the decision threshold exactly as published — for LncCat the cutoff is 0.5, with score >= 0.5 predicted as lncRNA and < 0.5 as PCT — so the comparison is apples-to-apples. Compute PRE, SEN, SPE, ACC, F1, MCC, and AUC with lncRNAs as the positive class and PCTs as the negative class, write them to a machine-readable table (e.g. integrated_model_evaluation.csv), and assert that recomputed values match the reported ones: integrated MCC 0.9236 (F1 0.9537, Table 14) and sORF-integrated MCC 0.8346 (F1 0.9245, Table 15). Additionally compare against competing methods' values (CPAT, CNCI, PLEK, CPC2, LncFinder, CPPred, mRNN, RNAsamba) to verify the claimed margins, such as MCC improvements of at least 11.90%, 12.96%, and 42.61% on human, mouse, and zebrafish sORF test datasets. Treat mismatches as diagnostics: check threshold, class polarity, dataset version, and train/validation/test split (70/10/20) before concluding non-reproducibility.

## Related tools

- **LncCat** (Model under benchmark: its reported MCC/F1 values on integrated and sORF test sets are the reference numbers to reproduce and compare against) — https://github.com/a525076133/LncCat
- **CatBoost** (Gradient-boosting classifier used to retrain/refit the LncCat model whose test metrics are recomputed) — https://github.com/a525076133/LncCat
- **RNAsamba** (Competing lncRNA predictor whose published benchmark scores are compared against LncCat's in the metric comparison) — http://github.com/apcamargo/RNAsamba

## Examples

```
from sklearn.metrics import matthews_corrcoef, f1_score; preds = (scores >= 0.5).astype(int); print(matthews_corrcoef(y_true, preds), f1_score(y_true, preds))
```

## Evaluation signals

- Integrated long-ORF test MCC equals the reported 0.9236 with F1 0.9537 (Table 14)
- sORF-integrated test MCC equals the reported 0.8346 with F1 0.9245 (Table 15)
- Prediction threshold is fixed at 0.5 (score >= 0.5 = lncRNA, < 0.5 = PCT), matching the paper's stated cutoff
- Recomputed sORF margins exceed the best compared method by at least the published percentages (human 11.90%, mouse 12.96%, zebrafish 42.61% MCC improvement)
- Output CSV contains all seven metrics (PRE, SEN, SPE, ACC, F1, MCC, AUC) for both test sets with lncRNA as positive class

## Limitations

- CatBoost hyperparameter tuning settings and CatBoost/BERT versions are not reported, so exact reproduction may be unattainable
- Species lacking annotations cannot be benchmarked, and models cannot be built for all species
- README-reported sORF improvement percentages (6.88%, 22.98%, 39.43%) differ from the abstract card values (11.90%, 12.96%, 42.61%), so verify which figures the paper version supports

## Evidence

- [readme] Published MCC benchmarks across species: "The Matthew’s Correlation Coefficient (MCC) of LncCat achieves 0.9503, 0.9219, 0.8591, 0.8672 and 0.9047 on the human, mouse, zebrafish"
- [methods] Decision cutoff used in evaluation: "the cutoff value is set at 0.5, which means that the prediction score is greater than or equal to 0.5 for lncRNA and less than 0.5 for PCT"
- [abstract] sORF benchmark margins claimed by the paper: "LncCat dramatically improves the MCC by at least 11.90%, 12.96% and 42.61% on sORF test datasets of human, mouse, and zebrafish"
- [methods] Dataset used for the benchmark comparison: [evidence span withheld — anchor status: not_found]
