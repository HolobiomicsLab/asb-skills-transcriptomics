---
name: cross-tool-prediction-comparison
description: Use when you hold a labeled RNA transcript FASTA test set (e.g., H-Test
  mRNAs vs lncRNAs) and need to verify or replicate reported accuracy differences
  across multiple lncRNA-coding-potential predictors rather than run just one tool.
when_to_use_negative:
- Your goal is to discover novel lncRNAs from RNA-Seq samples (e.g., the 180 TARGET
  osteosarcoma samples) rather than to compare classifiers on a labeled benchmark
  set.
- The input sequences are genomic contigs or proteins rather than transcript FASTA
  sequences — LncDC and the six competitors are trained on RNA transcripts.
- Transcripts lack known mRNA/lncRNA labels, so classification accuracy against ground
  truth cannot be computed.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0659
tools:
- name: LncDC
  role: Subject tool under benchmark; run via lncDC.py on H-Test with the default
    human XGBoost model (SIF + PF features)
  repo: https://github.com/lim74/LncDC
- name: CPAT
  role: Competitor; logistic regression model run with defaults and additionally re-trained
    on H-Train
- name: PLEK
  role: Competitor; SVM with improved k-mer scheme run with prebuilt model
- name: CPC2
  role: Competitor; SVM using Fickett score, ORF length, ORF integrity and PI features
- name: CPPred
  role: Competitor; CTD feature-based predictor run with prebuilt model
- name: LncFinder
  role: Competitor; R package using ORF, secondary structure and physicochemical property
    features
- name: COME
  role: Competitor; integrates sequence-derived and experiment-based features
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:8f9ac7454863d50cc02a885de36de740ee5c6fd7942b4c24fc382a8c9596b0f9
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
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-tool-prediction-comparison

## Summary

Reproduces a head-to-head accuracy benchmark of a new lncRNA identification tool (LncDC) against six state-of-the-art classifiers — CPAT, PLEK, CPC2, CPPred, LncFinder, and COME — on a common labeled human transcript test set (H-Test).

## When to use

Use when you hold a labeled RNA transcript FASTA test set (e.g., H-Test mRNAs vs lncRNAs) and need to verify or replicate reported accuracy differences across multiple lncRNA-coding-potential predictors rather than run just one tool.

## When NOT to use

- Your goal is to discover novel lncRNAs from RNA-Seq samples (e.g., the 180 TARGET osteosarcoma samples) rather than to compare classifiers on a labeled benchmark set.
- The input sequences are genomic contigs or proteins rather than transcript FASTA sequences — LncDC and the six competitors are trained on RNA transcripts.
- Transcripts lack known mRNA/lncRNA labels, so classification accuracy against ground truth cannot be computed.

## Inputs

- H-Test human transcript sequences in FASTA format (mRNAs and lncRNAs) with known ground-truth labels
- H-Train human transcript FASTA (for re-training the CPAT logistic regression model)
- Prebuilt/default models shipped with each of the six competitor tools

## Outputs

- Per-tool prediction CSV files labeling each transcript as 'lncrna' or 'mrna' (e.g. lncdc_h_test.csv)
- benchmark_accuracy.csv with accuracy per tool configuration (PLEK 0.9387, CPAT 0.9527, re-trained CPAT 0.9578, COME 0.9636, CPPred 0.9652, LncDC 0.9799)

## How to apply

Assemble a single common evaluation FASTA (H-Test human mRNAs and lncRNAs with known labels) and run each competing tool with its own default prebuilt model: PLEK (SVM with improved k-mer scheme), CPAT (logistic regression, optionally re-trained on H-Train), CPC2 (SVM using Fickett score, ORF length, ORF integrity and PI features), CPPred (CTD features), LncFinder (R package using ORF, secondary structure and physicochemical property features), and COME (sequence-derived plus experiment-based features). Then run the tool under test on the same FASTA, e.g. LncDC with `python bin/lncDC.py -i <h_test.fasta> -o lncdc_h_test.csv -t 8`, which labels each transcript 'lncrna' or 'mrna'. Parse every prediction CSV, compare labels against the known H-Test ground truth, compute classification accuracy (plus sensitivity/specificity/precision/F-score/MCC if desired) for all seven tool configurations, and write them into benchmark_accuracy.csv. Judge success by reproducing the target ordering: LncDC accuracy 0.9799 (MCC 0.9598) should exceed CPPred 0.9652, COME 0.9636, re-trained CPAT 0.9578, CPAT 0.9527, and PLEK 0.9387.

