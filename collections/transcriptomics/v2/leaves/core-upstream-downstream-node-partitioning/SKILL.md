---
name: core-upstream-downstream-node-partitioning
description: Use when a DC3-derived subpopulation-specific TF-target network (directed
  weighted adjacency matrix W_ij) is too large to interpret and you need the dense
  core regulatory circuitry plus its upstream/downstream effectors identified and
  significance-tested.
when_to_use_negative:
- You only have raw scRNA-seq/scATAC-seq/bulk HiChIP data and have not yet run DC3
  to obtain subpopulation-specific profiles and the TF-target score network — run
  the deconvolution/README pipeline first
- The network is already a small curated regulatory module where density testing adds
  no interpretive value
- Edges carry no directionality or scores (e.g., plain co-expression), since the QP
  objective Σ W_ij·x_i·y_j requires directed, weighted TF-target relations
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DC3
  role: Produces the subpopulation-specific gene expression, chromatin accessibility
    and enhancer-promoter contact profiles from which the TF-target regulatory network
    (and thus W_ij) is built
  repo: github.com/SUwonglab/DC3
- name: scipy.optimize
  role: Solves the quadratic program max Σ W_ij·x_i·y_j with L1-type (β = 1) simplex
    constraints via SLSQP or projected gradient to obtain the sparse x, y membership
    vectors defining the dense subnetwork
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:ae41677290537edf449d0b85f765d8b6f1061972137e0e9bf61885518b5a411d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  - scipy.optimize
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# core-upstream-downstream-node-partitioning

## Summary

Extracts the statistically dense subnetwork of key regulatory relationships from a large subpopulation-specific gene regulatory network (built from DC3-inferred profiles) by solving a quadratic program over TF-target edge weights, then partitions it into core (cross-regulating TFs), upstream, and downstream node sets.

## When to use

Use when a DC3-derived subpopulation-specific TF-target network (directed weighted adjacency matrix W_ij) is too large to interpret and you need the dense core regulatory circuitry plus its upstream/downstream effectors identified and significance-tested.

## When NOT to use

- You only have raw scRNA-seq/scATAC-seq/bulk HiChIP data and have not yet run DC3 to obtain subpopulation-specific profiles and the TF-target score network — run the deconvolution/README pipeline first
- The network is already a small curated regulatory module where density testing adds no interpretive value
- Edges carry no directionality or scores (e.g., plain co-expression), since the QP objective Σ W_ij·x_i·y_j requires directed, weighted TF-target relations

## Inputs

- Directed weighted TF-target adjacency matrix W_ij (edge-weight matrix) derived from the subpopulation-specific regulatory network built from DC3-inferred accessibility, expression and loop profiles
- Sparsity parameter β (paper's choice: β = 1)
- Null-model settings: 1000 random networks × 1000 edge switches per network

## Outputs

- Dense subnetwork node list partitioned into upstream U, core C, and downstream D node sets
- Empirical p-value per subpopulation (reported: 0.0230, 0.0180, 0.0320 for subpopulations 1–3)
- dense_subnetwork_results.tsv

## How to apply

Load the TF-target score adjacency matrix W_ij for each subpopulation-specific network and solve the quadratic program max Σ_ij W_ij·x_i·y_j subject to Σ_i x_i^β = 1, Σ_j y_j^β = 1, x_i, y_i ≥ 0 with β = 1 (L1-type constraint promoting sparse solutions), using scipy.optimize (e.g., SLSQP or projected gradient). Treat nodes with nonzero x_i or y_i as the dense subnetwork, then partition it into upstream U = {x_i > 0, y_i = 0}, core C = {x_i > 0, y_i > 0} (TFs that densely cross-regulate each other for robust state maintenance), and downstream D = {x_i = 0, y_i > 0}. Build the null by generating 1000 random networks via 1000 degree-preserving edge switches (swap sources and targets of two edges, keeping each node's in-/out-degree), extract dense subnetworks from each, and compute the empirical p-value of the real network's optimal objective against this null. Accept the subnetwork as statistically dense when the p-value is significant (paper reports p = 0.0230, 0.0180, and 0.0320 for subpopulations 1–3) and write results to dense_subnetwork_results.tsv.

## Related tools

- **DC3** (Produces the subpopulation-specific gene expression, chromatin accessibility and enhancer-promoter contact profiles from which the TF-target regulatory network (and thus W_ij) is built) — github.com/SUwonglab/DC3
- **scipy.optimize** (Solves the quadratic program max Σ W_ij·x_i·y_j with L1-type (β = 1) simplex constraints via SLSQP or projected gradient to obtain the sparse x, y membership vectors defining the dense subnetwork)

## Examples

```
from scipy.optimize import minimize; res = minimize(lambda v: -(W @ v[:n] @ v[n:]), x0, bounds=[(0, None)]*(2*n), constraints=[{'type': 'eq', 'fun': lambda v: v.sum() - 2}], method='SLSQP')
```

## Evaluation signals

- Optimal objective value of the real network is significant against the degree-preserving switch-generated null (paper p-values: 0.0230, 0.0180, 0.0320 for subpopulations 1–3)
- Partition invariants: core nodes satisfy x_i > 0 AND y_i > 0; upstream satisfies x_i > 0, y_i = 0; downstream satisfies x_i = 0, y_i > 0; no node falls outside U∪C∪D among nonzero-support nodes
- Constraint check: Σ_i x_i = 1 and Σ_j y_j = 1 with β = 1, all x_i, y_i ≥ 0 at the solver solution
- Each random network preserves per-node in-degree and out-degree (verify 1000 switches per network × 1000 networks)
- Results file dense_subnetwork_results.tsv contains one row per subpopulation with objective value and empirical p-value

## Limitations

- Empirical p-values are close to the 0.05 threshold (0.0230–0.0320), so density significance for individual subpopulations is marginal and sensitive to the 1000-network null size
- The β = 1 (L1) choice controls sparsity; other β values yield different subnetwork support and must be justified separately
- Quality of the extracted subnetwork is bounded by the fidelity of the DC3 deconvolution and the upstream TF-target scoring that produced W_ij

## Evidence

- [results] Core subnetwork definition via dense TF cross-regulation: [evidence span withheld — anchor status: not_found]
- [discussion] DC3 profiles as the substrate for core subnetwork extraction: "the accessibility, expression and loop profiles inferred by DC3 can serve as a foundation for further analyses of the regulatory systems, such as the extraction of core subnetworks"
- [intro] Purpose: derive core regulatory networks and downstream effectors per subpopulation: "we illustrate the value of results from DC3 by using them to derive the core regulatory network and their downstream effectors in each of the subpopulation"
