---
name: logistic-regression-model-retraining
description: Use when benchmarking an lncRNA identification tool whose shipped model
  (e.g., CPAT's logistic regression) was trained on a different dataset, and you have
  a labeled training set (H-Train) plus a test set (H-Test) to retrain and re-evaluate
  it.
when_to_use_negative:
- When the goal is to reproduce default prebuilt-model performance exactly for tool-as-shipped
  comparability — use the off-the-shelf CPAT model and report 0.9527 instead
- When no labeled training data (e.g., H-Train) exist for the target organism and
  the prebuilt model must be used as-is, such as for non-model organisms without well-annotated
  lncRNAs
- When the classifier being benchmarked is not logistic-regression-based (e.g., LncDC's
  XGBoost, PLEK's SVM, CPC2's SVM), where retraining follows each tool's own training
  procedure instead
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: CPAT
  role: Logistic regression classifier whose model is retrained on the H-Train dataset
    and re-evaluated on H-Test
- name: LncDC
  role: Reference XGBoost-based lncRNA detection tool (accuracy 0.9799 on H-Test)
    against which the re-trained CPAT baseline is compared
  repo: https://github.com/lim74/LncDC
- name: Python
  role: Implementation language used to run the retraining and prediction steps and
    compile benchmark_accuracy.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:41f51bc2d16fa09c4fd3712c8dcc938b45d2aa07701eaa63bb534ab4707bada4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lim74/LncDC
  tools:
  - COME
  - CPAT
  - CPC2
  - CPPred
  - LncDC
  - LncFinder
  - PLEK
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# logistic-regression-model-retraining

## Summary

Retrain CPAT's logistic regression lncRNA/mRNA classifier on the labeled H-Train human transcript set instead of relying on its default prebuilt model, so the benchmark reflects a fair, task-adapted baseline for the LncDC comparison on H-Test.

## When to use

Use when benchmarking an lncRNA identification tool whose shipped model (e.g., CPAT's logistic regression) was trained on a different dataset, and you have a labeled training set (H-Train) plus a test set (H-Test) to retrain and re-evaluate it.

## When NOT to use

- When the goal is to reproduce default prebuilt-model performance exactly for tool-as-shipped comparability — use the off-the-shelf CPAT model and report 0.9527 instead
- When no labeled training data (e.g., H-Train) exist for the target organism and the prebuilt model must be used as-is, such as for non-model organisms without well-annotated lncRNAs
- When the classifier being benchmarked is not logistic-regression-based (e.g., LncDC's XGBoost, PLEK's SVM, CPC2's SVM), where retraining follows each tool's own training procedure instead

## Inputs

- H-Train dataset: human mRNA and lncRNA transcript sequences in FASTA format with known labels (training set for the logistic regression model)
- H-Test dataset: human mRNA and lncRNA transcript sequences in FASTA format with known labels (evaluation set)
- CPAT logistic regression tooling (framework whose model coefficients are refit on H-Train)

## Outputs

- Re-trained CPAT logistic regression model trained on H-Train
- Per-transcript predictions ('lncrna'/'mrna'-equivalent labels) for H-Test
- Accuracy entry for re-trained CPAT (target: 0.9578) added to benchmark_accuracy.csv alongside default CPAT (0.9527)

## How to apply

CPAT classifies transcripts using a logistic regression model over sequence-derived features, but its prebuilt model was not trained on the benchmark's own data. To retrain, take the H-Train human dataset with known lncRNA/mRNA labels, fit the CPAT logistic regression model on those transcripts, then run the re-trained model on the H-Test FASTA sequences. Score predictions against known H-Test labels using accuracy (and sensitivity/specificity/precision/F-score/MCC where reported). Record the re-trained accuracy alongside the default-model accuracy in benchmark_accuracy.csv — the article reports re-trained CPAT at 0.9578 versus 0.9527 for the default model, showing that retraining on H-Train yields a stronger, fairer baseline against LncDC's 0.9799.

## Related tools

- **CPAT** (Logistic regression classifier whose model is retrained on the H-Train dataset and re-evaluated on H-Test)
- **LncDC** (Reference XGBoost-based lncRNA detection tool (accuracy 0.9799 on H-Test) against which the re-trained CPAT baseline is compared) — https://github.com/lim74/LncDC
- **Python** (Implementation language used to run the retraining and prediction steps and compile benchmark_accuracy.csv)

## Evaluation signals

- Re-trained CPAT accuracy on H-Test should reach 0.9578, exceeding the default prebuilt CPAT accuracy of 0.9527
- Predictions on H-Test must map to the two known classes (lncRNA vs mRNA) and match every transcript ID in the input FASTA
- The re-trained accuracy should still remain below LncDC's 0.9799, confirming LncDC outperforms all six competing tool configurations
- Retrained model must be fit only on H-Train — any leakage of H-Test sequences into training invalidates the comparison
- All seven tool configurations (PLEK 0.9387, CPAT 0.9527, re-trained CPAT 0.9578, COME 0.9636, CPPred 0.9652, LncFinder, LncDC 0.9799) should be compiled consistently into benchmark_accuracy.csv

## Limitations

- The accuracy gain from retraining (0.9527 → 0.9578) is modest, consistent with the article's observation that further improvement is difficult once features already achieve high accuracy
- Retraining is only defensible when adequate well-annotated training lncRNAs exist for the organism; the README notes that for non-model organisms, models may need to be trained on data from evolutionarily close model organisms (e.g., Zebrafish, Yeast, Soybean, Drosophila melanogaster, Caenorhabditis elegans, Arabidopsis thaliana, Oryza sativa)

## Evidence

- [intro] CPAT differentiates mRNAs and lncRNAs by using a logistic regression model with several features: "CPAT differentiates mRNAs and lncRNAs by using a logistic regression model with several features"
- [methods] Re-train the CPAT logistic regression model on the H-Train dataset and evaluate the re-trained CPAT on H-Test.: [evidence span withheld — anchor status: not_found]
- [results] verify the reproduced accuracies match the reported targets — PLEK 0.9387, CPAT 0.9527, re-trained CPAT 0.9578: [evidence span withheld — anchor status: not_found]
- [readme] could train a model with the data from evolutionary closed model organisms, such as Zebrafish, Yeast, Soybean, *Drosophila melanogaster*: "you could train a model with the data from evolutionary closed model organisms, such as Zebrafish, Yeast, Soybean, *Drosophila melanogaster*"
