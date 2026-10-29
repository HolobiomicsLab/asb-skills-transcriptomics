---
name: cell-cycle-gene-annotation-mapping
description: 'Use when preparing a gene-by-cell expression matrix for ccRemover: gene
  names must be mapped to human (''human'') or mouse (''mouse'') cell-cycle annotations
  to build the if_cc indicator vector before running ccRemover.'
when_to_use_negative:
- The dataset's gene identifiers cannot be matched to Biomart or org.Xx.eg.db annotations
  (e.g. outdated or non-standard symbols), since ccRemover relies on a known set of
  cell-cycle genes and incomplete annotation mapping will corrupt the control-gene
  set
- The analysis goal is removal of non-cell-cycle confounders (batch, technical replicates)
  — ccRemover is designed for removing the cell-cycle effect exclusively and will
  not remove these effects
- No gene-name annotation at all is available (e.g. the matrix lacks gene symbols),
  making species-level annotation mapping impossible
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: gene_indexer
  role: ccRemover helper function that maps gene names to cell-cycle gene indices
    given species and name_type
  repo: cran/ccRemover
- name: Biomart
  role: Used to download lists of genes annotated to the cell cycle
- name: org.Mm.eg.db
  role: Mouse cell-cycle gene annotations
- name: org.Hs.eg.db
  role: Human cell-cycle gene annotations
- name: ccRemover
  role: Consumes the if_cc indicator vector produced by this mapping to split genes
    into cell-cycle and control sets
  repo: cran/ccRemover
provenance:
  source_task_ids:
  - task_005
  - task_001
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:c4424ca26e9ecf33ea604a56e297034e25b3f6c5ce58885d7c855e780272fee2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cran/ccRemover
  tools:
  - Biomart
  - Python
  - ccRemover
  - gene_indexer
  - get_diff
  - org.Hs.eg.db
  - org.Mm.eg.db
  - stats
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# cell-cycle-gene-annotation-mapping

## Summary

Maps a scRNA-Seq dataset's gene identifiers to known cell-cycle annotated genes using annotation databases (Biomart, org.Mm.eg.db, org.Hs.eg.db via the gene_indexer helper), producing the if_cc boolean indicator vector that ccRemover requires to split genes into cell-cycle (XA) and control (XB) sets.

## When to use

Use when preparing a gene-by-cell expression matrix for ccRemover: gene names must be mapped to human ('human') or mouse ('mouse') cell-cycle annotations to build the if_cc indicator vector before running ccRemover.

## When NOT to use

- The dataset's gene identifiers cannot be matched to Biomart or org.Xx.eg.db annotations (e.g. outdated or non-standard symbols), since ccRemover relies on a known set of cell-cycle genes and incomplete annotation mapping will corrupt the control-gene set
- The analysis goal is removal of non-cell-cycle confounders (batch, technical replicates) — ccRemover is designed for removing the cell-cycle effect exclusively and will not remove these effects
- No gene-name annotation at all is available (e.g. the matrix lacks gene symbols), making species-level annotation mapping impossible

## Inputs

- Vector of gene names/symbols from the scRNA-Seq expression matrix (genes as rows, cells as columns)
- Species identifier: 'human' or 'mouse' (Homo Sapiens or Mus Musculus)
- Gene name type, e.g. name_type = 'symbols'

## Outputs

- if_cc boolean indicator vector marking cell-cycle annotated genes (e.g. 757 of 10,977 genes in the lung adenocarcinoma dataset)
- Indices of cell-cycle annotated genes used to split the matrix into cell-cycle gene set XA and control gene set XB

## How to apply

Retrieve cell-cycle gene lists from annotation databases — Biomart to download genes annotated to the cell cycle, with org.Mm.eg.db for mouse and org.Hs.eg.db for human annotations. Use the ccRemover helper gene_indexer, passing the dataset's gene names, species ('human' or 'mouse'), and name_type (e.g. 'symbols'), to obtain the indices of cell-cycle annotated genes; in the Kim et al. lung adenocarcinoma data this identified 757 cell-cycle annotated genes among the 10,977 genes surviving filtering. Build the if_cc boolean indicator vector marking these genes, with all non-annotated genes serving as control genes (the article recommends using all genes not annotated as cell-cycle genes as controls). The if_cc vector plus the log-transformed, gene-centered matrix are then passed to ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10).

## Related tools

- **gene_indexer** (ccRemover helper function that maps gene names to cell-cycle gene indices given species and name_type) — cran/ccRemover
- **Biomart** (Used to download lists of genes annotated to the cell cycle)
- **org.Mm.eg.db** (Mouse cell-cycle gene annotations)
- **org.Hs.eg.db** (Human cell-cycle gene annotations)
- **ccRemover** (Consumes the if_cc indicator vector produced by this mapping to split genes into cell-cycle and control sets) — cran/ccRemover

## Examples

```
gene_cycle_gene_indices <- gene_indexer(gene_names, species = "human", name_type = "symbols"); if_cc <- (seq_len(nrow(dat)) %in% gene_cycle_gene_indices); xhat <- ccRemover(dat, if_cc = if_cc, cutoff = 3, max_it = 4, nboot = 200, ntop = 10)
```

## Evaluation signals

- The number of genes flagged as cell-cycle annotated should be plausible for the organism — e.g. 757 cell-cycle annotated genes among 10,977 retained genes in the Kim et al. lung adenocarcinoma data
- The if_cc vector length must equal the number of rows (genes) in the expression matrix, with non-annotated genes all marked FALSE to serve as control genes
- Gene indices returned by gene_indexer must correspond to gene names actually present in the input matrix (no unmapped/dropped symbols silently lost)
- After ccRemover runs with the derived if_cc vector, a corrected matrix xhat is produced in which cell-cycle PCs are removed while other biological signals (e.g. tumor-of-origin or cell-type structure) are preserved

## Limitations

- Annotation databases are always incomplete and inaccurate, so the known set of cell-cycle genes relied on by ccRemover may be imperfect
- If the cell-cycle effect only influences a few of the control genes, it may not be captured by the principal components and hence missed for detection
- ccRemover removes only the cell-cycle effect, not other confounding effects

## Evidence

- [methods] gene_indexer maps gene names to cell-cycle annotations: "gene_cycle_gene_indices <- gene_indexer(gene_names, species = "mouse", name_type = "symbols" )"
- [results] Biomart used to download cell-cycle gene lists: "Firstly Biomart was used to download lists of genes that were annotated to the cell cycle"
- [results] Annotation packages per species: "these were org.Mm.eg.db40 and org.Hs.eg.db41 for annotations for mouse and human respectively"
- [discussion] Control genes are non-annotated genes: "we recommend simply using all genes that are not annotated as cell-cycle genes as control genes"
- [results] 757 cell-cycle annotated genes in lung adenocarcinoma data: "leaving 10,977 genes of which 757 were annotated to the cell cycle"
- [methods] Supported species: "ccRemover is able to identify genes from Homo Sapiens ('"human"') and Mus Musculus ('"mouse"')."
- [methods] ccRemover call consuming the annotations: "xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)"
