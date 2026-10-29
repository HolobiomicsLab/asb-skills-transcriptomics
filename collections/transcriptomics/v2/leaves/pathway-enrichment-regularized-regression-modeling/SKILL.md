---
name: pathway-enrichment-regularized-regression-modeling
description: Use when a pathway enrichment analysis must account for redundancy/dependencies
  between gene sets (e.g. large GO/KEGG/Reactome databases), or when benchmarking
  enrichment robustness to noise on synthetic pathway data.
when_to_use_negative:
- When genesets are small and largely non-redundant so inter-pathway dependencies
  add little regularization signal
- When the analysis is plain over-representation analysis without per-gene p-values
  — pareg regresses differential expression p-values and requires them as input
- When no pathway similarity structure (term_network) is available and only the no-penalty
  pareg_noterms baseline is needed
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0203
tools:
- name: pareg
  role: 'Core method: regularized GLM estimating pathway enrichment scores with LASSO
    and network-fusion regularization based on pathway similarity; provides compute_term_similarities,
    similarity_sample, and as_enrichplot_object'
  repo: https://github.com/cbg-ethz/pareg
- name: R
  role: Implementation environment; pareg is an R/Bioconductor package installed via
    BiocManager
  repo: https://bioconductor.org/packages/release/bioc/html/pareg.html
- name: Snakemake
  role: Reproducible workflows for the benchmark sweeps included in the pareg GitHub
    repository
  repo: https://github.com/cbg-ethz/pareg
- name: enrichplot
  role: Converts pareg fits to enrichplot objects for downstream pathway network visualization
  repo: https://github.com/YuLab-SMU/enrichplot
- name: tidyverse
  role: Data wrangling and plotting of similarity matrices and PR-AUC results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad522
    title: Coherent pathway enrichment estimation by modeling inter-pathway dependencies
      using regularized regression
  source_content_hash: sha256:1d664c5f45fbd20b1996d5aecb98a3b8800402b9875d0b66e3814985e9ed9c3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/pareg
  tools:
  - R
  - Snakemake
  - enrichplot
  - pareg
  - tidyverse
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# pathway-enrichment-regularized-regression-modeling

## Summary

Apply pareg, a regularized generalized linear model that incorporates inter-pathway dependencies (e.g. Jaccard similarity) via LASSO and network fusion penalties, to estimate coherent pathway enrichment scores; validated on synthetic benchmarks by sweeping network_param (η) and comparing PR-AUC across regularization variants.

## When to use

Use when a pathway enrichment analysis must account for redundancy/dependencies between gene sets (e.g. large GO/KEGG/Reactome databases), or when benchmarking enrichment robustness to noise on synthetic pathway data.

## When NOT to use

- When genesets are small and largely non-redundant so inter-pathway dependencies add little regularization signal
- When the analysis is plain over-representation analysis without per-gene p-values — pareg regresses differential expression p-values and requires them as input
- When no pathway similarity structure (term_network) is available and only the no-penalty pareg_noterms baseline is needed

## Inputs

- Differential expression p-values per gene (Beta-distributed for de_genes, Uniform for background)
- Synthetic pathway database as term-to-gene mapping (df_terms)
- Pathway similarity matrix (e.g. Jaccard) supplied as term_network
- network_param (η) sweep values, e.g. {0.25, 0.75}

## Outputs

- Pathway enrichment scores from the fitted pareg regularized GLM
- PR-AUC table per regularization variant and η
- Pathway similarity heatmap and pathway network visualization (via as_enrichplot_object / ggraph)

## How to apply

Generate a synthetic pathway database with pronounced clustering (2 pathway groups of 10 pathways each, 15 group genes plus 50 background genes per group, each pathway containing 10 group and 10 background genes), then compute pairwise pathway similarities with pareg's compute_term_similarities using the Jaccard similarity_function and confirm group clustering in a ComplexHeatmap heatmap. Select 5 active pathways as ground-truth positives via similarity_sample(mat_similarities, 5), define de_genes as the union of their genes, and sample p-values from Beta(0.1, 1) for de_genes and a Uniform distribution for all other genes. Fit pareg in four regularization configurations (pareg_noterms, pareg_lasso, pareg_network, pareg_network_lasso), passing term_network = mat_similarities and sweeping network_param (η) over {0.25, 0.75}. Compute precision-recall AUC from the fitted enrichment scores against the selected active terms; the joint network+LASSO variant should rank highest at both noise settings, with LASSO contributing more at low noise and network fusion more at high noise.

## Related tools

- **pareg** (Core method: regularized GLM estimating pathway enrichment scores with LASSO and network-fusion regularization based on pathway similarity; provides compute_term_similarities, similarity_sample, and as_enrichplot_object) — https://github.com/cbg-ethz/pareg
- **R** (Implementation environment; pareg is an R/Bioconductor package installed via BiocManager) — https://bioconductor.org/packages/release/bioc/html/pareg.html
- **Snakemake** (Reproducible workflows for the benchmark sweeps included in the pareg GitHub repository) — https://github.com/cbg-ethz/pareg
- **enrichplot** (Converts pareg fits to enrichplot objects for downstream pathway network visualization) — https://github.com/YuLab-SMU/enrichplot
- **tidyverse** (Data wrangling and plotting of similarity matrices and PR-AUC results)

## Examples

```
BiocManager::install("pareg"); mat_similarities <- compute_term_similarities(df_terms, similarity_function = jaccard); fit <- pareg(df_terms, pvalues, term_network = mat_similarities, network_param = 0.75)
```

## Evaluation signals

- PR-AUC table must rank pareg_network_lasso above the single-penalty and no-penalty variants at both η=0.25 and η=0.75, reproducing the Supplementary Fig S1 pattern
- ComplexHeatmap of the Jaccard similarity matrix must show pronounced clustering of the two synthetic pathway groups
- P-value distribution check: de_genes drawn from Beta(0.1, 1) (concentrated near 0), all other genes from Uniform
- Similarity matrix must be square with one row/column per pathway; active_terms must contain exactly 5 pathways sampled via similarity_sample

## Limitations

- Robustness claims are demonstrated on synthetic benchmark data and an exploratory TCGA breast cancer analysis; performance depends on the quality of the pathway similarity matrix supplied as term_network
- Pathway databases like GO, KEGG and Reactome are growing in size and redundancy, which complicates the statistical enrichment computation that pareg addresses
- Different similarity measures can differ substantially (observable on GO Biological Process), so choice of similarity_function affects the network regularization

## Evidence

- [abstract] pareg, which is based on a regularized generalized linear model and directly incorporates dependencies between gene sets: "develop a novel gene set enrichment method, called pareg, which is based on a regularized generalized linear model and directly incorporates dependencies between gene sets"
- [methods] Jaccard similarity and modular enrichment classification: "The network regularization term is based on a pathway similarity matrix (e.g., defined by Jaccard similarity) and thus classifies this method as a modular enrichment analysis tool"
- [methods] compute_term_similarities with jaccard: "compute_term_similarities(
  df_terms,
  similarity_function = jaccard
)"
- [methods] Sampling active pathways as true positives: "active_terms <- similarity_sample(mat_similarities, 5)"
- [methods] P-value sampling scheme for synthetic benchmark: "The p-values of genes considered to be differentially expressed are sampled from a Beta distribution centered at $0$. The p-values for all other genes are drawn from a Uniform distribution."
- [readme] Installation from Bioconductor: "Install the latest stable version from Bioconductor:
```r
BiocManager::install("pareg")
```"
- [abstract] Robustness to noise claim: "We show that pareg is more robust to noise than competing methods."
