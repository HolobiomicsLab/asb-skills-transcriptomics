---
name: gene-set-perturbation-analysis
description: Use when RNA-seq expression data with case/control group labels (e.g.
  GEO GSE172114) must be mapped onto extracted network submodules and compared for
  perturbation strength via source-sink path ACEs and Bonferroni-combined P-values
  plus DEG counts.
when_to_use_negative:
- Data are not high-throughput gene expression with a binary case/control grouping
  — SEMrun/SEMac perturbation testing requires a group vector to contrast conditions
- The input graph is already a perturbation result table; this skill operates on igraph
  network modules, not on precomputed pathway enrichment tables
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SEMace()
  role: Estimates Average Causal Effects over source2sink paths per subnetwork with
    BH/Bonferroni adjustment at alpha=0.05 to produce ACEs% and PVAL(E)
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMgsa()
  role: SEM-based gene set analysis with 5000 permutations to obtain per-subnetwork
    DEG counts and perturbation direction
  repo: https://github.com/fernandoPalluzzi/SEMgraph/tree/master/SEMgsa
- name: SEMgraph
  role: R package providing SEMrun/SEMac/SEMgsa perturbation functions on igraph objects
    and lavaan-syntax models
  repo: https://CRAN.R-project.org/package=SEMgraph
- name: SEMdata
  role: Companion package supplying the COVID-19 RNA-seq dataset (GSE172114), KEGG
    pathways, and reference networks used as inputs
  repo: https://github.com/fernandoPalluzzi/SEMdata
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:0825852df8c3d6fb36c6065adc9ed20c8f2f9ec1ee406ef7a9962823e821c182
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - R
  - SEMacE
  - SEMace()
  - SEMdata
  - SEMgraph
  - SEMgsa
  - SEMgsa()
  - SEMtree()
  - org.Hs.eg.db
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# gene-set-perturbation-analysis

## Summary

Quantifies condition-specific perturbation of graph/pathway modules with SEMgraph: SEMace() computes Average Causal Effects (ACEs) over source2sink paths at alpha=0.05, and SEMgsa() ranks perturbed gene sets via 5000-permutation SEM-based testing, yielding ACEs%, PVAL(E)/PVAL(V), and No.DEGs per subnetwork.

## When to use

Use when RNA-seq expression data with case/control group labels (e.g. GEO GSE172114) must be mapped onto extracted network submodules and compared for perturbation strength via source-sink path ACEs and Bonferroni-combined P-values plus DEG counts.

## When NOT to use

- Data are not high-throughput gene expression with a binary case/control grouping — SEMrun/SEMac perturbation testing requires a group vector to contrast conditions
- The input graph is already a perturbation result table; this skill operates on igraph network modules, not on precomputed pathway enrichment tables

## Inputs

- RNA-seq expression matrix with case/control group labels (e.g. SEMdata COVID-19 dataset, GEO accession GSE172114)
- Extracted CAT subnetworks as igraph objects (e.g. Steiner tree ST, TAHC, EBC, WTC modules)
- org.Hs.eg.db for Entrez-to-symbol gene ID conversion

## Outputs

- Per-subnetwork perturbation table: method name, % significant source-sink paths (ACEs%), Bonferroni-combined ACE P-value (PVAL(E)), node P-value combination (PVAL(V)), and DEG count (No.DEGs)
- Ranked comparison of subnetworks by perturbation strength (e.g. table2_cat_perturbation_comparison.csv)
- SEMgsa() output listing perturbed pathways/genes with adjusted P-values and perturbation direction

## How to apply

First apply the nonparanormal transform with transformData() to relax the normality assumption on the expression matrix (e.g. the COVID-19 dataset GSE172114). For each candidate subnetwork, run SEMace() with type='optimal', effect='source2sink', method='BH' (or Bonferroni for combined path P-values), alpha=0.05, then count significant source-sink paths with > 2 nodes and express them as a percentage (ACEs%) of tested paths; also record the Bonferroni-combined ACE P-value PVAL(E) and combined node P-value PVAL(V). Run SEMgsa() on each subnetwork with n_rep=5000 permutations to obtain per-module DEG counts (No.DEGs) and perturbation direction. Assemble one row per subnetwork (method name, ACEs%, PVAL(E), No.DEGs) and rank modules by perturbation strength; in the COVID-19 benchmark, the Kou's Steiner tree (ST) module emerged as the most perturbed.

## Related tools

- **SEMace()** (Estimates Average Causal Effects over source2sink paths per subnetwork with BH/Bonferroni adjustment at alpha=0.05 to produce ACEs% and PVAL(E)) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMgsa()** (SEM-based gene set analysis with 5000 permutations to obtain per-subnetwork DEG counts and perturbation direction) — https://github.com/fernandoPalluzzi/SEMgraph/tree/master/SEMgsa
- **SEMgraph** (R package providing SEMrun/SEMac/SEMgsa perturbation functions on igraph objects and lavaan-syntax models) — https://CRAN.R-project.org/package=SEMgraph
- **SEMdata** (Companion package supplying the COVID-19 RNA-seq dataset (GSE172114), KEGG pathways, and reference networks used as inputs) — https://github.com/fernandoPalluzzi/SEMdata

## Examples

```
ace <- SEMace(graph = st.graph, data = data.npn, type = "optimal", effect = "source2sink", method = "BH", alpha = 0.05)
```

## Evaluation signals

- The ST (Kou's Steiner tree) subnetwork must rank as the most perturbed module, combining the highest ACEs% and lowest PVAL(E) — reproducing the paper's Table 2 comparative pattern
- SEMac() output must list only significant ACEs with > 2 nodes; percentage computed relative to tested source-sink paths
- SEMgsa() must report per-subnetwork DEG counts consistent across n_rep=5000 permutation replicates with a defined perturbation direction
- Combined P-values must reflect Bonferroni adjustment across all source-sink pairs per subnetwork

## Limitations

- Perturbation direction and P-values depend on the nonparanormal transform; skipping transformData() violates the normality-relaxation assumption the workflow depends on
- The KEGG/Reactome pathways and reference networks in SEMdata are freezed to benchmarked versions; newer database versions may change module composition and perturbation rankings
- The article's discussion text is unavailable, so no explicit author-stated caveats on generalizability beyond the applied datasets

## Evidence

- [methods] SEMac call grounding ACE estimation: "ace <- SEMace(graph = alsData$graph, data = data.npn, type = "optimal", effect = "source2sink", method = "BH", alpha = 0.05)"
- [methods] Nonparanormal transform rationale: "we apply a nonparanormal transform to relax the normality assumption."
- [methods] SEMgsa perturbation output demonstrated on pathway sets: [evidence span withheld — anchor status: not_found]
- [abstract] Comparison scope of perturbed modules from seed genes: "from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods."
- [readme] SEMdata provides the benchmark data and networks: "It contains useful high-throughput sequencing data, reference networks, and pathways for SEMgraph training"
