---
name: xgboost-model-prediction
description: Use when RNA transcript sequences are in FASTA format and you need binary
  lncRNA vs mRNA classification with an alignment-free XGBoost model, including ablation
  conditions (full features vs SIF+PF only vs SSF-only).
when_to_use_negative:
- The input is already a numeric feature table rather than raw RNA FASTA sequences
  — LncDC extracts SIFs, SSFs, and PFs from sequences itself.
- You need de novo transcript assembly or alignment-based homology search (e.g. CPC-style
  protein database alignment); LncDC is explicitly alignment-free.
- Only coding-sequence classification with protein databases is needed and a simpler
  tool like CPC2/CPAT suffices — invoking RNAfold via -r is time-consuming and unnecessary.
edam_operation: http://edamontology.org/operation_2489
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3512
tools:
- name: LncDC
  role: The prediction program that loads prebuilt XGBoost model/imputer/scaler artifacts
    and classifies FASTA transcripts as lncrna or mrna
  repo: https://github.com/lim74/LncDC
- name: XGBoost
  role: Classifier underlying LncDC; selected because it outperformed LR, DT, SVM,
    and RF in performance evaluation
- name: ViennaRNA
  role: Optional dependency whose RNAfold computes secondary structures needed for
    SSF features when -r is enabled
- name: scikit-learn
  role: Pinned dependency (1.1.3) whose imputer/scaler objects are shipped as pickles
    and used to compute evaluation metrics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:85e23024f298f38674729c231265114f653af2046b06dc501c14099309728c0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lim74/LncDC
  tools:
  - LncDC
  - Python
  - ViennaRNA
  - XGBoost
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2489
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# xgboost-model-prediction

## Summary

Apply the prebuilt LncDC XGBoost models (XGB_model_SIF_PF.pkl and XGB_model_SIF_PF_SSF.pkl with paired imputer and scaler pickles) to classify RNA transcripts in a FASTA file as 'lncrna' or 'mrna', optionally enabling secondary structure features via RNAfold.

## When to use

Use when RNA transcript sequences are in FASTA format and you need binary lncRNA vs mRNA classification with an alignment-free XGBoost model, including ablation conditions (full features vs SIF+PF only vs SSF-only).

## When NOT to use

- The input is already a numeric feature table rather than raw RNA FASTA sequences — LncDC extracts SIFs, SSFs, and PFs from sequences itself.
- You need de novo transcript assembly or alignment-based homology search (e.g. CPC-style protein database alignment); LncDC is explicitly alignment-free.
- Only coding-sequence classification with protein databases is needed and a simpler tool like CPC2/CPAT suffices — invoking RNAfold via -r is time-consuming and unnecessary.

## Inputs

- RNA transcript sequences in FASTA format (regular or gzip-compressed .gz)
- Prebuilt XGBoost model pickle (e.g. XGB_model_SIF_PF.pkl or XGB_model_SIF_PF_SSF.pkl)
- Prebuilt imputer pickle (imputer_SIF_PF.pkl / imputer_SIF_PF_SSF.pkl)
- Prebuilt scaler pickle (scaler_SIF_PF.pkl / scaler_SIF_PF_SSF.pkl)
- Hexamer table CSV (e.g. train_hexamer_table.csv)
- Sequence and secondary structure k-mer table prefix (e.g. train_ss_table, files like train_ss_table_k1.csv through k5.csv) when -r is enabled

## Outputs

- Prediction CSV (e.g. lncdc_human.csv) labeling each transcript as 'lncrna' or 'mrna'
- Derived metrics (accuracy, precision, MCC, specificity) computed with scikit-learn for benchmarking or ablation tables

## How to apply

Activate the lncdc conda environment (python 3.9, xgboost 1.7.1, scikit-learn 1.1.3) and run bin/lncDC.py on the input FASTA with -i/-o; the default human model (XGB_model_SIF_PF.pkl, imputer_SIF_PF.pkl, scaler_SIF_PF.pkl, train_hexamer_table.csv) yields the SIF+PF-only prediction. To include secondary structure features, add -r and -k with the ss k-mer table prefix — this invokes RNAfold (ViennaRNA) and is markedly slower. For ablation-style conditions, swap the -m/-p/-s artifacts (e.g. XGB_model_SIF_PF_SSF.pkl with imputer_SIF_PF_SSF.pkl/scaler_SIF_PF_SSF.pkl) and, for SSF-only evaluation, restrict to the 6 selected SSFs; then compute accuracy, precision, MCC, and specificity in scikit-learn. The rationale is that XGBoost outperformed LR, DT, SVM, and RF in LncDC's model comparison, and features are extracted directly from sequences without alignment.

## Related tools

- **LncDC** (The prediction program that loads prebuilt XGBoost model/imputer/scaler artifacts and classifies FASTA transcripts as lncrna or mrna) — https://github.com/lim74/LncDC
- **XGBoost** (Classifier underlying LncDC; selected because it outperformed LR, DT, SVM, and RF in performance evaluation)
- **ViennaRNA** (Optional dependency whose RNAfold computes secondary structures needed for SSF features when -r is enabled)
- **scikit-learn** (Pinned dependency (1.1.3) whose imputer/scaler objects are shipped as pickles and used to compute evaluation metrics)

## Examples

```
python bin/lncDC.py -i test/human_test_toy.fasta -o lncdc_human.csv -t 8
```

## Evaluation signals

- Output CSV labels every transcript with exactly 'lncrna' or 'mrna'.
- SIF+PF-only ablation on H-Test should reproduce accuracy 0.9783, precision 0.9746, MCC 0.9567 (vs full-model baseline accuracy 0.9799, precision 0.9786, MCC 0.9598).
- SSF-only (6 selected SSFs) condition on H-Test should reach accuracy 0.8680, precision 0.8929, specificity 0.8998.
- Requirement check via python test_requirements.py passes in the lncdc environment before running.
- Importing RNA (ViennaRNA) succeeds without error when -r is specified.

## Limitations

- Prediction with secondary structure features (-r) may take a longer time because secondary structure calculation by RNAfold is time-consuming.
- The enhancement from the 6 selected SSFs is acknowledged as marginal, since SIFs and PFs had already achieved high accuracy and further improvement became difficult.
- For non-model organisms lacking well-annotated lncRNAs, a model must be trained with data from evolutionarily close model organisms; the prebuilt human model cannot simply be reused.
- Self-trained predictions require matching hexamer table, model, imputer, and scaler artifacts; mixing artifacts across feature sets is invalid.

## Evidence

- [discussion] XGBoost outperformed other algorithms: "Five popular algorithms, LR, DT, SVM, RF, and XGBoost were compared in this study. Because XGBoost outperformed other algorithms in performance evaluation, it was used in the LncDC program"
- [readme] README prediction parameters: "python lncDC.py -i input.fa -o output -x hexamer_table.csv -m model.pkl -p imputer.pkl -s scaler.pkl -r -k ss_table -t number_of_threads"
- [readme] README -r rationale: "This may take a longer time because secondary structure calculation by RNAfold (The main program of ViennaRNA) is time-consuming."
- [discussion] H-Test SSF-only performance: "the LncDC model with only the 6 selected SSFs performed well in H-Test (precision:0.8929, specificity:0.8998)"
- [readme] Non-model organism note: "you could train a model with the data from evolutionary closed model organisms, such as Zebrafish, Yeast, Soybean, *Drosophila melanogaster*"
