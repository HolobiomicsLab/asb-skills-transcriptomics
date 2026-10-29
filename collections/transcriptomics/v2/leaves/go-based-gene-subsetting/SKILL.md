---
name: go-based-gene-subsetting
description: Use when preparing a raw single-cell expression matrix (genes as rows,
  cells as columns, gene_type 'gene_names') for redPATH pseudotime reconstruction
  and you need to subset to GO development genes before filtering.
when_to_use_negative:
- The trajectory method of interest relies on cycling or cell-cycle genes that GO
  development subsetting would discard
- Input genes are not standard gene name identifiers compatible with the 'gene_names'
  gene_type mapping
- A pseudotime method other than redPATH is used and requires a different (e.g. highly
  variable gene) feature selection scheme
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: redPATH
  role: Provides the get_GO_exp and filter_exp R functions that perform GO-based gene
    subsetting and dispersion filtering prior to pseudotime inference
  repo: https://github.com/tinglabs/redPATH
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:b3e51638c49e6fa4d7d7ffbfe20a432c5c1c6c58f2f072882d55d0307c400504
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglabs/redPATH
  tools:
  - DPT
  - Monocle2
  - R
  - SCORPIUS
  - TSCAN
  - reCAT
  - redPATH
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# go-based-gene-subsetting

## Summary

Reduce a gene-by-cell single-cell RNA-seq matrix to development-related Gene Ontology genes and filter by dispersion before pseudotime inference with redPATH. This preprocessing concentrates the Hamiltonian-path ordering on biologically relevant developmental signal and removes low-information genes.

## When to use

Use when preparing a raw single-cell expression matrix (genes as rows, cells as columns, gene_type 'gene_names') for redPATH pseudotime reconstruction and you need to subset to GO development genes before filtering.

## When NOT to use

- The trajectory method of interest relies on cycling or cell-cycle genes that GO development subsetting would discard
- Input genes are not standard gene name identifiers compatible with the 'gene_names' gene_type mapping
- A pseudotime method other than redPATH is used and requires a different (e.g. highly variable gene) feature selection scheme

## Inputs

- Raw gene-by-cell single-cell RNA-seq expression matrix with gene names as identifiers (e.g. llorens_exprs, m genes x n cells)
- Gene type designation ('gene_names') and organism identifier ('mouse') for GO mapping

## Outputs

- GO-subsetted expression matrix containing only development-related genes (get_GO_exp output, subset_GO_data)
- Dispersion-filtered matrix (filter_exp output, filtered_GO_data) ready for redpath() pseudotime inference

## How to apply

Run get_GO_exp on the raw expression matrix (e.g. llorens_exprs, 145 neural stem cells) with gene_type = 'gene_names', organism = 'mouse', and takelog = F, which reduces genes to those annotated under Cell Development, Cell Morphology, Cell Differentiation, Cell Fate and Cell Maturation from Gene Ontology. Then apply filter_exp to the GO subset with dispersion_threshold = 10 and threads = 2, retaining only genes passing the simple dispersion threshold. The resulting filtered_GO_data matrix is the direct input to redpath(threadnum = 4, base_path_range = c(4:8)); the same filtered matrix should also be fed to baselines (SCORPIUS, TSCAN, Monocle2, DPT) so orderings are comparable. Rationale: restricting to development GO terms plus dispersion filtering removes noise genes that would otherwise distort the consensus Hamiltonian path (cHMT) pseudotime.

## Related tools

- **redPATH** (Provides the get_GO_exp and filter_exp R functions that perform GO-based gene subsetting and dispersion filtering prior to pseudotime inference) — https://github.com/tinglabs/redPATH

## Examples

```
subset_GO_data <- get_GO_exp(llorens_exprs, gene_type = "gene_names", organism = "mouse", takelog = F); filtered_GO_data <- filter_exp(subset_GO_data, dispersion_threshold = 10, threads = 2)
```

## Evaluation signals

- get_GO_exp returns a matrix whose row (gene) count is strictly smaller than the input matrix and whose gene IDs are all mapped to the five development GO categories
- filter_exp with dispersion_threshold = 10 further reduces the GO subset; no error or NA cells are introduced (cell columns preserved)
- The filtered matrix runs successfully through redpath(threadnum = 4, base_path_range = c(4:8)) without dimension errors
- Downstream evaluation metrics (CI, BI, KC) computed on the resulting pseudotime (e.g. redPATH CI 0.868, BI 0.973, KC 0.832 on the 56-cell NSC-Llorens subset) meet or exceed baseline methods run on the identical filtered matrix

## Limitations

- Subsetting to development-related GO genes assumes the biological process of interest is development/differentiation; genes outside these GO categories are excluded from pseudotime inference
- The dispersion_threshold = 10 cutoff is a 'simple dispersion threshold' and may not suit datasets with different dynamic ranges
- The article notes the downstream approach 'may fail when the pseudotime results perform poorly', so poor feature selection propagates to poor ordering

## Evidence

- [methods] GO subset command with exact parameters: "subset_GO_data <- get_GO_exp(input_data, gene_type = "gene_names", organism = "mouse", takelog = F)"
- [methods] Filter step with dispersion threshold: "filtered_GO_data <- filter_exp(subset_GO_data, dispersion_threshold = 10, threads = 2)"
- [readme] README explanation of GO categories used for subsetting: "Reduce genes to ones which are related to Cell Development, Cell Morphology, Cell Differentiation, Cell Fate and Cell Maturation (from Gene Ontology)"
- [readme] redPATH call that consumes the filtered matrix: "redpath_pseudotime <- redpath(filtered_GO_data, threadnum = 4, base_path_range = c(4:8))"
