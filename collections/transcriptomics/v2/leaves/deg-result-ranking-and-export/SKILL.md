---
name: deg-result-ranking-and-export
description: Use when a haystack run has completed and you need to extract, rank,
  and export its DEG results ordered by log p-values / D_KL divergence for downstream
  inspection or verification of the top-ranked gene.
when_to_use_negative:
- Genes detected in very few cells have not yet been filtered (genes with less than
  10 cells with non-zero counts should be removed first)
- The expression data is continuous normalized read counts rather than binary detection
  data — the original method requires binary detection data (a gene is either detected
  or not in each cell)
- You need a newer, generalized version of the method — the default CRAN/GitHub master
  branch is version 1.0, not the version described in this paper (use the 'binary'
  branch instead)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: singleCellHaystack
  role: Provides haystack/haystack_highD for the DEG prediction run and show_result_haystack
    for retrieving results ranked by log p-values / D_KL divergence
  repo: github.com/alexisvdb/singleCellHaystack
- name: R
  role: Implementation language; ranked table exported with write.csv to bone_marrow_deg_table.csv
- name: Seurat
  role: Used in the article's methods for single-cell preprocessing of the input data
  repo: github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:910f07962072c5dc58d1725ecfbd152807264352cda7cc7d215b38a77b66a49b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# deg-result-ranking-and-export

## Summary

Retrieves and ranks the results of a singleCellHaystack differential expression analysis by log p-values / Kullback-Leibler divergence (D_KL) and exports the ranked gene table (e.g. bone_marrow_deg_table.csv), so the most significant DEG can be verified.

## When to use

Use when a haystack run has completed and you need to extract, rank, and export its DEG results ordered by log p-values / D_KL divergence for downstream inspection or verification of the top-ranked gene.

## When NOT to use

- Genes detected in very few cells have not yet been filtered (genes with less than 10 cells with non-zero counts should be removed first)
- The expression data is continuous normalized read counts rather than binary detection data — the original method requires binary detection data (a gene is either detected or not in each cell)
- You need a newer, generalized version of the method — the default CRAN/GitHub master branch is version 1.0, not the version described in this paper (use the 'binary' branch instead)

## Inputs

- haystack result object returned by the haystack / haystack_highD function (default parameters)
- gene detection matrix (genes x cells) previously filtered to remove genes detected in fewer than 10 cells
- multidimensional input coordinates used for the haystack run (e.g. 50 PCs, t-SNE, UMAP)

## Outputs

- Ranked DEG table ordered by log p-values / D_KL divergence (e.g. bone_marrow_deg_table.csv)
- Top-ranked gene identifier (expected: Ramp1 for the Tabula Muris bone marrow run)

## How to apply

After calling haystack (haystack_highD, default parameters) on a multidimensional input space — e.g. the first 50 PCs of the Tabula Muris bone marrow FACS-sorted matrix (5,250 cells, 13,756 genes) — retrieve the ranked results with show_result_haystack, ordered by log p-values / D_KL divergence. Genes were filtered beforehand with less than 10 cells with non-zero counts to eliminate very lowly expressed genes, which the method treats as binary detection data. Write the ranked table with write.csv (e.g. to bone_marrow_deg_table.csv) and verify the top-ranked gene against expected results (Ramp1 in the reference run, detected only in a subset of cells). Note the paper's version of the method is the 'binary' branch; the current CRAN/GitHub default is version 1.0, so pin @binary to reproduce the described behavior.

## Related tools

- **singleCellHaystack** (Provides haystack/haystack_highD for the DEG prediction run and show_result_haystack for retrieving results ranked by log p-values / D_KL divergence) — github.com/alexisvdb/singleCellHaystack
- **R** (Implementation language; ranked table exported with write.csv to bone_marrow_deg_table.csv)
- **Seurat** (Used in the article's methods for single-cell preprocessing of the input data) — github.com/satijalab/seurat

## Examples

```
res <- haystack(pcs50, dat.expression); ranked <- show_result_haystack(res); write.csv(ranked, "bone_marrow_deg_table.csv")
```

## Evaluation signals

- Top-ranked gene in the exported table matches the reference finding (Ramp1 for the Tabula Muris bone marrow FACS-sorted dataset)
- Exported CSV rows are ordered by log p-values / D_KL divergence from show_result_haystack
- All genes in the table pass the filter of detection in at least 10 cells with non-zero counts
- Table columns include the expected haystack outputs (p-values, log p-values, D_KL divergence)
- Runtime of the full haystack call on 5,250 cells / 13,756 genes / 50 PCs is on the order of ~225 s (median mode)

## Limitations

- P-values can be inflated due to double use of gene expression data (input coordinates such as PCs contain a large proportion of the variability of the original data); fix deferred to future updates
- The method requires binary detection data; continuous measures of expression (e.g. normalized read counts) are not supported — two genes with the same detection pattern get identical D_KL and p-value regardless of expression level
- Zero counts may reflect dropout events from lack of sensitivity; imputation is not applied
- The definition of the detection threshold is left to the user, introducing potential arbitrariness

## Evidence

- [abstract] Our method uses Kullback-Leibler divergence to find genes that are expressed in subsets of cells that are non-randomly positioned in a multidimensional space: "Our method uses Kullback-Leibler divergence to find genes that are expressed in subsets of cells that are non-randomly positioned in a multidimensional space"
- [methods] We filter genes with less 10 cells with non-zero counts.: "We filter genes with less 10 cells with non-zero counts. This reduces the computational time by eliminating very lowly expressed genes."
- [methods] res <- haystack(dat.tsne, dat.expression): "res <- haystack(dat.tsne, dat.expression)"
- [discussion] The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell).: "The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell)."
- [other] Retrieve the ranked results with show_result_haystack, ordered by log p-values / D_KL divergence.: [evidence span withheld — anchor status: not_found]
- [readme] For the old binary version of singleCellHaystack as described here, you can use the binary branch on GitHub: [evidence span withheld — anchor status: external_doc]
