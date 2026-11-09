---
name: binary-classification-accuracy-evaluation
description: Use when comparing multiple binary classifiers (e.g., lncRNA prediction
  tools) on a common labeled test set and you need reproducible per-tool accuracy
  values for a benchmark table.
when_to_use_negative:
- When the goal is lncRNA discovery on unlabeled RNA-Seq transcript assemblies rather
  than benchmarking classifiers against known labels
- When classifiers were trained on the same transcripts being evaluated — this skill
  requires an independent test set to avoid optimistic accuracy
- When you need multi-class or regression evaluation; this skill covers two-class
  lncRNA vs mRNA classification only
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3474
tools:
- name: LncDC
  role: XGBoost-based lncRNA/mRNA classifier being benchmarked; run via bin/lncDC.py
    on H-Test FASTA
  repo: https://github.com/lim74/LncDC
- name: CPAT
  role: 'Competing tool: logistic regression model with several features; also re-trained
    on H-Train as an extra configuration'
- name: PLEK
  role: 'Competing tool: SVM with improved k-mer scheme, run with default prebuilt
    model'
- name: CPC2
  role: 'Competing tool: SVM integrating Fickett score, ORF length, ORF integrity
    and PI features, run with default prebuilt model'
- name: CPPred
  role: 'Competing tool: CTD feature-based predictor, run with default prebuilt model'
- name: LncFinder
  role: 'Competing tool: R package using ORF, secondary structure and physicochemical
    property features, run with default prebuilt model'
- name: COME
  role: 'Competing tool: integrates sequence-derived and experiment-based features,
    run with default prebuilt model'
- name: Python
  role: Environment for running LncDC and computing accuracies against known labels
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:0e888c99a1f57a54f6bc529d67e0592d5946ac8c80abce46866000ee6e257b10
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# binary-classification-accuracy-evaluation

## Summary

Benchmark lncRNA/mRNA binary classifiers by running each tool with its default prebuilt models on a shared FASTA test set and computing accuracy against known labels, as done when comparing LncDC to CPAT, PLEK, CPC2, CPPred, LncFinder, and COME on the human H-Test dataset.

## When to use

Use when comparing multiple binary classifiers (e.g., lncRNA prediction tools) on a common labeled test set and you need reproducible per-tool accuracy values for a benchmark table.

## When NOT to use

- When the goal is lncRNA discovery on unlabeled RNA-Seq transcript assemblies rather than benchmarking classifiers against known labels
- When classifiers were trained on the same transcripts being evaluated — this skill requires an independent test set to avoid optimistic accuracy
- When you need multi-class or regression evaluation; this skill covers two-class lncRNA vs mRNA classification only

## Inputs

- Human test dataset (H-Test) transcript sequences in FASTA format (mRNAs and lncRNAs)
- Known binary labels (lncrna/mrna) for each H-Test transcript
- Optional H-Train dataset FASTA for re-training a competing tool (e.g., CPAT logistic regression)

## Outputs

- Per-tool prediction CSV files with 'lncrna' or 'mrna' labels (e.g., lncdc_h_test.csv)
- benchmark_accuracy.csv with accuracy per tool configuration
- Ranking of tools showing whether LncDC outperforms the six competing tools

## How to apply

Assemble a common evaluation set of transcripts (e.g., H-Test human mRNAs and lncRNAs) in FASTA format so all tools are scored on identical inputs. Run each competing tool — PLEK (SVM with improved k-mer scheme), CPAT (logistic regression), CPC2 (SVM with Fickett score, ORF length, ORF integrity and PI features), CPPred (CTD features), LncFinder (R package with ORF, secondary structure and physicochemical property features), and COME (sequence-derived plus experiment-based features) — using each tool's default prebuilt models, and optionally re-train one tool (e.g., CPAT on H-Train) as an additional configuration. Run LncDC on the same FASTA file with 'python bin/lncDC.py -i <h_test.fasta> -o lncdc_h_test.csv -t 8', which labels each transcript 'lncrna' or 'mrna'. Compute accuracy for each of the seven tool configurations by comparing predictions to known H-Test labels, and verify reproduced values against reported targets: PLEK 0.9387, CPAT 0.9527, re-trained CPAT 0.9578, COME 0.9636, CPPred 0.9652, and LncDC 0.9799 (sensitivity 0.9812, specificity 0.9786, precision 0.9786, F-score 0.9799, MCC 0.9598). Compile values into benchmark_accuracy.csv.

## Related tools

- **LncDC** (XGBoost-based lncRNA/mRNA classifier being benchmarked; run via bin/lncDC.py on H-Test FASTA) — https://github.com/lim74/LncDC
- **CPAT** (Competing tool: logistic regression model with several features; also re-trained on H-Train as an extra configuration)
- **PLEK** (Competing tool: SVM with improved k-mer scheme, run with default prebuilt model)
- **CPC2** (Competing tool: SVM integrating Fickett score, ORF length, ORF integrity and PI features, run with default prebuilt model)
- **CPPred** (Competing tool: CTD feature-based predictor, run with default prebuilt model)
- **LncFinder** (Competing tool: R package using ORF, secondary structure and physicochemical property features, run with default prebuilt model)
- **COME** (Competing tool: integrates sequence-derived and experiment-based features, run with default prebuilt model)
- **Python** (Environment for running LncDC and computing accuracies against known labels)

## Examples

```
python bin/lncDC.py -i h_test.fasta -o lncdc_h_test.csv -t 8
```

## Evaluation signals

- Reproduced accuracies match reported targets: PLEK 0.9387, CPAT 0.9527, re-trained CPAT 0.9578, COME 0.9636, CPPred 0.9652, LncDC 0.9799
- LncDC H-Test metrics within reported ranges: sensitivity 0.9812, specificity 0.9786, precision 0.9786, F-score 0.9799, MCC 0.9598
- Every tool is evaluated on the identical H-Test FASTA set so accuracies are directly comparable
- LncDC predictions output labels are limited to 'lncrna' or 'mrna' and match the row count of the input FASTA
- LncDC accuracy exceeds all six competing tools on both human and mouse test datasets

## Limitations

- PLEK's accuracy gap partly reflects training-data differences; comparisons rely on each tool's default prebuilt models rather than uniformly retrained models
- Secondary-structure-based prediction with LncDC (-r) is much slower because RNAfold secondary structure calculation is time-consuming
- Further accuracy improvement was difficult because SIFs and PFs had already achieved high accuracy, so gains from added SSFs may be marginal

## Evidence

- [abstract] Benchmarking experiments showed that LncDC consistently outperformed six state-of-the-art tools in distinguishing lncRNAs from mRNAs: "Benchmarking experiments showed that LncDC consistently outperformed six state-of-the-art tools in distinguishing lncRNAs from mRNAs"
- [methods] Run LncDC on H-Test: "python bin/lncDC.py -i test/human_test_toy.fasta -o lncdc_human.csv -t 8"
- [readme] Output labels: "The output file that will contain the prediction results in csv format. Long noncoding RNAs are labeled as 'lncrna', and message RNAs are labeled as 'mrna'."
- [readme] RNAfold cost: "This may take a longer time because secondary structure calculation by RNAfold (The main program of ViennaRNA) is time-consuming."
- [discussion] H-Test SSF-only performance: "the LncDC model with only the 6 selected SSFs performed well in H-Test (precision:0.8929, specificity:0.8998)"
