---
name: switching-permutation-null-network-generation
description: Use when a dense subnetwork has been extracted from a TF-target adjacency
  matrix and its edge-density objective value must be tested against a null distribution
  of random networks that preserve each node's in- and out-degree.
when_to_use_negative:
- The network has not yet had a dense subnetwork extracted via the quadratic program
  — the null is only meaningful as a significance test for that objective value
- The graph is undirected or unweighted — the switching permutation and x/y factorized
  quadratic program assume a directed weighted TF-target score matrix
- Degree preservation is not required — e.g., when comparing against Erdős–Rényi-style
  nulls where in-/out-degree structure should be allowed to vary
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: DC3
  role: Produces the subpopulation-specific expression, accessibility, and HiChIP
    (peak–gene pair) profiles from which the TF-target score adjacency matrix is constructed
    before null network generation
  repo: https://github.com/SUwonglab/DC3
- name: scipy.optimize
  role: Solves the quadratic program (SLSQP or projected gradient) used to extract
    the dense subnetwork from both the real and each random network
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:0692ba492366b856afbed813807b9b6725d3cb3788de7cc58d5f996e6e92cba7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/DC3
  tools:
  - DC3
  - scipy.optimize
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# switching-permutation-null-network-generation

## Summary

Generates degree-preserving random null networks by repeatedly swapping the source and target nodes of selected edges (switching permutation), enabling empirical significance testing of a dense subnetwork extracted via quadratic programming from subpopulation-specific regulatory networks.

## When to use

Use when a dense subnetwork has been extracted from a TF-target adjacency matrix and its edge-density objective value must be tested against a null distribution of random networks that preserve each node's in- and out-degree.

## When NOT to use

- The network has not yet had a dense subnetwork extracted via the quadratic program — the null is only meaningful as a significance test for that objective value
- The graph is undirected or unweighted — the switching permutation and x/y factorized quadratic program assume a directed weighted TF-target score matrix
- Degree preservation is not required — e.g., when comparing against Erdős–Rényi-style nulls where in-/out-degree structure should be allowed to vary

## Inputs

- Directed weighted TF-target score adjacency matrix W_ij (from DC3 subpopulation-specific regulatory network construction)
- Number of random networks (paper: 1000) and switches per network (paper: 1000)
- Quadratic program solution vectors x_i, y_j with β = 1

## Outputs

- 1000 degree-preserving random networks (edge lists or adjacency matrices)
- Null distribution of optimal quadratic-program objective values
- Empirical p-value for the real network's density (paper: p = 0.0230, 0.0180, 0.0320 for subpopulations 1–3)
- dense_subnetwork_results.tsv with upstream/core/downstream partition (U, C, D node sets)

## How to apply

First extract the dense subnetwork from a TF-target score adjacency matrix W_ij by solving the quadratic program max Σ_ij W_ij·x_i·y_j subject to Σ_i x_i^β = 1, Σ_j y_j^β = 1, x_i ≥ 0, y_i ≥ 0, using the paper's choice β = 1 (an L1-type constraint promoting sparse solutions) via scipy.optimize (e.g., SLSQP or projected gradient). Then generate 1000 random networks by switching permutation: repeatedly select two edges and swap their source and target nodes, performing 1000 switches per random network so that each node retains its in-degree and out-degree. Extract the dense subnetwork from each random network with the same quadratic program and record its optimal objective value, building an empirical null distribution. Compare the real network's objective value against this null to compute an empirical p-value; the paper reports p = 0.0230, 0.0180, and 0.0320 for subpopulations 1–3. Partition the accepted subnetwork into upstream U = {i | x_i > 0, y_i = 0}, core C = {i | x_i > 0, y_i > 0}, and downstream D = {i | x_i = 0, y_i > 0} sets, where the core consists of TFs that densely cross-regulate each other to robustly maintain the cellular state. Write results to dense_subnetwork_results.tsv.

## Related tools

- **DC3** (Produces the subpopulation-specific expression, accessibility, and HiChIP (peak–gene pair) profiles from which the TF-target score adjacency matrix is constructed before null network generation) — https://github.com/SUwonglab/DC3
- **scipy.optimize** (Solves the quadratic program (SLSQP or projected gradient) used to extract the dense subnetwork from both the real and each random network)

## Examples

```
python dc3.py -k 3 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- Each random network must preserve every node's original in-degree and out-degree (verify degree sequences match the real network exactly)
- Each random network results from exactly 1000 switches, and 1000 random networks are generated in total
- The extracted subnetwork is accepted as statistically dense only when the empirical p-value is significant; paper values are p = 0.0230, 0.0180, and 0.0320 for subpopulations 1–3
- The quadratic program constraints hold: Σ_i x_i^β = 1 and Σ_j y_j^β = 1 with β = 1, and x_i, y_j ≥ 0
- dense_subnetwork_results.tsv is produced with the U/C/D partition (x_i > 0 & y_i = 0 → upstream; both > 0 → core; x_i = 0 & y_i > 0 → downstream)

## Limitations

- Significance depends on the null: the switching permutation preserves degree structure, so density findings are conditional on that null model; the paper's reported p-values (0.0230–0.0320) are near the conventional 0.05 threshold
- The β = 1 choice is an L1-type constraint that promotes sparse solutions; other β values would change which nodes receive nonzero mass and thus the extracted subnetwork
- DC3 does not yet support jointly-profiled single-cell data types (e.g., sci-CAR), so the regulatory network feeding this skill must come from separately profiled scRNA-seq, scATAC-seq, and bulk HiChIP samples
- The README notes HiChIP preprocessing should restrict interactions to genes in the scRNA-seq data and enhancers in the scATAC-seq data; mis-specified restriction changes the network W_ij being tested

## Evidence

- [results] Solve the quadratic program with the paper's L1 constraint and switch-based null: [evidence span withheld — anchor status: not_found]
- [results] Switching permutation preserves node degrees: [evidence span withheld — anchor status: not_found]
- [results] Empirical p-values for the three subpopulations: [evidence span withheld — anchor status: not_found]
- [results] Core subnetwork interpretation from the article: [evidence span withheld — anchor status: not_found]
- [discussion] DC3 output profiles serve as the foundation for network extraction: "the accessibility, expression and loop profiles inferred by DC3 can serve as a foundation for further analyses of the regulatory systems, such as the extraction of core subnetworks"
