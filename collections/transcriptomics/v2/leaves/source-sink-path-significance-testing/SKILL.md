---
name: source-sink-path-significance-testing
description: Use when comparing perturbation strength of several extracted directed
  subnetworks (e.g., seven CAT modules) by testing source-to-sink path significance
  on case/control RNA-seq data and assembling a comparative table of ACEs%, combined
  path P-values, and DEG counts.
when_to_use_negative:
- Input expression data have not been nonparanormal-transformed and violate SEM normality
  assumptions
- The graph is not a directed subnetwork with identifiable source and sink nodes (e.g.,
  an undirected active module before Chu-Liu-Edmonds tree conversion)
- The goal is differential node or edge perturbation of a single fixed model (use
  SEMrun fit=1/fit=2) rather than source-sink path-level ACE testing across multiple
  candidate subnetworks
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_2258
tools:
- name: SEMgraph
  role: Provides transformData() for nonparanormal transformation and SEMace() for
    optimal source2sink ACE estimation with Bonferroni/BH adjustment at alpha=0.05
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMtree()
  role: Extracts the directed tree subnetworks (e.g., Kou's Steiner tree backbone)
    whose source-sink paths are tested
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMgsa
  role: Permutation-based (n_rep=5000) gene set analysis supplying per-subnetwork
    DEG counts and perturbation direction
  repo: https://github.com/fernandoPalluzzi/SEMgraph/tree/master/SEMgsa
- name: SEMdata
  role: Supplies the COVID-19 RNA-seq dataset (GSE172114) and KEGG reference network
    used to build the CAT subnetworks
  repo: https://github.com/fernandoPalluzzi/SEMdata
- name: igraph
  role: Graph representation for the directed subnetworks passed to SEMace()
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:c50864d5625d032d348bf7db06a6d9df35c7c4150f30685ffb65b092215930d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - R
  - SEMacE
  - SEMdata
  - SEMgraph
  - SEMgsa
  - SEMtree()
  - igraph
  - org.Hs.eg.db
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# source-sink-path-significance-testing

## Summary

Quantifies subnetwork perturbation by testing Average Causal Effects (ACEs) over all source-to-sink directed paths with SEMace(), counting significant paths (>2 nodes) as percentages and combining path P-values (Bonferroni) into PVAL(E), enabling ranking of candidate CAT subnetworks.

## When to use

Use when comparing perturbation strength of several extracted directed subnetworks (e.g., seven CAT modules) by testing source-to-sink path significance on case/control RNA-seq data and assembling a comparative table of ACEs%, combined path P-values, and DEG counts.

## When NOT to use

- Input expression data have not been nonparanormal-transformed and violate SEM normality assumptions
- The graph is not a directed subnetwork with identifiable source and sink nodes (e.g., an undirected active module before Chu-Liu-Edmonds tree conversion)
- The goal is differential node or edge perturbation of a single fixed model (use SEMrun fit=1/fit=2) rather than source-sink path-level ACE testing across multiple candidate subnetworks

## Inputs

- Nonparanormal-transformed RNA-seq expression matrix (data.npn) from COVID-19 dataset GSE172114 (46 critical vs 23 non-critical cases)
- Binary case/control group vector
- Directed subnetwork graphs (igraph objects) from the seven CAT extraction methods (e.g., SEMtree Kou Steiner tree, TAHC, EBC, WTC)
- Source and sink node definitions per subnetwork

## Outputs

- Per-path ACE table with P-values from SEMace() over all source2sink pairs
- Count and percentage of significant source-sink paths with >2 nodes per subnetwork (ACEs%)
- Bonferroni-combined path P-values PVAL(E) and node P-value combination PVAL(V)
- Per-subnetwork DEG counts and perturbation direction from SEMgsa() (n_rep=5000)
- Comparative table (table2_cat_perturbation_comparison.csv) ranking the seven CAT subnetworks by perturbation strength

## How to apply

First transform the expression matrix with SEMgraph::transformData() to a nonparanormal scale so SEM normality assumptions are relaxed (e.g., data.npn <- transformData(alsData$exprs)$data). For each directed subnetwork (here the seven CAT modules extracted from the COVID-19 GSE172114 case/control data), run SEMace() with type="optimal", effect="source2sink", alpha=0.05, and Bonferroni adjustment to test every source-sink pair. Count significant ACEs restricted to paths with >2 nodes (as in the ALS example, which found 3 such ACEs) and express them as a percentage of tested paths. Combine path P-values with Bonferroni into PVAL(E) (and node P-values into PVAL(V)), then run SEMgsa() with n_rep=5000 permutations to attach per-subnetwork DEG counts and perturbation direction. Rank subnetworks by perturbation strength; on the COVID-19 benchmark the Kou Steiner tree (ST) backbone from SEMtree() shows the strongest perturbation (highest ACEs%, lowest PVAL(E)).

## Related tools

- **SEMgraph** (Provides transformData() for nonparanormal transformation and SEMace() for optimal source2sink ACE estimation with Bonferroni/BH adjustment at alpha=0.05) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMtree()** (Extracts the directed tree subnetworks (e.g., Kou's Steiner tree backbone) whose source-sink paths are tested) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMgsa** (Permutation-based (n_rep=5000) gene set analysis supplying per-subnetwork DEG counts and perturbation direction) — https://github.com/fernandoPalluzzi/SEMgraph/tree/master/SEMgsa
- **SEMdata** (Supplies the COVID-19 RNA-seq dataset (GSE172114) and KEGG reference network used to build the CAT subnetworks) — https://github.com/fernandoPalluzzi/SEMdata
- **igraph** (Graph representation for the directed subnetworks passed to SEMace())

## Examples

```
ace <- SEMace(graph = graph, data = data.npn, type = "optimal", effect = "source2sink", method = "bonferroni", alpha = 0.05)
```

## Evaluation signals

- On the COVID-19 GSE172114 benchmark the table reproduces Table 2's comparative pattern with the ST (Kou Steiner tree) subnetwork as most perturbed: highest ACEs% and lowest Bonferroni-combined PVAL(E)
- The ACE count step flags only paths with >2 nodes; the article's ALS example yields exactly 3 significant ACEs with >2 nodes as a sanity check
- All ACE tests use alpha=0.05 with a multiple-testing adjustment (Bonferroni per the workflow; BH shown in the article call)
- Each of the seven CAT subnetworks contributes one row with method name, % significant paths, combined ACE P-value, and DEG count from SEMgsa (n_rep=5000)
- Expression data are nonparanormal-transformed (transformData) before SEM fitting; skipping the transform invalidates downstream P-values

## Limitations

- SEM-based ACE testing assumes (near-)normal variables; the nonparanormal transform is required to relax this assumption on RNA-seq data
- Only paths with >2 nodes are counted, so shorter two-step source-sink signals are excluded from ACEs%
- Bonferroni combination over all source-sink pairs is conservative and can mask weakly perturbed subnetworks
- Results are specific to the extracted directed trees; converting undirected active subnetworks via causal additive trees (Chu-Liu-Edmonds) is a prerequisite for method comparison

## Evidence

- [methods] Average Causal Effect (ACE) estimation (SEMace): "ace <- SEMace(graph = alsData$graph, data = data.npn, type = "optimal", effect = "source2sink", method = "BH", alpha = 0.05)"
- [methods] Nonparanormal data transformation: "we apply a nonparanormal transform to relax the normality assumption."
- [methods] Significant ACE counting: "## Found 3 significant ACEs with > 2 nodes"
- [readme] SEMgraph functionality scope: "Perturbed paths finding, community searching and sample scoring, together with graph plotting utilities, tracing model architecture modifications and perturbation"
- [abstract] COVID-19 benchmark dataset: "We applied SEMtree() to both Coronavirus disease (COVID-19) RNA-seq dataset (GEO accession: GSE172114) and simulated datasets with various differential expression patterns."