## Related tools

- **LncDC** (Subject tool under benchmark; run via lncDC.py on H-Test with the default human XGBoost model (SIF + PF features)) — https://github.com/lim74/LncDC
- **CPAT** (Competitor; logistic regression model run with defaults and additionally re-trained on H-Train)
- **PLEK** (Competitor; SVM with improved k-mer scheme run with prebuilt model)
- **CPC2** (Competitor; SVM using Fickett score, ORF length, ORF integrity and PI features)
- **CPPred** (Competitor; CTD feature-based predictor run with prebuilt model)
- **LncFinder** (Competitor; R package using ORF, secondary structure and physicochemical property features)
- **COME** (Competitor; integrates sequence-derived and experiment-based features)

## Examples

```
python bin/lncDC.py -i h_test.fasta -o lncdc_h_test.csv -t 8
```

## Evaluation signals

- Each tool's prediction CSV labels exactly one class ('lncrna' or 'mrna') per transcript, and every H-Test transcript receives a prediction.
- Reproduced accuracies fall on the reported values: PLEK 0.9387, CPAT 0.9527, re-trained CPAT 0.9578, COME 0.9636, CPPred 0.9652, LncDC 0.9799.
- LncDC's accuracy (and MCC 0.9598) exceeds all six competitor configurations — including the re-trained CPAT — in benchmark_accuracy.csv.
- All seven tool configurations are evaluated on the identical H-Test FASTA, so any accuracy difference is attributable to the classifier, not the input set.

## Limitations

- Competitors were run with default prebuilt models (except the explicit CPAT re-training on H-Train), which may not reflect each tool's best achievable performance.
- LncDC prediction with secondary structure features (-r) requires ViennaRNA and is much slower because RNAfold is time-consuming.
- Accuracy on other organisms may differ; for non-model organisms the README recommends training a model with data from evolutionarily close model organisms (e.g. Zebrafish, Yeast, Soybean, Drosophila melanogaster, C. elegans, Arabidopsis thaliana, Oryza sativa).

## Evidence

- [abstract] Benchmarking experiments showed that LncDC consistently outperformed six state-of-the-art tools in distinguishing lncRNAs from mRNAs: "Benchmarking experiments showed that LncDC consistently outperformed six state-of-the-art tools in distinguishing lncRNAs from mRNAs"
- [intro] We benchmarked the performance of LncDC against six existing tools (CPAT, PLEK, CPC2, CPPred, LncFinder, and COME): "We benchmarked the performance of LncDC against six existing tools (CPAT, PLEK, CPC2, CPPred, LncFinder, and COME)"
- [intro] CPAT differentiates mRNAs and lncRNAs by using a logistic regression model with several features: "CPAT differentiates mRNAs and lncRNAs by using a logistic regression model with several features"
- [intro] COME integrates sequence-derived and experiment-based features for lncRNA identification: "COME integrates sequence-derived and experiment-based features for lncRNA identification"
- [discussion] the LncDC model with only the 6 selected SSFs performed well in H-Test (precision:0.8929, specificity:0.8998): "the LncDC model with only the 6 selected SSFs performed well in H-Test (precision:0.8929, specificity:0.8998)"
- [readme] The output file that will contain the prediction results in csv format. Long noncoding RNAs are labeled as 'lncrna', and message RNAs are labeled as 'mrna'.: "The output file that will contain the prediction results in csv format. Long noncoding RNAs are labeled as 'lncrna', and message RNAs are labeled as 'mrna'."
