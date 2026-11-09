---
name: gene-degree-ranking-baseline-comparison
description: Use when you must reproduce or benchmark an ablation of a network driver-gene
  method against degree-only and unweighted baselines, scoring each list against a
  curated benchmark like the 616-gene CGC set.
when_to_use_negative:
- The network is not a differential co-expression network — degree is only meaningful
  as a baseline when computed on the same ΔZ-filtered network used by WMDS.net
- The evaluation benchmark differs from the CGC set or the β parameter is changed,
  since the F0.2 measure deliberately weights precision over recall and is integral
  to the published comparison
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: WMDS.net
  role: Produces the reference driver-gene list that the degree-only and unweighted
    MDS baselines are compared against, and defines the k genes taken for the degree
    ranking
  repo: https://github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Solves the weighted and unweighted (γ = 0, wi = 1) MDS binary integer programs
    via intlinprog from the Optimization ToolBox
- name: DawnRank
  role: Source of the gene interaction network whose hub (Gene_Net) defines the edges
    on which vertex degree is computed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:2e4731ffdc1905fdf8a5815d3aee27b74508e5225f3f651daa6fa4581616776c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - DawnRank
  - MATLAB
  - Matlab R2020b
  - WMDS.net
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gene-degree-ranking-baseline-comparison

## Summary

Builds a naive 'degree-only' baseline by ranking genes in the differential co-expression network by vertex degree and comparing it against weighted MDS (WMDS.net) and unweighted MDS driver-gene predictions, to quantify whether biological weighting and MDS-based network control add predictive value over simple degree heuristics.

## When to use

Use when you must reproduce or benchmark an ablation of a network driver-gene method against degree-only and unweighted baselines, scoring each list against a curated benchmark like the 616-gene CGC set.

## When NOT to use

- The network is not a differential co-expression network — degree is only meaningful as a baseline when computed on the same ΔZ-filtered network used by WMDS.net
- The evaluation benchmark differs from the CGC set or the β parameter is changed, since the F0.2 measure deliberately weights precision over recall and is integral to the published comparison

## Inputs

- Differential co-expression network (Gene_Net hub network intersected with TCGA genes, retaining significant ΔZ differential edges)
- WMDS.net predicted driver gene list
- 616-gene CGC (Cancer Gene Census) benchmark gene set

## Outputs

- Degree-only baseline driver gene list (top-k genes by vertex degree, k matched to WMDS.net list size)
- Randomly selected unweighted MDS (γ = 0, wi = 1) driver gene list
- Precision, recall, and F0.2-measure scores for all three lists, recorded in figure3a_fmeasure_comparison.csv

## How to apply

First compute the differential co-expression network from TCGA_BLCAtumor and TCGA_BLCAnormal FPKM expression matrices using the Fisher Z-transformed Pearson correlation ΔZ statistic over Gene_Net edges, retaining significant differential edges. Build the degree-only baseline by ranking genes in this differential network by vertex degree and taking the same number of top genes as WMDS.net predicts. Generate the unweighted MDS baseline by setting node weights to 1 (γ = 0) in the integer program f(x) = Σ wi·xi solved with MATLAB intlinprog, and randomly selecting one MDS configuration of the same differential network as the critical-node set. Score all three driver lists against the 616-gene CGC benchmark using precision = TP/(TP+FP), recall = TP/(TP+FN), and the Fβ-measure with β² = 0.2, which weights precision heavily. Verify the ordering F0.2 of WMDS.net > unweighted MDS > degree-only baseline holds and record all three scores in figure3a_fmeasure_comparison.csv.

## Related tools

- **WMDS.net** (Produces the reference driver-gene list that the degree-only and unweighted MDS baselines are compared against, and defines the k genes taken for the degree ranking) — https://github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Solves the weighted and unweighted (γ = 0, wi = 1) MDS binary integer programs via intlinprog from the Optimization ToolBox)
- **DawnRank** (Source of the gene interaction network whose hub (Gene_Net) defines the edges on which vertex degree is computed)

## Examples

```
x = intlinprog(w, 0, ones(n,1), sparse(-eye(n)-A), -ones(n,1), zeros(n,1), [], [], optimoptions('intlinprog')); idx = find(x); [p,r,f02] = score_against_cgc(net_names(idx), cgc_616, 0.2);
```

## Evaluation signals

- The reported ordering holds: F0.2 of WMDS.net > F0.2 of randomly selected unweighted MDS > F0.2 of the degree-only baseline (Figure 3A)
- The degree-only baseline list has exactly the same number of genes as the WMDS.net prediction, ensuring a like-for-like precision comparison
- All three precision/recall/F0.2 scores are computed against the same 616-gene CGC benchmark and recorded in figure3a_fmeasure_comparison.csv
- The unweighted MDS baseline uses the identical differential network as WMDS.net, differing only in node weights (wi = 1, γ = 0)
- Precision and recall are computed as TP/(TP+FP) and TP/(TP+FN) respectively, with Fβ using β² = 0.2

## Limitations

- The unweighted MDS baseline relies on a single randomly selected MDS configuration rather than averaging over multiple solutions, so its score may vary between runs
- The degree-only baseline 'performed much better' than expected partly because MDS itself — not the weights — contributes substantially; the ablation is needed to separate these effects
- Comparison is only as fair as the matched list sizes; taking a different number of top-degree genes changes precision/recall trade-offs

## Evidence

- [methods] Build a 'degree-only' baseline by ranking genes in the differential co-expression network by vertex degree and taking the same number of top genes as WMDS.net predicts.: [evidence span withheld — anchor status: not_found]
- [results] WMDS.net performed much better than the naive 'degree only' baseline, largely due to the use of MDS.: [evidence span withheld — anchor status: not_found]
- [readme] In this study, we implement the algorithm using the Matlab program language and solve the binary integer-programming problem using function “intlinprog”: "we implement the algorithm using the Matlab program language and solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab"
- [methods] Score all three driver lists against the 616-gene CGC benchmark using precision = TP/(TP+FP), recall = TP/(TP+FN), and the Fβ-measure with β² = 0.2.: [evidence span withheld — anchor status: not_found]
