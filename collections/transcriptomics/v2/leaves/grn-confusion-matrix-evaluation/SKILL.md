---
name: grn-confusion-matrix-evaluation
description: Use when one or more predicted GRNs (e.g., Augusta MI-only, Augusta with
  TFBM/DB refinement, GENIE3, TDAracne, KBoost) must be quantitatively compared against
  a gold-standard network with directed-edge ground truth.
when_to_use_negative:
- No gold-standard network is available for the organism/dataset, so no confusion
  matrix against known edges can be computed
- Evaluating Augusta Boolean network drafts, which the paper cautions 'should be treated
  with caution as they rather present first drafts than final BNs' rather than final
  networks to be scored
- Gold standard includes edge types/interaction semantics that the predictor explicitly
  does not model and cannot be simplified away
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: Augusta
  role: Produces the inferred GRNs (MI-based, with/without TFBM and curated-database
    refinement) whose edges are scored in the confusion matrix
  repo: https://github.com/JanaMus/Augusta
- name: GENIE3
  role: Baseline GRN inference tool run with default parameters; its output network
    is evaluated against the same gold standard
- name: TDAracne
  role: Baseline MI-based tool (number-of-bins set to 10 with percentile/rank normalization);
    output evaluated against the gold standard
- name: KBoost
  role: Baseline GRN inference tool run with default parameters; its output network
    is evaluated against the same gold standard
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:2f81928e5a13eb3c1d7f59707a9f4958034e276ca9499a922ff314fa88db76c9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WanluLiuLab/CRISPRTE
  repo_url: https://github.com/JanaMus/Augusta
  tools:
  - Augusta
  - GENIE3
  - KBoost
  - MEME Suite
  - OmniPath
  - Python
  - SignaLink
  - Signor
  - TDAracne
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# grn-confusion-matrix-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking protocol for scoring inferred gene regulatory networks (Augusta, GENIE3, TDAracne, KBoost) against gold-standard networks using confusion matrices that yield sensitivity, specificity, accuracy, and execution time, as done on the DREAM 4 In Silico Network Challenge and B. subtilis 168 datasets.

## When to use

Use when one or more predicted GRNs (e.g., Augusta MI-only, Augusta with TFBM/DB refinement, GENIE3, TDAracne, KBoost) must be quantitatively compared against a gold-standard network with directed-edge ground truth.

## When NOT to use

- No gold-standard network is available for the organism/dataset, so no confusion matrix against known edges can be computed
- Evaluating Augusta Boolean network drafts, which the paper cautions 'should be treated with caution as they rather present first drafts than final BNs' rather than final networks to be scored
- Gold standard includes edge types/interaction semantics that the predictor explicitly does not model and cannot be simplified away

## Inputs

- Predicted GRN edge lists from Augusta (full and no-refinement), GENIE3, TDAracne, KBoost
- Gold-standard network with directed edges (DREAM 4 gold standard) or positive/negative interactions (B. subtilis gold standard)
- Execution time measurements per inference tool

## Outputs

- Confusion matrix per tool (binary directed, or 3x3 multiclass over positive/negative/absent edges)
- Sensitivity, specificity, and accuracy per tool (per-edge-class for multiclass)
- Metrics table, e.g., dream4_benchmark_metrics.csv or bsubtilis_benchmark_results.csv

## How to apply

Predicted networks must first be format-matched to the gold standard: for the DREAM 4 evaluation, simplify Augusta networks by ignoring interaction types and converting negative edges to positive so only directed edges remain. For binary directed gold standards (DREAM 4, 100 genes, 21 samples), compute a confusion matrix of predicted vs. true edges and derive sensitivity, specificity, and overall accuracy, plus per-tool execution time. For multiclass gold standards such as B. subtilis 168 (which contains positive and negative interactions), use a 3x3 confusion matrix over positive, negative, and absent edge classes, compute sensitivity and specificity per class, average them, and report overall accuracy. Always record runtime per tool (and per Augusta pipeline step when applicable, e.g., Augusta full at 714.18 s vs. MI-only at 83.07 s on DREAM 4) and write metrics to a CSV such as dream4_benchmark_metrics.csv or bsubtilis_benchmark_results.csv for direct comparison against reported values.

## Related tools

- **Augusta** (Produces the inferred GRNs (MI-based, with/without TFBM and curated-database refinement) whose edges are scored in the confusion matrix) — https://github.com/JanaMus/Augusta
- **GENIE3** (Baseline GRN inference tool run with default parameters; its output network is evaluated against the same gold standard)
- **TDAracne** (Baseline MI-based tool (number-of-bins set to 10 with percentile/rank normalization); output evaluated against the gold standard)
- **KBoost** (Baseline GRN inference tool run with default parameters; its output network is evaluated against the same gold standard)

## Examples

```
python -c "import Augusta; Augusta.RNASeq_to_GRN(count_table_input='MyCT_file.csv', promoter_length=My_number, genbank_file_input='MyGB_file.gb', normalization_type='My_string', motifs_max_time=My_seconds)"
```

## Evaluation signals

- Reproduced metrics for Augusta full on DREAM 4 match reported values: sensitivity 0.06, specificity 0.91, accuracy 0.89 in 714.18 s
- Reproduced metrics for Augusta no refinement on DREAM 4 match: sensitivity 0.38, specificity 0.54, accuracy 0.54 in 83.07 s
- Edge-format check: predicted networks simplified to directed edges only (interaction types ignored, negatives converted to positive) before comparison with the DREAM 4 gold standard
- Multiclass check: B. subtilis evaluation uses a 3x3 confusion matrix over positive, negative, and absent edge classes with per-class sensitivity/specificity averaged
- Row count of the metrics CSV equals the number of evaluated tools/configurations (Augusta full, Augusta no refinement, GENIE3, TDAracne, KBoost)

## Limitations

- Augusta GRN refinement with de novo TFBM discovery (MEME Suite on 1000 bp upstream promoters) may be computationally very demanding
- Inferring autoregulations remains a significant challenge in network inference, affecting confusion-matrix counts on self-loops
- Inferred networks should be considered approximations, not perfectly accurate representations of biological system regulation, so benchmark metrics are not a complete quality measure
- Refinement steps require additional knowledge of a genome (annotated GenBank file) and cannot be performed for organisms absent from databases without de novo motifs

## Evidence

- [results] DREAM 4 Augusta full metrics: [evidence span withheld — anchor status: not_found]
- [results] DREAM 4 Augusta no-refinement metrics: [evidence span withheld — anchor status: not_found]
- [readme] Augusta README usage for GRN inference producing networks to evaluate: [evidence span withheld — anchor status: external_doc]
