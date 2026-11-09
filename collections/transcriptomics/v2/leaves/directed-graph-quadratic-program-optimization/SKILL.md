---
name: directed-graph-quadratic-program-optimization
description: Use when you have a directed weighted TF-target score matrix derived
  from subpopulation-specific profiles (e.g., DC3-derived HiChIP peak-gene pairs)
  and need the densely cross-regulating core subnetwork plus upstream/downstream effectors.
when_to_use_negative:
- The regulatory network has not yet been constructed from subpopulation-specific
  profiles — DC3 must first deconvolve bulk HiChIP into cluster-specific peak-gene
  pairs (cluster-specific-peaks-genes-pairs.txt)
- You need undirected density/clique-finding — this skill is specific to directed
  graphs with separate source (x) and target (y) membership variables
- You lack a meaningful edge-weight signal (e.g., unweighted, fully connected toy
  graph), since density significance is judged against a degree-preserving null and
  meaningless weights yield meaningless p-values
tools:
- name: DC3
  role: Produces the subpopulation-specific expression, accessibility and HiChIP loop
    profiles from which the TF-target score adjacency matrix for dense subnetwork
    extraction is built
  repo: github.com/SUwonglab/DC3
- name: scipy.optimize
  role: Solves the quadratic program (e.g., SLSQP or projected gradient) under the
    β=1 L1-type constraints to obtain the x and y membership vectors
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:a33181eb834f662d903e96c480e8141f85a009014655ab24226c8a1fc6db73e2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  - scipy.optimize
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# directed-graph-quadratic-program-optimization

## Summary

Extracts a statistically significant dense subnetwork from a directed weighted graph (e.g., a TF-target score adjacency matrix built from DC3 subpopulation-specific profiles) by solving max Σ_ij W_ij·x_i·y_j under L1-type (β=1) constraints, then partitioning nodes into upstream, core, and downstream sets.

## When to use

Use when you have a directed weighted TF-target score matrix derived from subpopulation-specific profiles (e.g., DC3-derived HiChIP peak-gene pairs) and need the densely cross-regulating core subnetwork plus upstream/downstream effectors.

## When NOT to use

- The regulatory network has not yet been constructed from subpopulation-specific profiles — DC3 must first deconvolve bulk HiChIP into cluster-specific peak-gene pairs (cluster-specific-peaks-genes-pairs.txt)
- You need undirected density/clique-finding — this skill is specific to directed graphs with separate source (x) and target (y) membership variables
- You lack a meaningful edge-weight signal (e.g., unweighted, fully connected toy graph), since density significance is judged against a degree-preserving null and meaningless weights yield meaningless p-values

## Inputs

- Directed weighted TF-target score adjacency matrix (edge-weight matrix W_ij) built from subpopulation-specific regulatory networks, e.g., from DC3 cluster-specific peak-gene pairs
- Exponent β for the L1-type norm constraints (paper uses β = 1)
- Null-model parameters: 1000 random networks, 1000 edge switches per network

## Outputs

- Dense subnetwork node set (nodes with nonzero x_i or nonzero y_i)
- Upstream (U), core (C), and downstream (D) node-set partition
- Empirical p-value from the degree-preserving random-network null distribution
- dense_subnetwork_results.tsv

## How to apply

Represent the network as an edge-weight matrix W_ij and solve the quadratic program max Σ_ij W_ij·x_i·y_j subject to Σ_i x_i^β = 1, Σ_j y_j^β = 1, x,y ≥ 0, using the paper's choice β = 1 (L1-type constraint promoting sparse solutions) via scipy.optimize (SLSQP or projected gradient). Select the dense subnetwork as nodes with nonzero x_i or nonzero y_i, then partition into upstream U = {x_i>0, y_i=0}, core C = {x_i>0, y_i>0}, and downstream D = {x_i=0, y_i>0}. Assess significance by generating 1000 degree-preserving random networks via edge-switching (1000 switches per network, preserving each node's in-degree and out-degree), extracting the dense subnetwork from each, and computing an empirical p-value by comparing the real network's optimal objective against the null distribution. Accept the subnetwork as statistically dense when the p-value is significant (paper reports p = 0.0230, 0.0180, 0.0320 for subpopulations 1–3) and write results to dense_subnetwork_results.tsv.

## Related tools

- **DC3** (Produces the subpopulation-specific expression, accessibility and HiChIP loop profiles from which the TF-target score adjacency matrix for dense subnetwork extraction is built) — github.com/SUwonglab/DC3
- **scipy.optimize** (Solves the quadratic program (e.g., SLSQP or projected gradient) under the β=1 L1-type constraints to obtain the x and y membership vectors)

## Examples

```
from scipy.optimize import minimize; res = minimize(lambda z: -sum(W[i,j]*x[i]*y[j] for i,j in edges), z0, method='SLSQP', constraints=[{'type':'eq','fun': lambda z: z[:n].sum()-1}, {'type':'eq','fun': lambda z: z[n:].sum()-1}], bounds=[(0,None)]*(2*n))
```

## Evaluation signals

- x and y membership vectors satisfy the equality constraints Σ_i x_i^β = 1 and Σ_j y_j^β = 1 with all components non-negative
- Every node in the partition is exactly one of upstream (x_i>0, y_i=0), core (x_i>0, y_i>0), or downstream (x_i=0, y_i>0); nodes with both zero are excluded
- Empirical p-value computed over 1000 edge-switched degree-preserving random networks is significant (paper reports p = 0.0230, 0.0180, 0.0320 for subpopulations 1–3)
- Random networks preserve per-node in-degree and out-degree (1000 switches per network), verifiable by degree-sequence comparison to the original graph
- Results file dense_subnetwork_results.tsv is produced with the partition and p-value per subpopulation

## Limitations

- The quadratic program with L1-type (β=1) constraints promotes sparse solutions but is non-convex; the optimizer choice (SLSQP vs projected gradient) may affect which local optimum is reached
- Significance depends on the degree-preserving edge-switching null (1000 random networks × 1000 switches); sparse or very dense graphs may give unreliable p-values
- Subpopulation-specific profiles must first be inferred by DC3 (and validated, e.g., by FACS-sorted marker-positive cells for HiChIP) before this skill is meaningful

## Evidence

- [discussion] the accessibility, expression and loop profiles inferred by DC3 can serve as a foundation for further analyses of the regulatory systems, such as the extraction of core subnetworks: "the accessibility, expression and loop profiles inferred by DC3 can serve as a foundation for further analyses of the regulatory systems, such as the extraction of core subnetworks"
- [intro] we illustrate the value of results from DC3 by using them to derive the core regulatory network and their downstream effectors in each of the subpopulation: "we illustrate the value of results from DC3 by using them to derive the core regulatory network and their downstream effectors in each of the subpopulation"
- [readme] cluster-specific-peaks-genes-pairs.txt  the cluster-specific peak-gene pairs. First column is the gene name, second column is the peak name: "cluster-specific-peaks-genes-pairs.txt  the cluster-specific peak-gene pairs. First column is the gene name, second column is the peak name, the third column is the p-value for gene"
- [abstract] DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data: "DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data"
