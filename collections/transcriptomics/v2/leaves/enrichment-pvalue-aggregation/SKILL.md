---
name: enrichment-pvalue-aggregation
description: Use when you have top-N gene lists (e.g., top 100 SSrGE-ranked eeSNV
  genes) from 5+ datasets and need a single cross-dataset pathway enrichment ranking,
  e.g., to reproduce Fig. 5-style KEGG pathway-gene graphs.
when_to_use_negative:
- You only have a single gene list and one enrichment run — a plain DAVID p-value
  ranking suffices without cross-dataset -log10(p) summation
- The genes are not derived from SSrGE eeSNV ranking or any ranked feature list (the
  skill assumes per-dataset top-N ranked genes as input)
- The annotation resource of interest is not KEGG via DAVID; other databases would
  need different p-value aggregation provenance
edam_operation: http://edamontology.org/operation_2436
tools:
- name: SSrGE
  role: Produces the top-100 ranked eeSNV gene lists per dataset via rank_genes();
    its garmire_SSrGE package (Python 2.7, scikit-learn 0.18, Numpy, Scipy, tabulate)
    supplies the input genes for enrichment
  repo: https://github.com/lanagarmire/SSrGE.git
- name: DAVID
  role: Annotation tool that receives each top-100 gene list and returns KEGG pathway
    enrichment p-values
- name: Gephi
  role: Renders the bipartite pathway-gene graph with force-directed graph drawing
    (spring-like attractive forces and electrical repulsions) for interactive exploration
- name: Cytoscape
  role: Alternative graph-analysis software to explore the bipartite pathway-gene
    relationships interactively
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:1ae604c3372670c5863a00bc5e101a02ae2b8253c4478c23a36577f0a459ddf3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/SSrGE.git
  tools:
  - Cytoscape
  - DAVID
  - Gephi
  - Python
  - SSrGE
  edam_operation: http://edamontology.org/operation_2436
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# enrichment-pvalue-aggregation

## Summary

Aggregate pathway enrichment evidence across multiple single-cell cancer datasets by submitting top-ranked eeSNV gene lists to DAVID, transforming KEGG p-values via -log10, and summing them per pathway to build a ranked bipartite pathway-gene network.

## When to use

Use when you have top-N gene lists (e.g., top 100 SSrGE-ranked eeSNV genes) from 5+ datasets and need a single cross-dataset pathway enrichment ranking, e.g., to reproduce Fig. 5-style KEGG pathway-gene graphs.

## When NOT to use

- You only have a single gene list and one enrichment run — a plain DAVID p-value ranking suffices without cross-dataset -log10(p) summation
- The genes are not derived from SSrGE eeSNV ranking or any ranked feature list (the skill assumes per-dataset top-N ranked genes as input)
- The annotation resource of interest is not KEGG via DAVID; other databases would need different p-value aggregation provenance

## Inputs

- Top-100 SSrGE-ranked eeSNV gene lists (one per scRNA-seq cancer dataset, e.g., GSE69405, GSE81383, GSE65253)
- KEGG pathway enrichment p-values returned by DAVID for each submitted gene list

## Outputs

- Per-pathway aggregate score = sum of -log10(p-value) across the five datasets
- Ranked table of enriched KEGG pathways (e.g., antigen processing and presentation at 15.80)
- Bipartite pathway-gene graph (edge: gene annotated to pathway) rendered via force-directed layout in Gephi or Cytoscape

## How to apply

First fit SSrGE (LASSO-based sparse modeling of the n_cells x n_SNVs binary matrix against the n_cells x n_genes expression matrix) on each of the five scRNA-seq cancer datasets and extract the top 100 highest-ranked genes per dataset using rank_genes() from the garmire_SSrGE package under Python 2.7. Submit each top-100 gene list separately to the DAVID annotation tool and retrieve KEGG pathway enrichment p-values. For each pathway, transform each dataset's p-value with -log10 and sum these across all datasets in which the pathway appears, producing a single aggregate score (e.g., the antigen processing and presentation pathway reached a sum -log10(p-value) of 15.80). Then build a bipartite pathway-gene graph where an edge connects a gene to a pathway it is annotated to, and render it with force-directed graph drawing (spring-like attractive forces, electrical repulsions) in Gephi or Cytoscape for interactive exploration.

## Related tools

- **SSrGE** (Produces the top-100 ranked eeSNV gene lists per dataset via rank_genes(); its garmire_SSrGE package (Python 2.7, scikit-learn 0.18, Numpy, Scipy, tabulate) supplies the input genes for enrichment) — https://github.com/lanagarmire/SSrGE.git
- **DAVID** (Annotation tool that receives each top-100 gene list and returns KEGG pathway enrichment p-values)
- **Gephi** (Renders the bipartite pathway-gene graph with force-directed graph drawing (spring-like attractive forces and electrical repulsions) for interactive exploration)
- **Cytoscape** (Alternative graph-analysis software to explore the bipartite pathway-gene relationships interactively)

## Examples

```
from garmire_SSrGE.ssrge import SSrGE; ssrge = SSrGE(snv_id_list=snv_id_list, gene_id_list=gene_id_list); ssrge.fit(X, Y); print ssrge.rank_genes()
```

## Evaluation signals

- The reproduced pathway-gene network matches Fig. 5, with the antigen processing and presentation pathway sum -log10(p-value) exactly equal to 15.80
- Each dataset contributes KEGG p-values only for the top-100 (not larger) eeSNV gene lists
- Every -log10(p-value) is non-negative and finite before summation (p-values in (0, 1])
- The bipartite graph contains an edge only where DAVID annotated the gene to that KEGG pathway
- Aggregated scores are reproducible across DAVID re-submissions of identical gene lists

## Limitations

- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads, so top-ranked eeSNV gene lists feeding enrichment may be incomplete
- LASSO may arbitrarily select one of several highly correlated SNV features corresponding to gene expression, affecting which genes reach the top-100
- DAVID/KEGG coverage depends on gene identifier mapping; pathways absent from a dataset contribute nothing to the aggregate sum

## Evidence

- [results] The antigen processing and presentation pathway is the most enriched pathway among the top-ranked eeSNV genes, with a sum -log10(p-value) of 15.80.: [evidence span withheld — anchor status: not_found]
- [discussion] Graph analysis software such as Gephi or Cytoscape can be utilized to explore the bipartite relationships in an interactive manner.: "Graph analysis software such as Gephi36 or Cytoscape54 can be utilized to explore the bipartite relationships in an interactive manner."
- [discussion] One can also easily rank the eeSNVs and the genes harboring them, for the purpose of identifying robust eeSNVs as genetic markers.: "One can also easily rank the eeSNVs and the genes harboring them, for the purpose of identifying robust eeSNVs as genetic markers for a variety of cancers."
- [discussion] We have used force-directed graph drawing algorithms involve spring-like attractive forces and electrical repulsions between nodes that are connected by edges.: "We have used force-directed graph drawing algorithms involve spring-like attractive forces and electrical repulsions between nodes that are connected by edges."
- [readme] print ssrge.rank_genes(): "print ssrge.rank_genes()"
