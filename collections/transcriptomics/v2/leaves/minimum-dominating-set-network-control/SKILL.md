---
name: minimum-dominating-set-network-control
description: Use when you have a differential co-expression network (nodes, adjacency
  sets, per-edge differential P-values) and need a compact driver-node set rather
  than a degree-ranked top-k list.
when_to_use_negative:
- The goal is a naive 'degree-only' hub ranking — the article shows degree-only baselines
  perform worse than MDS-based selection; use this skill precisely to avoid that failure
  mode.
- You need an unweighted MDS configuration only (γ = 0, w_i = 1); then the weighting
  step is unnecessary, though the same integer-program formulation applies.
- The input is raw expression data with no differential co-expression network yet
  — run the upstream differential-network builder (Pearson correlation, Fisher Z-transform,
  ΔZ statistic) first.
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3512
tools:
- name: WMDS.net
  role: Framework implementing the weighted minimum dominating set model; main program
    takes two-condition expression matrices and outputs the gene list of key nodes
    in the final network.
  repo: https://github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Solves the binary integer-programming problem via the intlinprog function
    from the Optimization ToolBox.
- name: DawnRank
  role: Source of the gene interaction network whose intersection with TCGA genes
    forms the Gene_Net hub network used for this skill.
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:4102d03b8a7d595b421a65a7f93ba1b839b0c7d37c989cbb5bc5330ca62a99bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - DawnRank
  - MATLAB
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# minimum-dominating-set-network-control

## Summary

A network-control skill from WMDS.net that identifies driver nodes of a differential gene co-expression network by casting minimum dominating set (MDS) selection as a weighted binary integer program, where node weights integrate vertex degree and the significance of co-expression differences between two conditions.

## When to use

Use when you have a differential co-expression network (nodes, adjacency sets, per-edge differential P-values) and need a compact driver-node set rather than a degree-ranked top-k list.

## When NOT to use

- The goal is a naive 'degree-only' hub ranking — the article shows degree-only baselines perform worse than MDS-based selection; use this skill precisely to avoid that failure mode.
- You need an unweighted MDS configuration only (γ = 0, w_i = 1); then the weighting step is unnecessary, though the same integer-program formulation applies.
- The input is raw expression data with no differential co-expression network yet — run the upstream differential-network builder (Pearson correlation, Fisher Z-transform, ΔZ statistic) first.

## Inputs

- Differential co-expression network from the upstream network-builder task (node set V, adjacency sets ∂i, node degrees, per-edge differential co-expression P-values p_ij)
- Two-conditions expression matrices with rows = gene symbols and columns = sample IDs (e.g. TCGA_BLCAtumor and TCGA_BLCAnormal FPKM matrices)

## Outputs

- driver_gene_list.csv containing the selected driver-node gene symbols
- Binary vector x ∈ {0,1} indicating the weighted dominating set on the network

## How to apply

Load the differential co-expression network produced upstream, extracting the node set V, adjacency sets ∂i, node degrees, and per-edge differential co-expression P-values p_ij. Compute each node's weight as w_i = (Σ_{j∈∂i} −log p_ij)^(−γ) with γ = 0.01, a value chosen so the weighted dominating set retains the same node count as all unweighted minimum dominating set configurations. Formulate the problem as a binary integer program minimizing f(x) = Σ_i w_i·x_i with binary variables x_i ∈ {0,1} subject to the dominating-set constraint x_i + Σ_{j∈∂i} x_j ≥ 1 for every gene i, and solve it with the intlinprog function from the MATLAB Optimization Toolbox (R2020b), following the WMDS.net heuristic weighted-MDS approximation. Validate that the returned binary vector is a valid dominating set, that the WDS size matches unweighted MDS configurations on the same network, and write the selected driver gene symbols to driver_gene_list.csv.

## Related tools

- **WMDS.net** (Framework implementing the weighted minimum dominating set model; main program takes two-condition expression matrices and outputs the gene list of key nodes in the final network.) — https://github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Solves the binary integer-programming problem via the intlinprog function from the Optimization ToolBox.)
- **DawnRank** (Source of the gene interaction network whose intersection with TCGA genes forms the Gene_Net hub network used for this skill.)

## Examples

```
x = intlinprog(w, 0, [], [], A, b, zeros(n,1), ones(n,1));  % A*x >= b encodes x_i + sum_{j in adj(i)} x_j >= 1
```

## Evaluation signals

- Every gene in the network is either selected (x_i = 1) or adjacent to a selected gene — i.e. x is a valid dominating set satisfying x_i + Σ_{j∈∂i} x_j ≥ 1.
- The WDS size matches the size of unweighted minimum dominating set configurations on the same network (γ = 0, w_i = 1 comparison).
- With γ = 0.01, driver genes with high degree and significant co-expression difference are preferentially selected, consistent with the weight formula w_i = (Σ −log p_ij)^(−γ).
- In ablation scoring against the 616-gene CGC benchmark (precision = TP/(TP+FP), recall = TP/(TP+FN), Fβ-measure with β² = 0.2), F0.2 of WMDS.net > F0.2 of a randomly selected unweighted MDS > F0.2 of the degree-only baseline.

## Limitations

- Requires an upstream differential co-expression network; the skill consumes networks and P-values, not raw expression matrices directly (though the README main program takes two-condition expression matrices as input).
- Solution depends on the solver: intlinprog follows a heuristic weighted-MDS approximation approach, and the unweighted MDS baseline can have multiple configurations of the same size requiring random selection.
- Validation in the article is against a 616-gene Cancer Gene Census benchmark; performance outside CGC-annotated driver genes is not established.

## Evidence

- [methods] Formulate the dominating-set problem as a binary integer program: [evidence span withheld — anchor status: not_found]
- [methods] Node weight formula and gamma: [evidence span withheld — anchor status: not_found]
- [intro] Solver and MATLAB version: "solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab version R2020b"
- [readme] README on inputs and outputs: "The inputs to the main program are the expression matrices of the two conditions (which are TCGA_BLCAtumor and TCGA_BLCAnormal in the tested data), and the output is the gene list of key nodes in the"
- [results] Ablation ordering result: [evidence span withheld — anchor status: not_found]
