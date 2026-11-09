---
name: classification-metric-computation
description: Use when you have binary lncRNA/mrna prediction labels (e.g., LncDC CSV
  outputs or baseline tool outputs on H-Test/M-Test) and must quantify accuracy, precision,
  specificity, or MCC for benchmark tables or ablation comparisons.
when_to_use_negative:
- Input is continuous model scores/probabilities rather than binary lncrna/mrna class
  labels — the paper's metrics assume discrete predictions
- Evaluating regression or multi-class tasks; this skill is specific to two-class
  lncRNA-vs-mRNA classification
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: scikit-learn
  role: Computes accuracy, precision, specificity, and MCC from the binary prediction
    labels
- name: LncDC
  role: Produces the prediction CSVs (lncrna/mrna labels) whose metrics are being
    computed
  repo: https://github.com/lim74/LncDC
- name: Python
  role: Implementation language for the metric computation scripts
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:9d3e554584fa76961c5407d7e774c2346c4e7ba9ec81774fafccd5468ee590e6
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
  - ViennaRNA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# classification-metric-computation

## Summary

Computes accuracy, precision, specificity, and Matthews correlation coefficient (MCC) for lncRNA/mRNA binary predictions, as done in LncDC's benchmarking (H-Test, M-Test) and ablation experiments, using scikit-learn on the CSV prediction outputs.

## When to use

Use when you have binary lncRNA/mrna prediction labels (e.g., LncDC CSV outputs or baseline tool outputs on H-Test/M-Test) and must quantify accuracy, precision, specificity, or MCC for benchmark tables or ablation comparisons.

## When NOT to use

- Input is continuous model scores/probabilities rather than binary lncrna/mrna class labels — the paper's metrics assume discrete predictions
- Evaluating regression or multi-class tasks; this skill is specific to two-class lncRNA-vs-mRNA classification

## Inputs

- Prediction CSV from LncDC or a baseline tool (CPAT, PLEK, CPC2, CPPred, LncFinder, COME) with lncrna/mrna labels per transcript
- Gold-standard class labels for the test FASTA set (H-Test or M-Test mRNA and lncRNA transcripts)

## Outputs

- Per-condition metric table with accuracy, precision, specificity, and MCC (e.g., ablation_results.csv, table2_mouse_benchmark.csv)

## How to apply

Parse each tool's prediction output (LncDC writes CSV files labeling transcripts as 'lncrna' or 'mrna') and align predictions with the known gold-standard labels of the test FASTA (e.g., H-Test, M-Test). Compute accuracy, precision, specificity, and MCC per condition in scikit-learn — for the LncDC paper, MCC is the headline metric alongside accuracy when ranking tools in Table 2. For ablation studies, compute the same metric set for each feature condition: full model (SIF+PF+SSF, XGB_model_SIF_PF_SSF.pkl), SIF+PF-only (XGB_model_SIF_PF.pkl), and SSF-only, then assemble all conditions into a single results table (e.g., ablation_results.csv or table2_mouse_benchmark.csv). Validate against reported reference values: LncDC mouse on M-Test accuracy ≈0.9758 / MCC ≈0.9516; removing the 6 SSFs drops H-Test accuracy 0.9799→0.9783 and MCC 0.9598→0.9567; SSF-only reaches accuracy 0.8680, precision 0.8929, specificity 0.8998.

## Related tools

- **scikit-learn** (Computes accuracy, precision, specificity, and MCC from the binary prediction labels)
- **LncDC** (Produces the prediction CSVs (lncrna/mrna labels) whose metrics are being computed) — https://github.com/lim74/LncDC
- **Python** (Implementation language for the metric computation scripts)

## Examples

```
python bin/lncDC.py -i test/human_test_toy.fasta -o lncdc_human.csv -t 8  # then compute accuracy, precision, specificity, MCC per condition in scikit-learn
```

## Evaluation signals

- LncDC mouse model on M-Test must reach accuracy ≈0.9758 and MCC ≈0.9516 (human model ≈0.9723/0.9446)
- SIF+PF-only ablation on H-Test should reproduce accuracy 0.9783, precision 0.9746, MCC 0.9567
- SSF-only condition on H-Test should reach accuracy 0.8680, precision 0.8929, specificity 0.8998
- Every prediction row must map to exactly one gold-standard transcript label before counting errors
- Output table must contain all four metrics (accuracy, precision, specificity, MCC) for each tool/model condition

## Limitations

- The improvement from the 6 selected SSFs is marginal — removing them degraded H-Test MCC only from 0.9598 to 0.9567 — because SIFs and PFs already achieved high accuracy and further improvement became difficult
- Metric comparisons across tools are only fair when all tools run on the identical test set (e.g., the same M-Test FASTA)

## Evidence

- [other] Compute accuracy, precision, MCC, and specificity for each tool/model: [evidence span withheld — anchor status: not_found]
- [discussion] Benchmarks reported in discussion: "the LncDC model with only the 6 selected SSFs performed well in H-Test (precision:0.8929, specificity:0.8998)"
- [readme] Output labels in README: "The output file that will contain the prediction results in csv format. Long noncoding RNAs are labeled as 'lncrna', and message RNAs are labeled as 'mrna'."
- [abstract] Benchmarking claim in abstract: "Benchmarking experiments showed that LncDC consistently outperformed six state-of-the-art tools in distinguishing lncRNAs from mRNAs"
