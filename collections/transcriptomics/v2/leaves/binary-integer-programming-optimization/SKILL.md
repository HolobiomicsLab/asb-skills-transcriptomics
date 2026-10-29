---
name: binary-integer-programming-optimization
description: Use when you have a differential gene co-expression network with per-edge
  P-values and must select a minimum dominating set of driver nodes, weighting nodes
  by degree and co-expression difference significance.
when_to_use_negative:
- Input is already a ranked gene list with no underlying network or per-edge P-values
  — node weights w_i = (Σ −log p_ij)^(−γ) cannot be computed
- Goal is an unweighted baseline rather than the biologically weighted model — use
  γ = 0 with w_i = 1 instead of the weighted formulation
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Matlab R2020b
  role: Provides the intlinprog binary integer-programming solver from the Optimization
    ToolBox used to solve the minimum dominating set problem
- name: WMDS.net
  role: Framework implementing the weighted MDS heuristic and pipeline that intlinprog
    is invoked within
  repo: https://github.com/chaofen123/WMDS.net
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:67d9486ceec656cc806eb089d9c87699f729dd75880c8c631556b9ece868f3a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - MATLAB
  - Matlab R2020b
  - WMDS.net
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# binary-integer-programming-optimization

## Summary

Formulates driver-node selection on a differential co-expression network as a weighted minimum dominating set binary integer program, solved with MATLAB intlinprog, so that node weights (degree plus differential co-expression significance) guide which genes are selected.

## When to use

Use when you have a differential gene co-expression network with per-edge P-values and must select a minimum dominating set of driver nodes, weighting nodes by degree and co-expression difference significance.

## When NOT to use

- Input is already a ranked gene list with no underlying network or per-edge P-values — node weights w_i = (Σ −log p_ij)^(−γ) cannot be computed
- Goal is an unweighted baseline rather than the biologically weighted model — use γ = 0 with w_i = 1 instead of the weighted formulation

## Inputs

- Differential co-expression network from the upstream network-builder task (node set V, adjacency sets ∂i, node degrees, per-edge P-values p_ij)
- Expression matrices of two conditions, e.g. TCGA_BLCAtumor and TCGA_BLCAnormal (rows = gene symbols, columns = sample IDs, FPKM)

## Outputs

- Binary vector x ∈ {0,1}^n indicating selected driver genes
- driver_gene_list.csv with selected driver-node gene symbols

## How to apply

Load the differential co-expression network's node set V, adjacency sets ∂i, node degrees, and per-edge differential co-expression P-values p_ij. Compute each node weight as w_i = (Σ_{j∈∂i} −log p_ij)^(−γ) with γ = 0.01, the hyperparameter that makes the weighted dominating set retain the same node count as all minimum dominating set configurations. Formulate the binary program: minimize f(x) = Σ_i w_i·x_i over x_i ∈ {0,1}, subject to x_i + Σ_{j∈∂i} x_j ≥ 1 for every gene i. Solve with the intlinprog function from the MATLAB Optimization Toolbox (R2020b), following the WMDS.net heuristic weighted-MDS approximation. Set γ = 0 with w_i = 1 to obtain an unweighted MDS baseline instead.

## Related tools

- **Matlab R2020b** (Provides the intlinprog binary integer-programming solver from the Optimization ToolBox used to solve the minimum dominating set problem)
- **WMDS.net** (Framework implementing the weighted MDS heuristic and pipeline that intlinprog is invoked within) — https://github.com/chaofen123/WMDS.net

## Examples

```
x = intlinprog(w, [], [], -A, -ones(n,1), [], [], zeros(n,1), ones(n,1));  % A encodes x_i + sum_{j in adj(i)} x_j >= 1, w_i=(sum(-log p_ij))^(-0.01)
```

## Evaluation signals

- Validate that the returned binary vector x is a valid dominating set: every gene is either selected or adjacent to a selected gene (x_i + Σ_{j∈∂i} x_j ≥ 1 holds for all i)
- Verify the weighted dominating set size matches the size of unweighted MDS configurations on the same network (guaranteed by γ = 0.01)
- For ablation, check the F0.2 ordering WMDS.net > randomly selected unweighted MDS > degree-only baseline against the 616-gene CGC benchmark
- Confirm the selected edge set respects the Δz differential co-expression statistic Δzij = (zij1 − zij2)/sqrt(1/(n1−3) + 1/(n2−3)) ~ N(0,1) used upstream to retain significant differential edges

## Limitations

- Requires per-edge differential co-expression P-values from the upstream differential network; node weights are undefined without them
- Relies on MATLAB intlinprog (Optimization ToolBox, R2020b), so reuse outside MATLAB requires an equivalent binary IP solver
- The weighted MDS is solved heuristically, so the returned solution is an approximation of the true weighted minimum dominating set

## Evidence

- [intro] intlinprog in MATLAB: "solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab version R2020b"
- [abstract] weighted MDS model: "we developed a weighted MDS network model (WMDS.net) to find the driver nodes of differential gene co-expression networks"
- [readme] input/output contract: "The inputs to the main program are the expression matrices of the two conditions (which are TCGA_BLCAtumor and TCGA_BLCAnormal in the tested data), and the output is the gene list of key nodes in the"
- [intro] node weight rationale: [evidence span withheld — anchor status: not_found]
