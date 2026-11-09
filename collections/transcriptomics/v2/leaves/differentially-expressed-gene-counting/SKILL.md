---
name: differentially-expressed-gene-counting
description: Use when comparing perturbation of multiple extracted subnetworks (e.g.,
  seven CAT modules on COVID-19 GSE172114 RNA-seq) and you need per-subnetwork DEG
  counts plus path-level ACE statistics to rank which module is most perturbed.
when_to_use_negative:
- Input is a single pre-specified pathway and you only need a pathway-level P-value,
  not a comparison of perturbation across multiple competing subnetwork extraction
  methods
- Data are already transformed/normalized expression values where a nonparanormal
  (transformData) step would be redundant or inappropriate
- The subnetworks are undirected or lack source-sink structure, since ACE estimation
  requires directed paths from source to sink
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0085
tools:
- name: SEMgraph
  role: Provides SEMace() for source-sink ACE P-value combination and hosts the SEMgsa
    gene set analysis tool used to obtain per-subnetwork DEG counts
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMdata
  role: Supplies the COVID-19 RNA-seq dataset (GSE172114) and reference networks used
    as inputs to the perturbation comparison
  repo: https://github.com/fernandoPalluzzi/SEMdata
- name: org.Hs.eg.db
  role: Bioconductor package used for gene ID conversion (Entrez to gene symbols)
    when preparing graph node labels
- name: igraph
  role: Graph representation format for the input subnetworks and extracted directed
    paths
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:862438a2effcfa3ebbbedd844cf148bde857c44d257703b20b014cc0a5f43bb1
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# differentially-expressed-gene-counting

## Summary

Quantify system perturbation of extracted subnetworks by counting differentially expressed genes (No.DEGs) via SEM-based gene set analysis (SEMgsa) with permutation testing, alongside source-sink ACE path percentages and Bonferroni-combined P-values, to rank subnetworks by perturbation strength.

## When to use

Use when comparing perturbation of multiple extracted subnetworks (e.g., seven CAT modules on COVID-19 GSE172114 RNA-seq) and you need per-subnetwork DEG counts plus path-level ACE statistics to rank which module is most perturbed.

## When NOT to use

- Input is a single pre-specified pathway and you only need a pathway-level P-value, not a comparison of perturbation across multiple competing subnetwork extraction methods
- Data are already transformed/normalized expression values where a nonparanormal (transformData) step would be redundant or inappropriate
- The subnetworks are undirected or lack source-sink structure, since ACE estimation requires directed paths from source to sink

## Inputs

- COVID-19 RNA-seq expression matrix (GEO accession GSE172114, 46 critical vs 23 non-critical cases)
- Binary case/control group labels
- Seven extracted CAT subnetworks as igraph objects (e.g., Steiner tree / ST backbone and alternative methods)

## Outputs

- table2_cat_perturbation_comparison.csv with one row per CAT subnetwork: method name, ACEs% (significant source-sink paths with >2 nodes), Bonferroni-combined ACE P-value (PVAL(E)), and No.DEGs from SEMgsa (n_rep=5000)
- Ranking of subnetworks by system perturbation strength

## How to apply

Load the COVID-19 RNA-seq expression matrix and case/control group labels (GEO accession GSE172114) and apply the nonparanormal transform with transformData() to relax the normality assumption. For each extracted CAT subnetwork, run SEMace() with type='optimal' and effect='source2sink' at alpha=0.05 using Bonferroni adjustment to obtain combined ACE P-values (PVAL(E)) over all source-sink pairs, and count significant source-sink paths (paths with > 2 nodes) as a percentage of tested paths (ACEs%). Then run SEMgsa() on each subnetwork with n_rep = 5000 permutations to obtain the per-subnetwork DEG count (No.DEGs) and perturbation direction. Assemble one row per subnetwork listing method name, % significant paths, combined ACE P-value, and DEG count, and rank subnetworks by perturbation strength. Validate against the paper's Table 2 pattern, where Kou's Steiner tree (ST) backbone shows the strongest perturbation (lowest combined ACE P-value / highest % significant paths, with low PVAL(V)).

## Related tools

- **SEMgraph** (Provides SEMace() for source-sink ACE P-value combination and hosts the SEMgsa gene set analysis tool used to obtain per-subnetwork DEG counts) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMdata** (Supplies the COVID-19 RNA-seq dataset (GSE172114) and reference networks used as inputs to the perturbation comparison) — https://github.com/fernandoPalluzzi/SEMdata
- **org.Hs.eg.db** (Bioconductor package used for gene ID conversion (Entrez to gene symbols) when preparing graph node labels)
- **igraph** (Graph representation format for the input subnetworks and extracted directed paths)

## Examples

```
ace <- SEMace(graph = st.tree, data = data.npn, type = "optimal", effect = "source2sink", method = "bonferroni", alpha = 0.05)
```

## Evaluation signals

- Reproduces the paper's Table 2 comparative pattern: the ST (Kou's Steiner tree) subnetwork is identified as the most perturbed among the seven CAT subnetworks
- ACE path percentage (ACEs%) counts only significant source-sink paths with > 2 nodes, computed at alpha = 0.05 with Bonferroni adjustment
- SEMgsa DEG counts (No.DEGs) are obtained with n_rep = 5000 permutations and include a perturbation direction
- Output table has exactly one row per CAT subnetwork with columns: method, % significant paths, combined ACE P-value, and DEG count
- The most perturbed subnetwork combines the highest ACEs% and lowest PVAL(E) with low node P-value combination (PVAL(V))

## Limitations

- The discussion section text is absent from the source document, so caveats discussed by the authors beyond the methods are not recoverable
- DEG counts depend on the permutation budget (n_rep = 5000); smaller budgets will change P-value stability and hence the ranking
- The comparison is defined over seven specific CAT subnetwork extraction methods; rankings may not generalize to other subnetwork extraction approaches

## Evidence

- [methods] ACE estimation with optimal type and source2sink effect at 5% significance: "ace <- SEMace(graph = alsData$graph, data = data.npn, type = "optimal", effect = "source2sink", method = "BH", alpha = 0.05)"
- [methods] Nonparanormal transform rationale: "we apply a nonparanormal transform to relax the normality assumption."
- [methods] Significant ACE path filter (>2 nodes): "## Found 3 significant ACEs with > 2 nodes"
- [readme] COVID-19 benchmark dataset: [evidence span withheld — anchor status: external_doc]
- [abstract] Perturbation extraction capability claim: "Compared to existing methods, SEMtree() is able to capture biologically relevant subnetworks with simple visualization of directed paths, good perturbation extraction, and classifier performance."
