---
name: monte-carlo-replication-aggregation
description: Use when comparing subnetwork detection methods (e.g., SEMtree ST/STr2z,
  BioNet, COSINE) on simulated gene-expression data where a planted perturbed module
  provides ground truth and per-replicate metrics must be aggregated over randomizations.
when_to_use_negative:
- When the input is a single real RNA-seq dataset (e.g., the COVID-19 GSE172114 or
  ALS GSE124439 studies) with no planted ground-truth module — use the perturbation
  workflow (SEMrun node/edge testing) instead of the simulation benchmark
- When no reference network or planted differential-expression module exists, since
  precision/recall/F1 against ground truth cannot be computed
- When the goal is de novo network inference rather than head-to-head comparison of
  already-implemented subnetwork detection methods
edam_operation: null
tools:
- name: SEMtree()
  role: Provides the ST and STr2z tree-based subnetwork detection methods being benchmarked
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMgraph
  role: R package implementing SEMtree() and the perturbation/filtering routines used
    to extract perturbed modules from gene P-values
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: igraph
  role: Represents the reference network and detected active subnetworks as igraph
    objects during simulation scoring
- name: R
  role: Language in which the simulation loop, metric computation, and CSV export
    are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:90c4c0928af213303c4e4a7d8c21f816398c3ee684b77b3e883d5681b49ad2ed
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
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Monte Carlo replication aggregation

## Summary

A simulation-based benchmarking skill that repeats an active-subnetwork detection study across 100 randomized simulated case/control datasets and aggregates per-replicate precision, recall, and F1 into a single benchmark_metrics.csv for method comparison.

## When to use

Use when comparing subnetwork detection methods (e.g., SEMtree ST/STr2z, BioNet, COSINE) on simulated gene-expression data where a planted perturbed module provides ground truth and per-replicate metrics must be aggregated over randomizations.

## When NOT to use

- When the input is a single real RNA-seq dataset (e.g., the COVID-19 GSE172114 or ALS GSE124439 studies) with no planted ground-truth module — use the perturbation workflow (SEMrun node/edge testing) instead of the simulation benchmark
- When no reference network or planted differential-expression module exists, since precision/recall/F1 against ground truth cannot be computed
- When the goal is de novo network inference rather than head-to-head comparison of already-implemented subnetwork detection methods

## Inputs

- Simulated case/control gene-expression data matrices (500 genes, 20 samples per condition, 100 randomization replicates)
- Gene P-values computed per replicate
- Reference network as an igraph object (e.g., a KEGG-derived interactome)
- Planted ground-truth perturbed module per simulation design

## Outputs

- Per-replicate precision, recall, and F1 per method (six methods x four datasets x 100 replicates)
- Aggregated precision and F1 over the 100 randomizations per dataset and method
- benchmark_metrics.csv containing per-replicate and aggregated metrics

## How to apply

Following the Ma et al. 2011 design, generate four simulated case/control gene-expression datasets (500 genes, 20 samples per condition) with different differential-expression patterns embedded in a known reference network, repeating the whole simulation 100 times (randomizations). For each replicate, compute gene P-values and run the six subnetwork detection methods — SEMtree() ST and STr2z from SEMgraph plus state-of-the-art active subnetwork methods including BioNet and COSINE — extracting perturbed modules with undirected edges. Convert each undirected active subnetwork into a directed tree using causal additive trees based on Chu–Liu–Edmonds' algorithm so all methods are scored on directed active subnetworks. Score every detected subnetwork against the planted perturbed module, computing precision, recall, and F1 per replicate and per method. Aggregate precision and F1 over the 100 randomizations for each of the four datasets and write both per-replicate and aggregated metrics to benchmark_metrics.csv.

## Related tools

- **SEMtree()** (Provides the ST and STr2z tree-based subnetwork detection methods being benchmarked) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMgraph** (R package implementing SEMtree() and the perturbation/filtering routines used to extract perturbed modules from gene P-values) — https://github.com/fernandoPalluzzi/SEMgraph
- **igraph** (Represents the reference network and detected active subnetworks as igraph objects during simulation scoring)
- **R** (Language in which the simulation loop, metric computation, and CSV export are executed)

## Examples

```
sem1 <- SEMrun(graph = alsData$graph, data = data.npn, group = alsData$group)
```

## Evaluation signals

- Aggregated precision for SEMtree() ST and STr2z falls in the ~90–80% range across all four simulated case datasets
- BioNet attains the highest precision, consistent with it recovering the smallest subnetworks
- COSINE aggregated F1 is approximately 60%, while ST, STr2z, and BioNet land at approximately 30–40% F1
- benchmark_metrics.csv contains both per-replicate rows and 100-randomization aggregated rows for each of the four datasets and six methods
- Every replicate's detected subnetwork is compared on a directed tree representation after Chu–Liu–Edmonds conversion, not the raw undirected graph

## Limitations

- Benchmark conclusions are tied to the specific Ma et al. 2011 simulation design (500 genes, 20 samples per condition, 100 randomizations); different simulation parameters may shift the precision/F1 rankings
- Converting undirected active subnetworks into directed trees via causal additive trees introduces an additional structure-learning step that all methods undergo, which may not reflect each method's native output
- Discussion section text is absent from the source document, limiting interpretive guidance on why ST/STr2z trade precision for lower recall

## Evidence

- [abstract] SEMs tree simulation benchmark performance: "Compared to existing methods, SEMtree() is able to capture biologically relevant subnetworks with simple visualization of directed paths, good perturbation extraction, and classifier performance."
- [abstract] Perturbed modules from gene P-values: "from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods."
- [methods] STr2z Fisher r-to-z weighting: "the fastest of them is based on the Fisher's "r-to-z" method, to test the group difference between correlation coefficients"
- [readme] SEMgraph installation for benchmark: "devtools::install_github("fernandoPalluzzi/SEMgraph")"
