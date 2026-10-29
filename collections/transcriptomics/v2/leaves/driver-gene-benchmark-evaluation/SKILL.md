---
name: driver-gene-benchmark-evaluation
description: Use when one or more predicted driver gene lists (from WMDS.net, unweighted
  MDS, or degree-only ranking) must be scored against a curated cancer gene benchmark
  to compare precision/recall trade-offs.
when_to_use_negative:
- The prediction target is not cancer driver genes and no curated benchmark gene set
  (like the CGC 616-gene set) is available for validation.
- The compared gene lists come from networks of different sizes or different node
  sets, making equal-size baseline comparisons invalid.
- You only need raw differential co-expression network construction — this skill evaluates
  downstream driver predictions, not the ΔZ Fisher-Z network itself.
edam_operation: http://edamontology.org/operation_0322
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: WMDS.net
  role: Produces the primary driver gene prediction being benchmarked; its F0.2 score
    is compared against unweighted MDS and degree-only baselines.
  repo: https://github.com/chaofen123/WMDS.net
- name: MATLAB
  role: Environment used to generate the baseline predictions (intlinprog for MDS
    solutions, degree ranking) and compute precision/recall/F0.2 metrics.
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:d80739c4c3034b381e30ee3a01647039af71ca9321e1e99f00d2b6b19c3336c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - MATLAB
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_0322
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# driver-gene-benchmark-evaluation

## Summary

Benchmarks predicted cancer driver gene lists against the 616-gene CGC benchmark using precision, recall, and the Fβ-measure with β² = 0.2, in order to verify that biologically weighted network-control methods (e.g. WMDS.net) beat unweighted MDS and degree-only baselines.

## When to use

Use when one or more predicted driver gene lists (from WMDS.net, unweighted MDS, or degree-only ranking) must be scored against a curated cancer gene benchmark to compare precision/recall trade-offs.

## When NOT to use

- The prediction target is not cancer driver genes and no curated benchmark gene set (like the CGC 616-gene set) is available for validation.
- The compared gene lists come from networks of different sizes or different node sets, making equal-size baseline comparisons invalid.
- You only need raw differential co-expression network construction — this skill evaluates downstream driver predictions, not the ΔZ Fisher-Z network itself.

## Inputs

- driver gene list from WMDS.net (weighted MDS, γ = 0.01) as gene symbols
- unweighted MDS driver gene list (γ = 0, wi = 1) of the same differential co-expression network
- degree-only baseline gene list (same size as WMDS.net prediction, ranked by vertex degree)
- 616-gene CGC (Cancer Gene Census) benchmark gene set

## Outputs

- precision, recall, and F0.2-measure per method (e.g. figure3a_fmeasure_comparison.csv)
- ranking of methods confirming or refuting the expected ordering WMDS.net > unweighted MDS > degree-only

## How to apply

Collect the driver gene lists to compare — typically the WMDS.net prediction from the weighted MDS integer program, an unweighted MDS configuration (γ = 0, wi = 1) randomly selected from the same differential co-expression network, and a degree-only baseline of equal size obtained by ranking genes by vertex degree. Score each list against the 616-gene CGC benchmark using precision = TP/(TP+FP), recall = TP/(TP+FN), and the Fβ-measure with β² = 0.2, which emphasizes precision because validated driver genes cover only a small fraction of all predictions. Record all three F0.2 scores (e.g. in figure3a_fmeasure_comparison.csv) and verify the expected ordering: F0.2 of WMDS.net > F0.2 of the randomly selected unweighted MDS > F0.2 of the degree-only baseline, since the paper found that incorporating biologically meaningful weights significantly improved driver gene prediction and that WMDS.net outperformed the degree-only baseline largely due to the use of MDS.

## Related tools

- **WMDS.net** (Produces the primary driver gene prediction being benchmarked; its F0.2 score is compared against unweighted MDS and degree-only baselines.) — https://github.com/chaofen123/WMDS.net
- **MATLAB** (Environment used to generate the baseline predictions (intlinprog for MDS solutions, degree ranking) and compute precision/recall/F0.2 metrics.)

## Examples

```
% MATLAB: score driver lists against 616-gene CGC benchmark
 TP_wmds = numel(intersect(wmdsGenes, cgc616)); precision = TP_wmds/numel(wmdsGenes);
```

## Evaluation signals

- F0.2 ordering holds: F0.2(WMDS.net) > F0.2(randomly selected unweighted MDS) > F0.2(degree-only baseline)
- Baseline lists have exactly the same number of genes as the WMDS.net prediction (degree-only list is the same size as WMDS.net's)
- Every TP is a gene symbol present in both the prediction list and the 616-gene CGC benchmark
- Precision = TP/(TP+FP) and recall = TP/(TP+FN) computed with the CGC set as ground truth and predictions as the denominator for FP
- All three F0.2 scores are recorded (e.g. in figure3a_fmeasure_comparison.csv) rather than only the winning method

## Limitations

- The Fβ-measure uses β² = 0.2, which heavily weights precision over recall; results are sensitive to this choice because true driver genes are a small subset of predictions.
- The unweighted MDS baseline relies on a random single MDS configuration, so scores may vary between runs on the same differential network.
- The degree-only baseline is naive by design and its poor performance is partly attributable to the absence of the dominating-set constraint, not only to the lack of weights.

## Evidence

- [methods] expected F0.2 ordering: [evidence span withheld — anchor status: not_found]
- [methods] metric definitions: [evidence span withheld — anchor status: not_found]
- [abstract] abstract performance claim: "WMDS.net is powerful among various cancer datasets and outperformed the other top-tier tools with a better balance between precision and recall"
- [readme] readme validation claim: "WMDS.net is robust and powerful among various cancer datasets and outperformed the other top-tier tools with a better balance between precision and recall."
