---
name: directed-tree-conversion-causal-additive-trees
description: Use when active subnetworks are returned as undirected graphs by multiple
  detection methods (e.g., from the Ma et al. 2011 simulation benchmark with 500 genes,
  20 samples per condition) and you need directionality to compare them or fit tree-based
  SEMs.
when_to_use_negative:
- The subnetworks are already directed (e.g., KEGG-derived directed graphs used by
  SEMgraph directly) — conversion would add an unnecessary, possibly lossy orientation
  step.
- The goal is structural equation modeling on a large reference graph rather than
  benchmarking subnetwork detection — use SEMrun/SEMdag on the full graph instead
  of collapsing modules to trees.
- Undirected output is required by downstream analysis (e.g., Steiner tree backbone
  extraction or TAHC clustering in SEMgraph), where orientation is irrelevant or harmful.
tools:
- name: SEMgraph
  role: R package providing SEMtree() ST/STr2z subnetwork detection and graph-to-SEM
    conversion; supports tree (arborescence) structure learning from correlation matrices
    and igraph objects for the directed-tree workflow
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: igraph
  role: Graph representation format for the undirected subnetworks in and directed
    trees out of the conversion step
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:d01fe64414ab2f55c684561ef06b41768dbb9d502f2a9f3a605a23e58d4e2063
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - R
  - SEMgraph
  - SEMtree()
  - igraph
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# directed-tree-conversion-causal-additive-trees

## Summary

Converts undirected active subnetworks produced by subnetwork detection methods (e.g., BioNet, COSINE, SEMtree ST/STr2z) into directed trees using causal additive trees based on Chu–Liu–Edmonds' algorithm, so that competing methods can be compared on a common directed-subnetwork footing.

## When to use

Use when active subnetworks are returned as undirected graphs by multiple detection methods (e.g., from the Ma et al. 2011 simulation benchmark with 500 genes, 20 samples per condition) and you need directionality to compare them or fit tree-based SEMs.

## When NOT to use

- The subnetworks are already directed (e.g., KEGG-derived directed graphs used by SEMgraph directly) — conversion would add an unnecessary, possibly lossy orientation step.
- The goal is structural equation modeling on a large reference graph rather than benchmarking subnetwork detection — use SEMrun/SEMdag on the full graph instead of collapsing modules to trees.
- Undirected output is required by downstream analysis (e.g., Steiner tree backbone extraction or TAHC clustering in SEMgraph), where orientation is irrelevant or harmful.

## Inputs

- Undirected active subnetworks (igraph objects) from subnetwork detection methods (SEMTree ST, STr2z, BioNet, COSINE)
- Gene P-values computed per replicate from simulated case/control expression data (500 genes, 20 samples per condition)
- Reference network (e.g., KEGG interactome as an igraph object)

## Outputs

- Directed trees (arborescences) representing the active subnetworks, usable for directed method comparison
- Per-replicate and aggregated precision/recall/F1 metrics against the planted perturbed module (benchmark_metrics.csv)

## How to apply

Run each active subnetwork detection method (SEMTree() ST, STr2z, BioNet, COSINE, etc.) on gene P-values over a reference network to obtain perturbed modules with undirected edges. Feed each undirected module into a causal additive tree implementation that applies Chu–Liu–Edmonds' algorithm to orient edges, producing an arborescence (directed tree) per subnetwork. This conversion is the rationale for the benchmark: only after all subnetworks are directed can precision, recall, and F1 against the planted perturbed module be computed on a comparable basis. In SEMgraph, tree/arborescence structure can also be derived from correlation matrices, and the resulting igraph objects can be converted to lavaan SEM syntax for downstream SEMrun fitting and perturbation scoring (e.g., node/edge perturbation with group contrast, nonparanormal-transformed data).

## Related tools

- **SEMgraph** (R package providing SEMtree() ST/STr2z subnetwork detection and graph-to-SEM conversion; supports tree (arborescence) structure learning from correlation matrices and igraph objects for the directed-tree workflow) — https://github.com/fernandoPalluzzi/SEMgraph
- **igraph** (Graph representation format for the undirected subnetworks in and directed trees out of the conversion step)

## Evaluation signals

- Every detected subnetwork, regardless of source method, ends up as a directed tree (arborescence) before scoring — no method is scored on undirected output.
- Aggregated metrics over 100 randomizations reproduce the paper's pattern: BioNet achieves the highest precision; ST/STr2z reach ~90–80% precision; ~60% F1 for COSINE; ~30–40% F1 for ST, STr2z, and BioNet.
- Each replicate's directed subnetwork is scored against the known planted perturbed module, yielding per-replicate precision, recall, and F1 that can be checked against benchmark_metrics.csv.
- The conversion is applied uniformly to all six methods, so any precision/recall difference after conversion reflects detection quality, not edge-orientation discrepancies.

## Limitations

- The Chu–Liu–Edmonds conversion imposes a tree (arborescence) topology, discarding any cyclic or multi-parent structure present in the undirected active subnetwork.
- Edge orientation introduced by causal additive trees is data-driven and may not reflect the true causal direction of the planted perturbed module.
- Simulation benchmark reflects the Ma et al. 2011 design (500 genes, 20 samples per condition, 100 randomizations); results may not transfer to other network sizes or effect-strength regimes.
- Discussion section text is absent from the source document, limiting available caveats on the comparison.

## Evidence

- [abstract] Active subnetworks converted to directed trees for fair comparison: "The latter are supplied to causal additive trees based on Chu–Liu–Edmonds’ algorithm (Chow and Liu, Approximating discrete probability distributions with dependence trees."
- [abstract] Undirected subnetworks from five detection methods precede the conversion: "the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods."
- [abstract] Conversion rationale: comparability across methods: "This conversion allows to compare the methods in terms of directed active subnetworks."
- [results] SEMTree ST/STr2z benchmark precision on simulated data: [evidence span withheld — anchor status: not_found]
- [readme] SEMgraph supports tree (arborescence) learning as a native capability: [evidence span withheld — anchor status: external_doc]
