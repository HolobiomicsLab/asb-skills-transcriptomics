---
name: adjusted-rand-index-partition-comparison
description: Use when a clustering or differential-abundance method must be benchmarked
  against a baseline on data with ground-truth cell-state/condition labels, and partition
  agreement needs to be quantified with ARI plus chi-square statistics.
when_to_use_negative:
- Real datasets with no ground-truth partition — ARI against true labels cannot be
  computed; use DA-sensitivity benchmarking instead
- Data with strong clustering structure where the article notes ELVAR's improvement
  over its Louvain-analog is less noticeable
- Comparing against miloDE — the article explicitly did not benchmark this and flags
  it as an open question
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: ELVAR
  role: Provides Eva_partitions to generate EVA partitions across the alpha purity
    grid for ARI comparison against Louvain
  repo: https://github.com/aet21/ELVAR
- name: R
  role: Implementation language for the simulation, ARI computation, and chisq.test
    comparison
- name: Seurat
  role: Loads and preprocesses the TMS-derived counts and simulates the 200-cell monocyte
    dataset
- name: igraph
  role: Builds the undirected kNN state-manifold graph with self-loops removed and
    holds condition-label vertex attributes
- name: dplyr
  role: Tabulates ARI and chi-square results for validation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:c132ed4a24985e1d61ef1f99fb3fce423e45f3e250255956e12bc651545870b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/ELVAR
  tools:
  - ELVAR
  - EVA
  - Matrix
  - R
  - Seurat
  - dplyr
  - igraph
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# adjusted-rand-index-partition-comparison

## Summary

Benchmark cell-attribute aware clustering (EVA/ELVAR) against a Louvain baseline on simulated scRNA-Seq data with known ground-truth labels by computing adjusted Rand Index (ARI) between inferred and true partitions, complemented by chi-square tests of condition-by-state contingency tables.

## When to use

Use when a clustering or differential-abundance method must be benchmarked against a baseline on data with ground-truth cell-state/condition labels, and partition agreement needs to be quantified with ARI plus chi-square statistics.

## When NOT to use

- Real datasets with no ground-truth partition — ARI against true labels cannot be computed; use DA-sensitivity benchmarking instead
- Data with strong clustering structure where the article notes ELVAR's improvement over its Louvain-analog is less noticeable
- Comparing against miloDE — the article explicitly did not benchmark this and flags it as an open question

## Inputs

- Simulated scRNA-Seq count matrix with known condition labels (e.g. Tabula Muris Senis-derived monocyte counts, 200 cells)
- Ground-truth cell-state assignment vector for each simulated cell
- Cell-cell k-nearest-neighbour state-manifold graph (igraph undirected, self-loops removed) with condition label as vertex attribute
- Inferred partitions from EVA (alpha = 0.1–0.9) and Louvain (alpha = 0)

## Outputs

- Table of adjusted Rand Index values per method and per alpha parameter value
- Chi-square statistic and P-values from condition-by-state contingency tests (chisq.test) for enriched communities
- Ranking of EVA vs Louvain clustering recovery identifying the optimal alpha

## How to apply

Simulate a labeled scRNA-Seq dataset (e.g. the 200-cell classical monocyte perturbation model: 100 perturbed vs 100 normal cells built from Tabula Muris Senis lung counts in R/Seurat), retaining ground-truth cell-state assignments. Build the undirected cell-cell kNN state-manifold graph with igraph (self-loops removed) and attach the simulated condition label as a vertex attribute. Run ELVAR's Eva_partitions over the alpha purity grid 0.1–0.9 (resolution=1, threshold=1e-7), and run the standard Louvain baseline (alpha=0 behaviour) as reference. For each method and parameter value, compute the adjusted Rand Index between the inferred partition and the ground-truth simulated labels; separately build condition-by-state contingency tables and compare differential-abundance calls with chisq.test. The comparison is judged successful when EVA at the optimal alpha recovers higher ARI and more significant chi-square P-values than Louvain, consistent with the Supplementary Fig. S1 benchmark.

## Related tools

- **ELVAR** (Provides Eva_partitions to generate EVA partitions across the alpha purity grid for ARI comparison against Louvain) — https://github.com/aet21/ELVAR
- **R** (Implementation language for the simulation, ARI computation, and chisq.test comparison)
- **Seurat** (Loads and preprocesses the TMS-derived counts and simulates the 200-cell monocyte dataset)
- **igraph** (Builds the undirected kNN state-manifold graph with self-loops removed and holds condition-label vertex attributes)
- **dplyr** (Tabulates ARI and chi-square results for validation)

## Examples

```
library(ELVAR); eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha=0.8, Vattr.name="condition"); adjustedRandIndex(membership(eva.o), truth.labels)
```

## Evaluation signals

- EVA at the optimal alpha yields a strictly higher adjusted Rand Index against the ground-truth simulated labels than the Louvain baseline
- Chi-square P-values for EVA-enriched communities are more significant than for Louvain-enriched communities on the same condition-by-state tables
- Improvement should appear only at larger alpha values, matching the Supplementary Fig. S1 benchmark pattern
- ARI values fall in [−1, 1] with ~1 indicating perfect recovery of the 100 perturbed / 100 normal simulated structure
- Graph construction invariants hold: igraph object is undirected with self-loops removed and every vertex carries the condition attribute

## Limitations

- The EVA-over-Louvain improvement was observed only at larger purity parameter (alpha) values
- ELVAR carries increased computational complexity and runtimes compared to Milo or the Louvain-analog, though deemed not practically prohibitive
- ELVAR is more sensitive to false positives in the clustering attribute (sample/phenotype label), which can affect quality of inferred communities

## Evidence

- [results] Compute the adjusted Rand Index between each inferred partition and the ground-truth simulated labels to quantify clustering recovery for EVA versus Louvain.: [evidence span withheld — anchor status: not_found]
- [methods] eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha= 0.8, Vattr.name= "age");: "eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha= 0.8, Vattr.name= "age");"
- [discussion] the improvement of ELVAR over its Louvain-analog was more pronounced in studies displaying weaker clustering structure: "the improvement of ELVAR over its Louvain-analog was more pronounced in studies displaying weaker clustering structure (e.g. the immune-cell subsets changing with age in lung tissue, or the stem-cell"
- [readme] It implements an Extended Louvain clustering Algorithm (EVA) that takes cell attribute information into acccount when inferring cellular communities from the cell-cell nearest neighbour graph.: "It implements an Extended Louvain clustering Algorithm (EVA) that takes cell attribute information into acccount when inferring cellular communities from the cell-cell nearest neighbour graph."
