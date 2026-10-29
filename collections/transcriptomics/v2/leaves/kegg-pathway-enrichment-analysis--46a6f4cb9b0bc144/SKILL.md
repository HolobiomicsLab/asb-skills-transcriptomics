---
name: kegg-pathway-enrichment-analysis
description: Use when you have ranked gene lists (e.g., SSrGE rank_genes() output)
  from multiple scRNA-seq datasets and need to test which KEGG pathways are over-represented
  among top-ranked genes and combine evidence across datasets.
when_to_use_negative:
- Input genes are an unranked or unfiltered gene list rather than top-ranked SSrGE
  eeSNV genes — enrichment would not reflect genotype-phenotype linkage.
- You need pathway analysis of gene-expression matrices directly rather than eeSNV-harboring
  genes; SSrGE ranking and eeSNV extraction must precede this skill.
- The datasets are non-cancer scRNA-seq datasets (e.g., GSE79457, GSE81903, GSE71485,
  GSE80232) outside the five cancer datasets this enrichment was designed for.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3170
tools:
- name: SSrGE
  role: Produces the ranked gene lists (rank_genes()) from fitted SNV and gene-expression
    matrices whose top 100 genes are tested for KEGG enrichment
  repo: https://github.com/lanagarmire/SSrGE.git
- name: DAVID
  role: Annotation tool to which each top-100 gene list is submitted to obtain KEGG
    pathway enrichment p-values
- name: Gephi
  role: Graph analysis software used to explore the bipartite pathway-gene relationships
    with force-directed drawing
- name: Cytoscape
  role: Alternative graph analysis software for interactive exploration of the bipartite
    pathway-gene network
- name: Python 2.7
  role: Runtime environment for the garmire_SSrGE package used to rank genes prior
    to DAVID submission
  repo: https://github.com/lanagarmire/SSrGE.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:3fa98686d122fcc1f680adee5267a6b0c36195e7a357c7a6c6c1de99bc4619ed
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# kegg-pathway-enrichment-analysis

## Summary

Submit top-100 SSrGE-ranked eeSNV-harboring gene lists from each of five scRNA-seq cancer datasets to the DAVID annotation tool to identify KEGG pathways enriched among the highest-ranked genes, aggregating enrichment evidence by summing -log10(p-value) across datasets.

## When to use

Use when you have ranked gene lists (e.g., SSrGE rank_genes() output) from multiple scRNA-seq datasets and need to test which KEGG pathways are over-represented among top-ranked genes and combine evidence across datasets.

## When NOT to use

- Input genes are an unranked or unfiltered gene list rather than top-ranked SSrGE eeSNV genes — enrichment would not reflect genotype-phenotype linkage.
- You need pathway analysis of gene-expression matrices directly rather than eeSNV-harboring genes; SSrGE ranking and eeSNV extraction must precede this skill.
- The datasets are non-cancer scRNA-seq datasets (e.g., GSE79457, GSE81903, GSE71485, GSE80232) outside the five cancer datasets this enrichment was designed for.

## Inputs

- SSrGE-ranked gene list per dataset (rank_genes() output of the garmire_SSrGE package)
- Top 100 highest-ranked eeSNV genes per scRNA-seq cancer dataset (five datasets)
- KEGG pathway annotation retrieved from DAVID (per-pathway enrichment p-values)

## Outputs

- Per-dataset KEGG pathway enrichment p-value table from DAVID
- Aggregated pathway scores (sum of -log10(p-value) across datasets)
- Bipartite pathway-gene network graph rendered with force-directed layout (Gephi/Cytoscape)

## How to apply

Fit SSrGE on each dataset's SNV matrix (n_cells x n_SNVs, binary) and gene-expression matrix, then call the rank_genes() function of the garmire_SSrGE package (Python 2.7) to obtain ranked gene lists. Extract the top 100 highest-ranked genes per dataset and submit each list independently to the DAVID annotation tool to retrieve KEGG pathway enrichment p-values. For each pathway, aggregate evidence across the five cancer datasets (GSE69405, GSE81383, GSE65253, Ting, Patel, Kim-type scRNA-seq cancer data) by summing -log10(p-value), so pathways recurrently enriched rank highest. Build a bipartite pathway-gene graph where edges connect genes annotated to a pathway, and render it with force-directed graph drawing (spring-like attractive forces, electrical repulsions) in Gephi or Cytoscape. Validation compares the reproduced network against Fig. 5, where the antigen processing and presentation pathway is expected to have a sum -log10(p-value) of 15.80.

## Related tools

- **SSrGE** (Produces the ranked gene lists (rank_genes()) from fitted SNV and gene-expression matrices whose top 100 genes are tested for KEGG enrichment) — https://github.com/lanagarmire/SSrGE.git
- **DAVID** (Annotation tool to which each top-100 gene list is submitted to obtain KEGG pathway enrichment p-values)
- **Gephi** (Graph analysis software used to explore the bipartite pathway-gene relationships with force-directed drawing)
- **Cytoscape** (Alternative graph analysis software for interactive exploration of the bipartite pathway-gene network)
- **Python 2.7** (Runtime environment for the garmire_SSrGE package used to rank genes prior to DAVID submission) — https://github.com/lanagarmire/SSrGE.git

## Examples

```
python2 -c "from garmire_SSrGE.ssrge import SSrGE; ssrge=SSrGE(); ssrge.fit(X,Y); print ssrge.rank_genes()"
```

## Evaluation signals

- The reproduced pathway-gene network matches Fig. 5 of the paper.
- The antigen processing and presentation pathway has a sum -log10(p-value) of 15.80 across the five datasets.
- Each submitted gene list contains exactly the top 100 SSrGE-ranked genes per dataset.
- Enrichment evidence is aggregated per pathway as a sum of -log10(p-value), not a mean or maximum.
- rank_genes() output is reproducible when SSrGE is re-fit on the same SNV and gene-expression matrices with identical parameters.

## Limitations

- LASSO penalization may arbitrarily select one of several highly correlated SNV features corresponding to gene expression, affecting which genes are ranked and submitted.
- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads, so some enriched pathways may be underrepresented.
- The claim that eeSNVs of HLA genes serve as cellular state fingerprints is speculative and not experimentally verified; the antigen processing and presentation enrichment reflects this HLA-centric signal.

## Evidence

- [results] The antigen processing and presentation pathway is the most enriched pathway among the top-ranked eeSNV genes, with a sum -log10(p-value) of 15.80.: [evidence span withheld — anchor status: not_found]
- [discussion] Graph analysis software such as Gephi36 or Cytoscape54 can be utilized to explore the bipartite relationships in an interactive manner.: "Graph analysis software such as Gephi36 or Cytoscape54 can be utilized to explore the bipartite relationships in an interactive manner."
- [methods] print ssrge.rank_genes(): "print ssrge.rank_genes()"
- [discussion] One can also easily rank the eeSNVs and the genes harboring them, for the purpose of identifying robust eeSNVs as genetic markers for a variety of cancers.: "One can also easily rank the eeSNVs and the genes harboring them, for the purpose of identifying robust eeSNVs as genetic markers for a variety of cancers."
- [readme] git clone https://github.com/lanagarmire/SSrGE.git: "git clone https://github.com/lanagarmire/SSrGE.git"
