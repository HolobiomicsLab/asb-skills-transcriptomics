---
name: simulated-gene-expression-data-generation
description: 'Use when a controlled ground truth is needed to compare subnetwork detection
  methods: generate simulated case/control expression data with a known planted perturbed
  module inside a reference network.'
when_to_use_negative:
- The goal is analysis of a real dataset (e.g., GSE172114 COVID-19 or GSE124439 ALS
  RNA-seq); simulation is only for controlled benchmarking, not for biological interpretation.
- No ground-truth perturbed module can be planted or simulated, so precision/recall/F1
  against a known module cannot be computed.
tools:
- name: SEMtree()
  role: Downstream subnetwork detection method (ST and STr2z variants) evaluated on
    the simulated datasets; implemented in the SEMgraph R package
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMgraph
  role: R package providing SEMtree() and supporting graph/data handling (igraph or
    lavaan representation) for the simulated-data benchmark
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: igraph
  role: Graph representation used for the reference network and detected active subnetworks
    in the simulation benchmark
- name: SEMdata
  role: Companion package supplying high-throughput data, reference networks, and
    pathways usable for SEMgraph training/benchmarking
  repo: https://github.com/fernandoPalluzzi/SEMdata
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:1cb522533aba42a94abd8f6d93be37d9417b3de1f5b0abbec38627e1fe12d8f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - R
  - SEMdata
  - SEMgraph
  - SEMtree()
  - igraph
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# simulated-gene-expression-data-generation

## Summary

Generates synthetic case/control gene-expression datasets (500 genes, 20 samples per condition, 100 randomizations) with differential-expression patterns planted in a known reference network, following the Ma et al. 2011 design, to provide ground truth for benchmarking subnetwork detection methods such as SEMtree() ST/STr2z, BioNet, and COSINE.

## When to use

Use when a controlled ground truth is needed to compare subnetwork detection methods: generate simulated case/control expression data with a known planted perturbed module inside a reference network.

## When NOT to use

- The goal is analysis of a real dataset (e.g., GSE172114 COVID-19 or GSE124439 ALS RNA-seq); simulation is only for controlled benchmarking, not for biological interpretation.
- No ground-truth perturbed module can be planted or simulated, so precision/recall/F1 against a known module cannot be computed.

## Inputs

- Known reference network (e.g., an igraph object such as the KEGG interactome: 5007 nodes, 44755 edges)
- Simulation design parameters: 500 genes, 20 samples per condition, 4 differential-expression pattern variants, 100 randomizations
- Seed/planted perturbed gene list defining the differential-expression module

## Outputs

- Four simulated case/control gene-expression matrices (genes x samples) per replicate
- Per-replicate and aggregated precision, recall, and F1 per subnetwork method
- benchmark_metrics.csv containing per-replicate and aggregated metrics

## How to apply

Generate four simulated case/control gene-expression datasets with 500 genes and 20 samples per condition, embedding different differential-expression patterns in a known reference network, and repeat the simulation 100 times (randomizations) following the Ma et al. 2011 design. For each replicate, compute gene P-values that reflect the planted perturbation so that active subnetwork methods (SEMrerooted SEMtree() ST and STr2z from SEMgraph, plus BioNet, COSINE, and other state-of-the-art detectors) can be run against known truth. Score every detected subnetwork against the planted module by precision, recall, and F1 per replicate and method, then aggregate over the 100 randomizations per dataset and method. Write per-replicate and aggregated metrics to benchmark_metrics.csv so the benchmark pattern can be verified against published values. The rationale is that the SEMtree() paper demonstrates method properties on both real (GSE172114 COVID-19 RNA-seq) and simulated datasets with various differential-expression patterns, and simulation is what makes precision/recall/F1 computable.

## Related tools

- **SEMtree()** (Downstream subnetwork detection method (ST and STr2z variants) evaluated on the simulated datasets; implemented in the SEMgraph R package) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMgraph** (R package providing SEMtree() and supporting graph/data handling (igraph or lavaan representation) for the simulated-data benchmark) — https://github.com/fernandoPalluzzi/SEMgraph
- **igraph** (Graph representation used for the reference network and detected active subnetworks in the simulation benchmark)
- **SEMdata** (Companion package supplying high-throughput data, reference networks, and pathways usable for SEMgraph training/benchmarking) — https://github.com/fernandoPalluzzi/SEMdata

## Examples

```
devtools::install_github("fernandoPalluzzi/SEMgraph"); devtools::install_github("fernandoPalluzzi/SEMdata")
```

## Evaluation signals

- Each simulation replicate contains 500 genes and 20 samples per condition, with 100 randomizations per dataset and 4 datasets total.
- Aggregated precision for SEMtree() ST and STr2z is approximately 90-80% across all case datasets.
- BioNet shows the highest precision (recovering the smallest subnetworks); COSINE reaches approximately 60% F1.
- ST, STr2z, and BioNet each achieve approximately 30-40% F1, matching the published benchmark pattern.
- benchmark_metrics.csv contains both per-replicate and aggregated precision/recall/F1 for all six methods.

## Limitations

- Precision-recall/F1 conclusions depend on the planted differential-expression patterns; results reflect the specific Ma et al. 2011 simulation design, not necessarily real perturbation biology.
- Simulated active subnetworks must be converted into directed trees (causal additive trees, Chu-Liu-Edmonds' algorithm) before fair comparison across methods.
- The paper's discussion section text is absent from the source document, so limitations of the simulation design are not fully documented.

## Evidence

- [abstract] Simulated datasets for benchmarking: "We applied SEMtree() to both Coronavirus disease (COVID-19) RNA-seq dataset (GEO accession: GSE172114) and simulated datasets with various differential expression patterns."
- [methods] Data transformation for simulation-based SEM analysis: "we apply a nonparanormal transform to relax the normality assumption."
- [readme] Companion data package: [evidence span withheld — anchor status: external_doc]
