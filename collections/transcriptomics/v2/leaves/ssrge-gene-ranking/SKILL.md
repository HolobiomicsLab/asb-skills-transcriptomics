---
name: ssrge-gene-ranking
description: Use when you have fitted SSrGE on paired SNV (n_cells x n_SNVs binary)
  and gene-expression matrices and need a ranked gene list (with SNV ids) to select
  top-ranked eeSNV genes for subpopulation markers or pathway enrichment.
when_to_use_negative:
- Input is a single unpaired matrix (SNV or expression alone) — SSrGE ranking requires
  both a coupled SNV matrix and a gene-expression matrix from the same cells
- The environment cannot run Python 2.7 with scikit-learn 0.18, which the package
  requires
- The goal is de novo SNV calling from raw reads rather than ranking already-called
  eeSNVs and genes
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: SSrGE
  role: Python package providing fit(), rank_genes(), rank_eeSNVs(), and rank_features_for_a_subgroup()
    to produce eeSNV and gene rankings via LASSO-penalized sparse linear models
  repo: https://github.com/lanagarmire/SSrGE.git
- name: Python 2.7
  role: Runtime environment required by the garmire_SSrGE package (pip2 install -r
    requirements.txt)
- name: Docker (opoirion/ssrge)
  role: Containerized distribution of the SSrGE download, alignment, and SNV calling
    pipelines
  repo: opoirion/ssrge
- name: DAVID
  role: Downstream consumer of top-100 ranked gene lists for KEGG pathway enrichment
    p-values
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:ff039a136bc4d90f4ca1e3b0b7c3eb69506c997522a53177da5c1b379ca55433
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/SSrGE.git
  tools:
  - Cytoscape
  - DAVID
  - Docker (opoirion/ssrge)
  - Gephi
  - Python
  - Python 2.7
  - SSrGE
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# ssrge-gene-ranking

## Summary

Rank eeSNVs and the genes harboring them after fitting SSrGE sparse linear models, so that robust SNV-based genetic markers (e.g., top 100 ranked genes per scRNA-seq cancer dataset) can be identified and carried into downstream enrichment analyses such as DAVID KEGG pathway testing.

## When to use

Use when you have fitted SSrGE on paired SNV (n_cells x n_SNVs binary) and gene-expression matrices and need a ranked gene list (with SNV ids) to select top-ranked eeSNV genes for subpopulation markers or pathway enrichment.

## When NOT to use

- Input is a single unpaired matrix (SNV or expression alone) — SSrGE ranking requires both a coupled SNV matrix and a gene-expression matrix from the same cells
- The environment cannot run Python 2.7 with scikit-learn 0.18, which the package requires
- The goal is de novo SNV calling from raw reads rather than ranking already-called eeSNVs and genes

## Inputs

- Binary SNV matrix (n_cells x n_SNVs), encoded X_c,n = 1 if SNV present in cell
- Gene-expression matrix (n_cells x n_Genes), float values from scRNA-seq
- SNV id list: tuples of (gene id harboring the SNV, user-defined SNV id such as genome position)
- Gene id list ordered to correspond to the gene matrix
- Optional CNV matrix (n_cells x n_genes) passed to fit()
- Optional subgroup cell index list for rank_features_for_a_subgroup()

## Outputs

- Ranked gene list from ssrge.rank_genes()
- Ranked eeSNV feature list from ssrge.rank_eeSNVs()
- Per-subgroup objects with ranked_genes, ranked_eeSNVs, significant_genes, significant_eeSNVs
- Top-100 ranked gene list per dataset for enrichment submission

## How to apply

Instantiate SSrGE with snv_id_list (tuples of gene id harboring the SNV plus a user-defined id such as genome position) and gene_id_list, then fit on the binary SNV matrix (X_c,n = 1 if SNV s present in cell c) and the gene-expression matrix using LASSO penalization, which acts as feature selection to minimize spurious SNVs. Call ssrge.rank_genes() to obtain the gene ranking derived from the accumulative ranking of their linked eeSNVs; alternatively ssrge.rank_eeSNVs() ranks the eeSNVs themselves and rank_features_for_a_subgroup([...]) restricts ranking to a cell subgroup. If a fixed number of features is desired, set nb_ranked_features instead of relying on the alpha regularization parameter. Extract the top 100 highest-ranked genes per dataset and feed them to downstream tools (e.g., DAVID for KEGG pathway enrichment, Gephi/Cytoscape for bipartite pathway-gene graphs). SSrGE requires Python 2.7 with Numpy, Scipy, scikit-learn 0.18, and tabulate, and can be run locally or via the opoirion/ssrge docker container.

## Related tools

- **SSrGE** (Python package providing fit(), rank_genes(), rank_eeSNVs(), and rank_features_for_a_subgroup() to produce eeSNV and gene rankings via LASSO-penalized sparse linear models) — https://github.com/lanagarmire/SSrGE.git
- **Python 2.7** (Runtime environment required by the garmire_SSrGE package (pip2 install -r requirements.txt))
- **Docker (opoirion/ssrge)** (Containerized distribution of the SSrGE download, alignment, and SNV calling pipelines) — opoirion/ssrge
- **DAVID** (Downstream consumer of top-100 ranked gene lists for KEGG pathway enrichment p-values)

## Examples

```
from garmire_SSrGE.ssrge import SSrGE; ssrge = SSrGE(snv_id_list=snv_id_list, gene_id_list=gene_id_list, alpha=0.01); ssrge.fit(X, Y); print ssrge.rank_genes()
```

## Evaluation signals

- rank_genes() output length and ordering match the gene_id_list dimensionality of the fitted Y matrix
- Top-100 ranked genes per dataset reproduce the article's KEGG enrichment result: antigen processing and presentation pathway with sum -log10(p-value) = 15.80 across the five scRNA-seq cancer datasets (GSE69405, GSE81383, GSE65253, Kim, Ting/Patel)
- Ranked genes are dominated by genes harboring high-scoring eeSNVs (e.g., B2M, HLA genes) consistent with known cancer-relevant mutational hotspots
- Setting nb_ranked_features=K yields exactly K retained features in the transformed matrix X_r (verified via X_r.shape)
- python2 test/test_ssrge.py -v passes, confirming the installation is functional

## Limitations

- LASSO may arbitrarily select one of several highly correlated SNV features corresponding to the same gene expression pattern
- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads
- nb_ranked_features must be lower than the number of eeSNVs obtained with the specified alpha
- Python 2.7 (>=2.7) with scikit-learn 0.18 is mandatory; the package is not drop-in for Python 3

## Evidence

- [discussion] SSrGE uses an accumulative ranking approach to select eeSNVs linked to the expression of a particular gene.: "SSrGE uses an accumulative ranking approach to select eeSNVs linked to the expression of a particular gene."
- [discussion] LASSO penalization as feature selection: "Secondly, LASSO penalization works as a feature selection method and minimizes the spurious SNVs (false positive) from the filtered set of eeSNVs."
- [readme] rank_genes usage from README: "a SNV id should be a tuple containing the gene id harboring the given SNV and a user defined SNV id (genome position for example)."
- [methods] nb_ranked_features semantics: "Instead of relying on the regularization parameter (alpha), to select the number of eeSNVs, the `nb_ranked_features` argument can be specified"
- [results] Source-card finding on pathway enrichment: [evidence span withheld — anchor status: not_found]
- [readme] Python 2.7 requirement from README: "pip2 install -r requirements.txt --user # python 2.7.X must be used"
