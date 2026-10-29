---
name: functional-annotation-submission
description: Use when you have top-ranked genes (e.g., the top 100 SSrGE-ranked eeSNV
  genes per dataset) and need to identify enriched KEGG pathways and link genes to
  pathways in a bipartite network.
when_to_use_negative:
- You only have raw .sra/.fastq reads and have not yet fit SSrGE to obtain ranked
  eeSNV genes — run the SNV calling and SSrGE ranking pipeline first
- The SNV/gene-expression matrices are already a feature table of selected eeSNVs
  and you only need gene ranking, not pathway annotation
tools:
- name: SSrGE
  role: Fit sparse linear models on SNV and gene-expression matrices and produce the
    ranked gene list via rank_genes()
  repo: https://github.com/lanagarmire/SSrGE.git
- name: DAVID
  role: External annotation tool receiving the top-100 gene lists and returning KEGG
    pathway enrichment p-values
- name: Gephi
  role: Render the bipartite pathway-gene graph with force-directed graph drawing
- name: Cytoscape
  role: Interactive exploration of the bipartite pathway-gene relationships
- name: Python 2.7
  role: Runtime environment for the garmire_SSrGE package used to generate ranked
    gene lists
  repo: https://github.com/lanagarmire/SSrGE.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:c7049552862613ea9b55abbbd5492cc42a17b3efc91dbabaa859bedec3a285da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/SSrGE.git
  tools:
  - Cytoscape
  - DAVID
  - Gephi
  - Python
  - Python 2.7
  - SSrGE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# functional-annotation-submission

## Summary

Submit SSrGE-ranked eeSNV gene lists to DAVID for KEGG pathway enrichment, then aggregate enrichment evidence across datasets and visualize pathway-gene bipartite relationships with force-directed graphs in Gephi or Cytoscape.

## When to use

Use when you have top-ranked genes (e.g., the top 100 SSrGE-ranked eeSNV genes per dataset) and need to identify enriched KEGG pathways and link genes to pathways in a bipartite network.

## When NOT to use

- You only have raw .sra/.fastq reads and have not yet fit SSrGE to obtain ranked eeSNV genes — run the SNV calling and SSrGE ranking pipeline first
- The SNV/gene-expression matrices are already a feature table of selected eeSNVs and you only need gene ranking, not pathway annotation

## Inputs

- SNV matrix (n_cells x n_SNVs, binary) per scRNA-seq cancer dataset
- Gene-expression matrix (n_cells x n_Genes) per dataset
- SSrVE/SSrGE ranked gene list (rank_genes() output)
- Top-100 highest-ranked eeSNV gene lists per dataset
- DAVID KEGG pathway enrichment p-values

## Outputs

- KEGG pathway enrichment p-values per gene list per dataset
- Aggregated pathway scores (sum of -log10(p-value) across datasets)
- Ranked eeSNV gene lists (top 100 per dataset)
- Bipartite pathway-gene network rendered with force-directed graph drawing

## How to apply

Fit SSrGE on each dataset's SNV (n_cells x n_SNVs binary) and gene-expression matrices under Python 2.7 (with Numpy, Scipy, scikit-learn 0.18, tabulate), then call rank_genes() to obtain the ranked gene list. Extract the top 100 highest-ranked genes per dataset and submit each list to the DAVID annotation tool, retrieving KEGG pathway enrichment p-values. Per pathway, aggregate evidence by summing -log10(p-value) across the five cancer datasets (e.g., the antigen processing and presentation pathway reaches a sum -log10(p-value) of 15.80). Build a bipartite pathway-gene graph with an edge per gene annotated to a pathway, and render it using force-directed graph drawing (spring-like attractive forces and electrical repulsions) in Gephi or Cytoscape.

## Related tools

- **SSrGE** (Fit sparse linear models on SNV and gene-expression matrices and produce the ranked gene list via rank_genes()) — https://github.com/lanagarmire/SSrGE.git
- **DAVID** (External annotation tool receiving the top-100 gene lists and returning KEGG pathway enrichment p-values)
- **Gephi** (Render the bipartite pathway-gene graph with force-directed graph drawing)
- **Cytoscape** (Interactive exploration of the bipartite pathway-gene relationships)
- **Python 2.7** (Runtime environment for the garmire_SSrGE package used to generate ranked gene lists) — https://github.com/lanagarmire/SSrGE.git

## Examples

```
from garmire_SSrGE.ssrge import SSrGE; ssrge = SSrGE(snv_id_list=snv_id_list, gene_id_list=gene_id_list); ssrge.fit(X, Y); print ssrge.rank_genes()
```

## Evaluation signals

- The reproduced network matches Fig. 5, with the antigen processing and presentation pathway sum -log10(p-value) equal to 15.80
- Each dataset contributes exactly the top 100 highest-ranked genes from the SSrGE rank_genes() output
- Graph edges in the bipartite network correspond exactly to gene-annotated-to-pathway relationships from DAVID KEGG results
- The visualization uses force-directed graph drawing with spring-like attractive forces and electrical repulsions between connected nodes

## Limitations

- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads, so ranked gene lists may not capture all relevant pathways
- In case of correlated features, LASSO may arbitrarily select one of several highly correlated SNV features corresponding to gene expression, affecting which genes are ranked
- The package requires Python 2.7 with scikit-learn 0.18, which constrains reproduction on modern environments

## Evidence

- [other] The antigen processing and presentation pathway is the most enriched pathway among the top-ranked eeSNV genes, with a sum -log10(p-value) of 15.80.: [evidence span withheld — anchor status: not_found]
- [discussion] Graph analysis software such as Gephi or Cytoscape can be utilized to explore the bipartite relationships in an interactive manner.: "Graph analysis software such as Gephi36 or Cytoscape54 can be utilized to explore the bipartite relationships in an interactive manner."
- [discussion] We have used force-directed graph drawing algorithms involve spring-like attractive forces and electrical repulsions between nodes that are connected by edges.: "We have used force-directed graph drawing algorithms involve spring-like attractive forces and electrical repulsions between nodes that are connected by edges."
- [readme] pip2 install -r requirements.txt --user # python 2.7.X must be used: "pip2 install -r requirements.txt --user # python 2.7.X must be used"
- [methods] ranked_feature = ssrge.rank_eeSNVs(): "ranked_feature = ssrge.rank_eeSNVs()"
