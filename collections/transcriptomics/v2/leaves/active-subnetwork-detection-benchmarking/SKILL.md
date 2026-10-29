---
name: active-subnetwork-detection-benchmarking
description: Use when comparing subnetwork detection methods on simulated case/control
  expression data with known planted perturbed modules, so precision, recall, and
  F1 against the ground truth can be computed per replicate.
when_to_use_negative:
- Analyzing a single real dataset (e.g. ALS GSE124439 or COVID-19 GSE172114) for biologically
  interpretable DRNs/DREs — no planted ground truth exists, so precision/recall/F1
  cannot be computed
- Evaluating path-level or community-level SEM analyses (SEMpath, ACE, clustering)
  rather than whole-subnetwork detection method comparison
- Comparing methods that output inherently directed graphs when undirected-to-directed
  tree conversion would bias the evaluation
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SEMtree() / SEMgraph
  role: R package supplying the ST (Steiner tree backbone) and STr2z (Fisher r-to-z
    weighted) subnetwork detection methods under benchmark, plus SEMrun for perturbation
    P-values
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMdata
  role: Companion package providing reference networks (KEGG, Reactome, STRING) and
    training data for constructing the simulation backbone
  repo: https://github.com/fernandoPalluzzi/SEMdata
- name: igraph
  role: Graph representation used for the undirected active subnetworks and directed
    tree conversion
- name: BioNet / COSINE
  role: State-of-the-art active subnetwork detection methods used as benchmark comparators
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:3872975049956b9345cb90299a3f71f68d6edf9a2a9e62c232fcb747b7a57843
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - BioNet / COSINE
  - R
  - SEMdata
  - SEMgraph
  - SEMtree()
  - SEMtree() / SEMgraph
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# active-subnetwork-detection-benchmarking

## Summary

Benchmark active subnetwork detection methods — SEMtree() ST and STr2z alongside BioNet, COSINE, and other state-of-the-art tools — on simulated case/control gene-expression data with planted perturbed modules, scoring precision, recall, and F1 over repeated randomizations.

## When to use

Use when comparing subnetwork detection methods on simulated case/control expression data with known planted perturbed modules, so precision, recall, and F1 against the ground truth can be computed per replicate.

## When NOT to use

- Analyzing a single real dataset (e.g. ALS GSE124439 or COVID-19 GSE172114) for biologically interpretable DRNs/DREs — no planted ground truth exists, so precision/recall/F1 cannot be computed
- Evaluating path-level or community-level SEM analyses (SEMpath, ACE, clustering) rather than whole-subnetwork detection method comparison
- Comparing methods that output inherently directed graphs when undirected-to-directed tree conversion would bias the evaluation

## Inputs

- Simulated case/control gene-expression datasets (500 genes, 20 samples per condition) with planted perturbed modules in a known reference network
- Gene P-values per replicate (e.g. from SEMrun node perturbation)
- Directed reference network (e.g. the KEGG interactome, 5007 nodes / 44755 edges)

## Outputs

- Directed active subnetworks (arborescences) per method and replicate after Chu–Liu–Edmonds conversion
- Per-replicate precision, recall, and F1 for each method
- Aggregated precision and F1 over 100 randomizations per dataset and method (benchmark_metrics.csv)

## How to apply

Following the Ma et al. 2011 design, generate four simulated case/control gene-expression datasets (500 genes, 20 samples per condition) embedding different differential-expression patterns in a known reference network, repeating each simulation 100 times. For every replicate, compute gene P-values and run the detection methods — SEMtree() ST (Steiner tree backbone via Kou's algorithm) and STr2z (Fisher r-to-z weighting of group correlation differences) from SEMgraph, plus state-of-the-art active subnetwork detectors such as BioNet and COSINE — to extract perturbed modules with undirected edges. Convert each undirected subnetwork into a directed tree with causal additive trees based on Chu–Liu–Edmonds' algorithm so all methods are compared on directed active subnetworks. Score every detected subnetwork against the planted module for precision, recall, and F1 per replicate, then aggregate over the 100 randomizations per dataset and method.

## Related tools

- **SEMtree() / SEMgraph** (R package supplying the ST (Steiner tree backbone) and STr2z (Fisher r-to-z weighted) subnetwork detection methods under benchmark, plus SEMrun for perturbation P-values) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMdata** (Companion package providing reference networks (KEGG, Reactome, STRING) and training data for constructing the simulation backbone) — https://github.com/fernandoPalluzzi/SEMdata
- **igraph** (Graph representation used for the undirected active subnetworks and directed tree conversion)
- **BioNet / COSINE** (State-of-the-art active subnetwork detection methods used as benchmark comparators)

## Examples

```
data.npn <- transformData(alsData$exprs)$data; sem1 <- SEMrun(graph = alsData$graph, data = data.npn, group = alsData$group)
```

## Evaluation signals

- Aggregated precision for SEMtree() ST and STr2z falls in the ~90–80% range across all four case datasets, just below BioNet, which attains the highest precision with the smallest subnetworks
- F1 patterns reproduce the paper: ~60% for COSINE and ~30–40% for ST, STr2z, and BioNet
- Exactly 100 randomizations × 4 simulated datasets are aggregated per method; per-replicate and aggregated rows are both present in benchmark_metrics.csv
- All methods are compared on the same footing: every undirected subnetwork has been converted to a directed tree via causal additive trees (Chu–Liu–Edmonds) before scoring

## Limitations

- The Discussion section text is absent from the source document, so caveats discussed by the authors are not available
- SEMtree ST and STr2z show lower F1 (~30–40%) than COSINE (~60%), trading recall for high precision (~90–80%)
- BioNet recovers the smallest subnetworks, so method rankings are sensitive to subnetwork size differences

## Evidence

- [abstract] from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods.: "from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods."
- [abstract] The latter are supplied to causal additive trees based on Chu–Liu–Edmonds’ algorithm: "The latter are supplied to causal additive trees based on Chu–Liu–Edmonds’ algorithm"
- [abstract] This conversion allows to compare the methods in terms of directed active subnetworks.: "This conversion allows to compare the methods in terms of directed active subnetworks."
- [abstract] We applied SEMtree() to both Coronavirus disease (COVID-19) RNA-seq dataset (GEO accession: GSE172114) and simulated datasets with various differential expression patterns.: "We applied SEMtree() to both Coronavirus disease (COVID-19) RNA-seq dataset (GEO accession: GSE172114) and simulated datasets with various differential expression patterns."
- [readme] Tree structure learning methods implemented with graph and data-driven SEM-based algorithms.: [evidence span withheld — anchor status: external_doc]
