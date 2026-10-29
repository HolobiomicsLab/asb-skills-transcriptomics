---
name: binary-classifier-performance-metrics
description: Use when several tuned binary classifiers (e.g. LR, DT, SVM, RF, XGBoost)
  must be ranked on a held-out labeled test set to decide which algorithm a tool should
  adopt, as in LncDC's five-algorithm selection on H-Test.
when_to_use_negative:
- Input is a feature table still needing model training and SMOTE balancing — the
  metric evaluation comes after tenfold CV tuning, not before.
- The task is ranking feature sets or ablation variants (e.g. SIF+PF vs SIF+PF+SSF)
  on the same model rather than selecting among different algorithms.
- Predictions are multi-class or the labels are not the binary lncRNA/mRNA classes;
  MCC and the specificity/sensitivity panel as configured assume a two-class problem.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: LncDC
  role: Tool whose final XGBoost model is selected based on the metric comparison;
    predictions on H-Test provide the labels to score
  repo: https://github.com/lim74/LncDC
- name: XGBoost
  role: One of the five compared algorithms; the winner (accuracy 0.9800, AUC 0.9974
    on H-Test) becomes the LncDC model
- name: scikit-learn
  role: Provides the LR, DT, SVM, RF implementations and the tenfold cross-validated
    grid search used to produce the models being scored
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:56226f8014dc068b5e01bc3a00bdba887ac39e88e452d4cf5b202fefe920ab8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lim74/LncDC
  tools:
  - LncDC
  - Python
  - XGBoost
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# binary-classifier-performance-metrics

## Summary

Computes and compares a six-metric panel (accuracy, AUC, sensitivity, specificity, precision, MCC) for binary lncRNA/mRNA classifiers evaluated on a held-out test set (H-Test), producing the ranked comparison table used to select the final algorithm for LncDC.

## When to use

Use when several tuned binary classifiers (e.g. LR, DT, SVM, RF, XGBoost) must be ranked on a held-out labeled test set to decide which algorithm a tool should adopt, as in LncDC's five-algorithm selection on H-Test.

## When NOT to use

- Input is a feature table still needing model training and SMOTE balancing — the metric evaluation comes after tenfold CV tuning, not before.
- The task is ranking feature sets or ablation variants (e.g. SIF+PF vs SIF+PF+SSF) on the same model rather than selecting among different algorithms.
- Predictions are multi-class or the labels are not the binary lncRNA/mRNA classes; MCC and the specificity/sensitivity panel as configured assume a two-class problem.

## Inputs

- H-Test FASTA transcripts with ground-truth lncRNA/mRNA labels
- Tuned model objects or prediction files from the five candidate classifiers (LR, DT, SVM, RF, XGBoost)
- LncDC prediction CSV in which long noncoding RNAs are labeled 'lncrna' and messenger RNAs are labeled 'mrna' (output of lncDC.py -o)

## Outputs

- Comparison table of accuracy, AUC, sensitivity, specificity, precision, and MCC per algorithm on H-Test
- Ranked selection of the best-performing algorithm (XGBoost in the LncDC paper) with its metric values (accuracy 0.9800, AUC 0.9974)

## How to apply

Build the feature matrix for each candidate algorithm from the H-Train mRNA (with paired CDS) and lncRNA FASTA sequences using the lncDC-train.py pipeline (57 extracted features: sequence intrinsic features, secondary structure features, and protein features), then balance it with SMOTE over-sampling so no minority-class transcripts are discarded. Train LR, DT, SVM, RF, and XGBoost from scikit-learn and xgboost with tenfold cross-validated hyperparameter grid search, then freeze each tuned model and apply it to H-Test. For each model compute accuracy, AUC, sensitivity, specificity, precision, and MCC; for AUC use the predicted class scores, and for the others use the predicted labels ('lncrna' vs 'mrna') against H-Test ground truth. Tabulate all six metrics per algorithm, rank the algorithms, and select the top performer — in the paper this was XGBoost, which won in all criteria on H-Test with accuracy 0.9800 and AUC 0.9974, and was therefore adopted in the LncDC program. Acceptance check: the selected algorithm must outperform the alternatives across the metric panel, not just on accuracy.

## Related tools

- **LncDC** (Tool whose final XGBoost model is selected based on the metric comparison; predictions on H-Test provide the labels to score) — https://github.com/lim74/LncDC
- **XGBoost** (One of the five compared algorithms; the winner (accuracy 0.9800, AUC 0.9974 on H-Test) becomes the LncDC model)
- **scikit-learn** (Provides the LR, DT, SVM, RF implementations and the tenfold cross-validated grid search used to produce the models being scored)

## Examples

```
python bin/lncDC.py -i test/human_test_toy.fasta -o lncdc_human.csv -t 8  # score lncdc_human.csv labels against H-Test truth with sklearn.metrics (accuracy, roc_auc_score, matthews_corrcoef, ...)
```

## Evaluation signals

- All six metrics (accuracy, AUC, sensitivity, specificity, precision, MCC) are reported for each of the five algorithms, not just accuracy and AUC.
- The top-ranked algorithm should win across all criteria — replicating the paper's result means XGBoost leads in every metric with accuracy 0.9800 and AUC 0.9974 on H-Test.
- Metric values are computed on the held-out H-Test transcripts with ground-truth lncRNA/mRNA labels, never on the SMOTE-balanced training matrix.
- For a model retrained with only the 6 selected SSFs, the reported H-Test reference points are precision 0.8929 and specificity 0.8998.
- AUC is derived from class scores while the other metrics use the binary 'lncrna'/'mrna' labels emitted in the CSV output.

## Limitations

- Further accuracy improvement over the base model was difficult because SIFs and PFs already achieved a high accuracy, limiting headroom for the SSF-enhanced variant.
- The enhancement from the 6 selected secondary structure features is acknowledged as marginal ('not a giant step'), so small metric differences between feature sets may not be meaningful.
- The five-algorithm comparison used a single human test dataset (H-Test); generalization across species or datasets is not established by this skill alone.

## Evidence

- [discussion] Five popular algorithms, LR, DT, SVM, RF, and XGBoost were compared in this study. Because XGBoost outperformed other algorithms in performance evaluation, it was used in the LncDC program: "Five popular algorithms, LR, DT, SVM, RF, and XGBoost were compared in this study. Because XGBoost outperformed other algorithms in performance evaluation, it was used in the LncDC program"
- [discussion] the LncDC model with only the 6 selected SSFs performed well in H-Test (precision:0.8929, specificity:0.8998): "the LncDC model with only the 6 selected SSFs performed well in H-Test (precision:0.8929, specificity:0.8998)"
- [abstract] Benchmarking experiments showed that LncDC consistently outperformed six state-of-the-art tools in distinguishing lncRNAs from mRNAs: "Benchmarking experiments showed that LncDC consistently outperformed six state-of-the-art tools in distinguishing lncRNAs from mRNAs"
- [readme] The output file that will contain the prediction results in csv format. Long noncoding RNAs are labeled as 'lncrna', and message RNAs are labeled as 'mrna'.: "The output file that will contain the prediction results in csv format. Long noncoding RNAs are labeled as 'lncrna', and message RNAs are labeled as 'mrna'."
