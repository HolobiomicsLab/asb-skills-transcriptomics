---
name: jaccard-similarity-matrix-construction
description: Use when a pathway enrichment analysis needs inter-pathway dependencies
  quantified from gene set overlaps — e.g. before fitting pareg with network regularization,
  or for rendering a pathway network view — and the input is a term-to-gene mapping
  (df_terms) such as GO BP terms of size 50-500.
when_to_use_negative:
- The analysis does not model inter-pathway dependencies — e.g. running plain over-representation
  analysis or pareg's no-regularization variant (pareg_noterms) requires no term similarity
  matrix.
- The pathway database is so small or non-redundant that pathway overlap carries no
  useful signal; Jaccard similarity is only meaningful when pathways share genes.
- You need semantic (ontology-based) pathway similarity rather than gene-set-overlap
  similarity; the article compares similarity measures on GO Biological Process because
  they can differ substantially.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0085
tools:
- name: pareg
  role: Provides the compute_term_similarities helper, the jaccard similarity_function,
    pre-computed data(pathway_similarities) matrices, and consumes the matrix as term_network
    in its regularized GLM
  repo: https://github.com/cbg-ethz/pareg
- name: R
  role: Implementation environment; pareg is an R/Bioconductor package
  repo: https://bioconductor.org/packages/release/bioc/html/pareg.html
- name: Heatmap (ComplexHeatmap)
  role: Visual QC of the computed similarity matrix to confirm pathway group clustering
- name: ggpairs
  role: Pairwise comparison of different similarity measures on GO Biological Process
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad522
    title: Coherent pathway enrichment estimation by modeling inter-pathway dependencies
      using regularized regression
  source_content_hash: sha256:3ba8119550541c0ee8cd86e52154a8ebf3ca84e70d09b9cb4d8591e80ef61728
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/pareg
  tools:
  - Heatmap (ComplexHeatmap)
  - R
  - Snakemake
  - enrichplot
  - ggpairs
  - ggraph
  - pareg
  - tidyverse
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# jaccard-similarity-matrix-construction

## Summary

Construct a pairwise pathway similarity matrix from a pathway database (gene set annotations) using pareg's compute_term_similarities helper with the Jaccard similarity function; this matrix supplies the network regularization (term_network) input to pareg's regularized GLM and enables pathway-network visualization.

## When to use

Use when a pathway enrichment analysis needs inter-pathway dependencies quantified from gene set overlaps — e.g. before fitting pareg with network regularization, or for rendering a pathway network view — and the input is a term-to-gene mapping (df_terms) such as GO BP terms of size 50-500.

## When NOT to use

- The analysis does not model inter-pathway dependencies — e.g. running plain over-representation analysis or pareg's no-regularization variant (pareg_noterms) requires no term similarity matrix.
- The pathway database is so small or non-redundant that pathway overlap carries no useful signal; Jaccard similarity is only meaningful when pathways share genes.
- You need semantic (ontology-based) pathway similarity rather than gene-set-overlap similarity; the article compares similarity measures on GO Biological Process because they can differ substantially.

## Inputs

- Pathway/term-to-gene dataframe (df_terms with term and gene columns), e.g. GO biological process terms filtered to size 50-500
- Optional: pre-computed similarity matrices from data(pathway_similarities, package = "pareg")
- TCGA-BRCA gene-level p-value table (gene, pvalue) when the matrix feeds a pareg fit

## Outputs

- Symmetric pairwise pathway similarity matrix (pathways x pathways, Jaccard index values in [0,1])
- Similarity heatmap (ComplexHeatmap) for QC of pathway clustering
- term_network input object consumed by pareg's regularized GLM fit

## How to apply

Load the pathway database as a term-to-gene dataframe (e.g. GO biological process terms filtered to size 50-500) and call pareg's compute_term_similarities helper with similarity_function = jaccard to obtain a symmetric, square similarity matrix. Alternatively, load pre-computed matrices shipped with pareg via data(pathway_similarities, package = "pareg"). Pass this matrix as term_network to pareg's fit (together with network_param, e.g. eta = 1 for the TCGA-BRCA analysis or eta swept over {0.25, 0.75} in the ablation), combining it with LASSO regularization. Sanity-check the matrix before use — e.g. with a ComplexHeatmap heatmap confirming pronounced group clustering on synthetic databases, or ggpairs plots comparing similarity measures on GO Biological Process. The same matrix also supports similarity-based pathway sampling (similarity_sample) and network visualization (plot(fit, min_similarity = 0.1)).

## Related tools

- **pareg** (Provides the compute_term_similarities helper, the jaccard similarity_function, pre-computed data(pathway_similarities) matrices, and consumes the matrix as term_network in its regularized GLM) — https://github.com/cbg-ethz/pareg
- **R** (Implementation environment; pareg is an R/Bioconductor package) — https://bioconductor.org/packages/release/bioc/html/pareg.html
- **Heatmap (ComplexHeatmap)** (Visual QC of the computed similarity matrix to confirm pathway group clustering)
- **ggpairs** (Pairwise comparison of different similarity measures on GO Biological Process)

## Examples

```
mat_similarities <- compute_term_similarities(df_terms, similarity_function = jaccard)
```

## Evaluation signals

- The output is a square, symmetric matrix with one row/column per pathway term and off-diagonal entries in [0, 1] (Jaccard index).
- On the synthetic benchmark (2 pathway groups of 10 pathways each), a ComplexHeatmap of the matrix must show pronounced clustering of pathways into the two groups.
- When used as term_network with network_param set (e.g. eta = 1 for TCGA-BRCA), the pareg fit's top-25 ranked terms must include axoneme assembly, retrograde vesicle-mediated transport, and synaptic vesicle recycling.
- Network visualization via plot(fit, min_similarity = 0.1) on the top-50 non-isolated terms must show clustered pathway groups consistent with the reported TCGA-BRCA findings.
- In the ablation benchmark, using the Jaccard matrix as term_network must yield pareg_network_lasso with the highest PR-AUC at eta = 0.25 and eta = 0.75 (Supplementary Fig S1 pattern).

## Limitations

- Different pathway similarity measures can disagree substantially — on GO Biological Process the article observes 'how much pathway similarity measures can differ from each other', so Jaccard choice affects the network regularization.
- Some terms are 'isolated' (no sufficiently similar neighbors) and are excluded from the network view at min_similarity = 0.1.
- Pre-computed pathway_similarities are only provided for selected pathway databases; other databases require computing similarities yourself.

## Evidence

- [methods] compute_term_similarities call with jaccard: "compute_term_similarities(
  df_terms,
  similarity_function = jaccard
)"
- [intro] pre-computed similarity matrices: "pareg provides various pre-computed similarity measures (`r names(pathway_similarities[[1]])`) for selected pathway databases (`r names(pathway_similarities)`) in matrix form."
- [methods] network regularization defines pareg as modular enrichment: "The network regularization term is based on a pathway similarity matrix (e.g., defined by Jaccard similarity) and thus classifies this method as a modular enrichment analysis tool"
- [readme] README installation of the R package: "Install the latest stable version from Bioconductor:
```r
BiocManager::install("pareg")
```"
- [intro] Jaccard counts shared genes: "For example, when interpreting pathways as gene sets, one could count how many genes are shared between two sets."
