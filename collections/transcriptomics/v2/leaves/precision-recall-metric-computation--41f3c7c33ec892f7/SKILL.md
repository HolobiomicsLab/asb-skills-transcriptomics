---
name: precision-recall-metric-computation
description: Use when multiple active-subnetwork detection methods are run on simulated
  datasets with a known perturbed module, and quantitative precision/recall/F1 benchmarking
  against the ground truth is required.
when_to_use_negative:
- The input is real experimental data (e.g., ALS, COVID-19, or FTD datasets) with
  no known ground-truth perturbed module, so precision/recall against a planted module
  cannot be defined.
- The goal is to run subnetwork detection itself rather than to benchmark detection
  methods that have already produced candidate subnetworks.
- Methods are to be compared on undirected subnetworks without the Chu–Liu–Edmonds
  directed-tree conversion, since the article's comparison is framed in terms of directed
  active subnetworks.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: SEMtreet() / SEMgraph
  role: Provides the ST (Steiner tree) and STr2z (Fisher r-to-z weighted tree) perturbation-extraction
    methods whose subnetworks are scored with precision/recall/F1 in the benchmark
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: igraph
  role: Graph representation for the active subnetworks and directed trees over which
    set-overlap metrics are computed
- name: R
  role: Implementation environment for the metric computation and aggregation over
    the 100 simulation randomizations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:3f13f3c7ecc24b96c075eb6116ade7222785f84558668584900e722f9ff31962
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - R
  - SEMgraph
  - SEMtree()
  - SEMtreet() / SEMgraph
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# precision-recall-metric-computation

## Summary

Scores subnetwork detections against a known planted perturbed module by computing per-replicate and aggregated precision, recall, and F1, enabling benchmark comparison of methods such as SEMtree() ST/STr2z, BioNet, and COSINE on simulated case/control data.

## When to use

Use when multiple active-subnetwork detection methods are run on simulated datasets with a known perturbed module, and quantitative precision/recall/F1 benchmarking against the ground truth is required.

## When NOT to use

- The input is real experimental data (e.g., ALS, COVID-19, or FTD datasets) with no known ground-truth perturbed module, so precision/recall against a planted module cannot be defined.
- The goal is to run subnetwork detection itself rather than to benchmark detection methods that have already produced candidate subnetworks.
- Methods are to be compared on undirected subnetworks without the Chu–Liu–Edmonds directed-tree conversion, since the article's comparison is framed in terms of directed active subnetworks.

## Inputs

- Detected active subnetwork (node set) per method per replicate, from six subnetwork detection methods (SEMtreet ST, STr2z, BioNet, COSINE, etc.)
- Planted (ground-truth) perturbed module from each simulated case/control gene-expression dataset (500 genes, 20 samples per condition)
- Undirected active subnetworks converted to directed trees via Chu–Liu–Edmonds causal additive trees

## Outputs

- Per-replicate precision, recall, and F1 for each method and dataset
- Aggregated mean precision and F1 over 100 randomizations per method per dataset
- benchmark_metrics.csv containing per-replicate and aggregated metrics

## How to apply

For each replicate, take the detected active subnetwork from each method (ST, STr2z via SEMtree(), plus BioNet, COSINE, and other state-of-the-art detectors) and score it against the planted perturbed module embedded in the simulated dataset (500 genes, 20 samples per condition, per the Ma et al. 2011 design). Compute precision as the fraction of detected nodes in the planted module, recall as the fraction of planted nodes recovered, and F1 as their harmonic mean, per replicate and per method. First convert all undirected active subnetworks into directed trees using causal additive trees based on Chu–Liu–Edmonds' algorithm, so that every method is compared on directed active subnetworks. Repeat over 100 randomizations for each of the four simulated case datasets, then aggregate precision and F1 per method per dataset. Write both per-replicate and aggregated metrics to benchmark_metrics.csv.

## Related tools

- **SEMtreet() / SEMgraph** (Provides the ST (Steiner tree) and STr2z (Fisher r-to-z weighted tree) perturbation-extraction methods whose subnetworks are scored with precision/recall/F1 in the benchmark) — https://github.com/fernandoPalluzzi/SEMgraph
- **igraph** (Graph representation for the active subnetworks and directed trees over which set-overlap metrics are computed)
- **R** (Implementation environment for the metric computation and aggregation over the 100 simulation randomizations)

## Examples

```
m <- read.csv("benchmark_metrics.csv"); aggregate(cbind(precision, F1) ~ method + dataset, m, mean)
```

## Evaluation signals

- Aggregated mean precision for SEMtree() ST and STr2z falls in the ~90–80% range across all four simulated case datasets
- BioNet attains the highest precision (it recovers the smallest subnetworks), consistent with the paper's reported pattern
- Mean F1 is approximately 60% for COSINE and approximately 30–40% for ST, STr2z, and BioNet
- benchmark_metrics.csv exists and contains both per-replicate rows and per-method, per-dataset aggregates over 100 randomizations
- All compared subnetworks are directed trees (converted via Chu–Liu–Edmonds causal additive trees), so metrics are computed on comparable directed node/edge sets

## Limitations

- High precision (~90–80%) for ST/STr2z does not imply high overall performance: F1 remains only ~30–40%, indicating low recall relative to the planted module
- The comparison depends on converting undirected active subnetworks to directed trees, which may not reflect each method's native edge semantics
- Results are specific to the Ma et al. 2011-style simulation design (500 genes, 20 samples per condition, 100 randomizations, four differential-expression patterns)

## Evidence

- [abstract] Compared to existing methods, SEMtree() is able to capture biologically relevant subnetworks with simple visualization of directed paths, good perturbation extraction, and classifier performance.: "Compared to existing methods, SEMtree() is able to capture biologically relevant subnetworks with simple visualization of directed paths, good perturbation extraction, and classifier performance."
- [abstract] from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods.: "from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods."
- [abstract] This conversion allows to compare the methods in terms of directed active subnetworks.: "This conversion allows to compare the methods in terms of directed active subnetworks."
- [methods] Generate four simulated case/control gene-expression datasets with 500 genes and 20 samples per condition, embedding different differential-expression patterns in a known reference network, repeating the simulation 100 times: [evidence span withheld — anchor status: not_found]
- [results] On the simulated data, SEMtree() ST and STr2z achieve high precision, around 90%−80% for all case datasets, just below the precision of BioNet, which recovers the smallest subnetworks.: [evidence span withheld — anchor status: not_found]
- [readme] Tree structure learning methods implemented with graph and data-driven SEM-based algorithms.: [evidence span withheld — anchor status: external_doc]
