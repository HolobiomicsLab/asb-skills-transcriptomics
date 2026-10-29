---
name: fmeasure-precision-recall-scoring
description: Use when a predicted gene list (e.g., driver genes from WMDS.net) must
  be quantitatively compared against a benchmark gene set such as the 616-gene CGC
  list, and a precision-weighted Fβ score is the desired metric.
when_to_use_negative:
- When the comparison is against a gold standard that is not a fixed benchmark gene
  set (e.g., replication across independent cohorts) — precision/recall against CGC
  is not meaningful there
- When recall should dominate precision (β² > 1, e.g., β² = 2); this skill fixes β²
  = 0.2 to privilege precision in driver gene prediction
- When predictions are ranked scores rather than discrete gene lists — TP/FP counts
  require a defined predicted set
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: WMDS.net
  role: Produces the primary predicted driver gene list that is scored against the
    CGC benchmark
  repo: github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Environment (intlinprog, Optimization Toolbox) in which the MDS solutions
    and the three-way scoring comparison are computed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:ea273d3fb0dd22e06e91c967a1c0e232d72b6334fffce6d6332e28e9abb5362d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# fmeasure-precision-recall-scoring

## Summary

Scores predicted driver gene lists against a curated benchmark (the 616-gene CGC set) using precision = TP/(TP+FP), recall = TP/(TP+FN), and the Fβ-measure with β² = 0.2, weighting recall heavily to emphasize precision. Used to rank WMDS.net against unweighted MDS and degree-only baselines on the TCGA bladder cancer differential co-expression network.

## When to use

Use when a predicted gene list (e.g., driver genes from WMDS.net) must be quantitatively compared against a benchmark gene set such as the 616-gene CGC list, and a precision-weighted Fβ score is the desired metric.

## When NOT to use

- When the comparison is against a gold standard that is not a fixed benchmark gene set (e.g., replication across independent cohorts) — precision/recall against CGC is not meaningful there
- When recall should dominate precision (β² > 1, e.g., β² = 2); this skill fixes β² = 0.2 to privilege precision in driver gene prediction
- When predictions are ranked scores rather than discrete gene lists — TP/FP counts require a defined predicted set

## Inputs

- Predicted driver gene list from WMDS.net (key nodes of the differential co-expression network)
- Predicted driver gene list from the unweighted MDS baseline (γ = 0, wi = 1)
- Predicted driver gene list from the degree-only baseline (top-degree genes, same count as WMDS.net)
- 616-gene CGC (Cancer Gene Census) benchmark gene set

## Outputs

- Precision, recall, and F0.2 score for each of the three driver lists
- figure3a_fmeasure_comparison.csv with all three F0.2 scores

## How to apply

For each predicted gene list, classify each gene as a true positive if it appears in the 616-gene CGC benchmark, then compute precision = TP/(TP+FP) and recall = TP/(TP+FN). Combine them with the Fβ-measure using β² = 0.2 so precision dominates (F = (1+β²)·P·R/(β²·P+R)), reflecting that fewer false-positive driver calls are preferred. Apply the same scoring procedure to all competing lists — WMDS.net drivers, a randomly selected unweighted MDS (γ = 0, wi = 1) of the same differential network, and the top-degree genes — taking exactly the same number of predictions for each so scores are comparable. Verify the expected ordering F0.2(WMDS.net) > F0.2(unweighted MDS) > F0.2(degree-only) and record all three scores in figure3a_fmeasure_comparison.csv.

## Related tools

- **WMDS.net** (Produces the primary predicted driver gene list that is scored against the CGC benchmark) — github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Environment (intlinprog, Optimization Toolbox) in which the MDS solutions and the three-way scoring comparison are computed)

## Examples

```
Fb = (1+0.2^2)*(P*R)/((0.2^2)*P+R); % F0.2 score of driver list vs 616-gene CGC benchmark, beta^2=0.2
```

## Evaluation signals

- Ordering invariant: F0.2 of WMDS.net > F0.2 of a randomly selected unweighted MDS > F0.2 of the degree-only baseline (Figure 3A expectation)
- Precision = TP/(TP+FP) and recall = TP/(TP+FN) must sum TP, FP over the exact predicted lists, with the degree-only baseline truncated to the same number of genes as WMDS.net predicts
- F0.2 computed with β² = 0.2 (not the default β² = 1 F1 score); recomputing with the F1 formula should not reproduce the reported values
- All three scores present in figure3a_fmeasure_comparison.csv, one row per baseline method
- True positives are members of the 616-gene CGC benchmark set only

## Limitations

- Scores depend on the 616-gene CGC benchmark; a prediction missed by CGC is counted as a false positive even if biologically plausible
- The unweighted MDS baseline score is stochastic (one randomly selected MDS configuration), so the F0.2 value can vary across runs
- The heavy precision weighting (β² = 0.2) understates recall improvements, so lists with broad coverage but more false positives will be penalized

## Evidence

- [methods] Score all three driver lists against the 616-gene CGC benchmark using precision = TP/(TP+FP), recall = TP/(TP+FN), and the Fβ-measure with β² = 0.2.: [evidence span withheld — anchor status: not_found]
- [methods] verify the reported ordering in Figure 3A holds, i.e. F0.2 of WMDS.net > F0.2 of randomly selected unweighted MDS > F0.2 of the degree-only baseline: [evidence span withheld — anchor status: not_found]
- [readme] WMDS.net is robust and powerful among various cancer datasets and outperformed the other top-tier tools with a better balance between precision and recall.: "WMDS.net is robust and powerful among various cancer datasets and outperformed the other top-tier tools with a better balance between precision and recall."
