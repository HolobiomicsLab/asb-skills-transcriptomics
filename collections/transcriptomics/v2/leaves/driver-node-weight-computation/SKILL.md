---
name: driver-node-weight-computation
description: Use when you have a differential co-expression network with per-edge
  differential co-expression P-values p_ij and need node weights for a weighted minimum
  dominating set formulation of driver-node selection.
when_to_use_negative:
- The network is a plain gene interaction/hub network without per-edge differential
  co-expression P-values, so −log p_ij sums cannot be computed (e.g. the DawnRank-derived
  Gene_Net used only as the hub backbone).
- The goal is driver-node selection by methods other than weighted minimum dominating
  set, such as ranking-based or per-sample approaches like DawnRank.
- Only a single physiological state is available, so differential co-expression significance
  between two states cannot be assessed.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3366
tools:
- name: WMDS.net
  role: Framework whose WMDS model defines the node weight formula and supplies the
    pipeline in which weights feed the weighted dominating set integer program
  repo: https://github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Implementation language; the Optimization ToolBox function intlinprog solves
    the weighted binary integer program built from the computed node weights
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:4acd998c9dc0aa5fd66ec2027dcb0d160d30b09b2d898a3deb4c46125bba390d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# driver-node-weight-computation

## Summary

Computes per-node weights w_i = (Σ_{j∈∂i} −log p_ij)^(−γ), with γ = 0.01, from a differential gene co-expression network so that driver-node selection via weighted minimum dominating set (WMDS) integrates node degree and the significance of co-expression difference between two physiological states.

## When to use

Use when you have a differential co-expression network with per-edge differential co-expression P-values p_ij and need node weights for a weighted minimum dominating set formulation of driver-node selection.

## When NOT to use

- The network is a plain gene interaction/hub network without per-edge differential co-expression P-values, so −log p_ij sums cannot be computed (e.g. the DawnRank-derived Gene_Net used only as the hub backbone).
- The goal is driver-node selection by methods other than weighted minimum dominating set, such as ranking-based or per-sample approaches like DawnRank.
- Only a single physiological state is available, so differential co-expression significance between two states cannot be assessed.

## Inputs

- Differential gene co-expression network: node set V, adjacency sets ∂i, node degrees
- Per-edge differential co-expression P-values p_ij between two physiological states (e.g. tumor vs normal expression matrices with rows as gene symbols, columns as sample ids)

## Outputs

- Per-node weight vector w_i = (Σ_{j∈∂i} −log p_ij)^(−γ) with γ = 0.01
- Weighted binary integer program objective f(x) = Σ_i w_i·x_i ready for intlinprog

## How to apply

Load the differential co-expression network from the upstream network-builder step, extracting the node set V, adjacency sets ∂i, node degrees, and the per-edge differential co-expression P-values p_ij. For each node, sum −log p_ij over its neighbors and raise the negative of that sum to the power γ = 0.01, giving w_i = (Σ_{j∈∂i} −log p_ij)^(−γ); this makes high-degree genes with significant co-expression differences cheap to select in the subsequent weighted dominating set objective f(x) = Σ_i w_i·x_i. The exponent γ = 0.01 is a hyperparameter chosen so the weighted dominating set retains the same node count as unweighted minimum dominating set configurations on the same network. Implement the weight computation in Matlab (R2020b), in the same pipeline that later calls intlinprog from the Optimization ToolBox to solve the binary integer program with the dominating-set constraint x_i + Σ_{j∈∂i} x_j ≥ 1. As in the WMDS.net workflow applied to TCGA RNA-seq data (e.g. TCGA_BLCAtumor vs TCGA_BLCAnormal), the weights are computed before the integer program is formulated, and the resulting driver gene list is written to driver_gene_list.csv.

## Related tools

- **WMDS.net** (Framework whose WMDS model defines the node weight formula and supplies the pipeline in which weights feed the weighted dominating set integer program) — https://github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Implementation language; the Optimization ToolBox function intlinprog solves the weighted binary integer program built from the computed node weights)

## Examples

```
% Matlab R2020b: w = (-sum(-log(p_ij),2)).^(-0.01); % then solve via intlinprog: x = intlinprog(w, 0, [], [], A, b, ones(n,1), ones(n,1));
```

## Evaluation signals

- Verify each weight satisfies w_i = (Σ_{j∈∂i} −log p_ij)^(−γ) with γ = 0.01 by recomputing from the network's adjacency sets and edge P-values.
- Check that the resulting weighted dominating set has the same node count as unweighted minimum dominating set configurations on the same network — the stated purpose of the γ hyperparameter.
- Confirm weights are strictly positive and inversely related to the −log p_ij degree-sum (higher-degree, more significant genes get smaller weights, hence are favored in the minimization).
- Validate that feeding the weights into the binary integer program with constraint x_i + Σ_{j∈∂i} x_j ≥ 1 yields a valid dominating set covering every gene.

## Limitations

- γ = 0.01 is a hyperparameter; the documented guarantee is only that the weighted dominating set retains the same node count as minimum dominating set configurations, which constrains valid γ choices.
- Weights depend entirely on the quality of the upstream differential co-expression P-values p_ij; the article's missing discussion section means no published discussion of failure modes is available.

## Evidence

- [methods] weighted binary integer programming weight formula: [evidence span withheld — anchor status: not_found]
- [intro] weight rationale — degree and differential significance: "WMDS.net integrates the degree of nodes in the network and the significance of gene co-expression difference between two physiological states into the measurement of node controllability"
- [intro] solver implementation detail: "solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab version R2020b"
- [readme] input data format for the pipeline: "The inputs to the main program are the expression matrices of the two conditions (which are TCGA_BLCAtumor and TCGA_BLCAnormal in the tested data)"
- [methods] γ hyperparameter guarantee: [evidence span withheld — anchor status: not_found]
