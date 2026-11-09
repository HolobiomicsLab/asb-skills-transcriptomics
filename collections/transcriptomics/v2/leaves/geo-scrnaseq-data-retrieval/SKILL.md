---
name: geo-scrnaseq-data-retrieval
description: Use when an analysis requires reproducing a published single-cell result
  from raw public data, specifically when a GEO accession (e.g., GSE122043) must be
  converted into MDIC3's exp.txt/metadata.txt input format.
when_to_use_negative:
- The expression data is not a gene-by-cell matrix from a public repository — e.g.,
  already a processed feature table or spatial transcriptomics object that needs no
  GEO retrieval
- Cell labels are unavailable or cannot be matched one-to-one to the expression matrix
  columns, since MDIC3 requires matching exp.txt and metadata.txt
- A cell type contains fewer than 10 cells, which the MDIC3 README advises against
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MDIC3
  role: Consumes the exp.txt and metadata.txt files produced by this retrieval/formatting
    step for cell-cell communication inference
  repo: https://github.com/LYxiaotai/MDIC3
- name: GNIPLR
  role: Downstream GRN inference tool (lucky.GRN_GNIPLR) that operates on the retrieved
    expression matrix once formatted
  repo: https://github.com/zyllluck/GNIPLR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:c89e498234535300e6f9384b4f503d29c6ce5b99f88253d70e66c62ddeb23c65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LYxiaotai/MDIC3
  tools:
  - GNIPLR
  - MDIC3
  - Python
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# geo-scrnaseq-data-retrieval

## Summary

Retrieve scRNA-seq datasets from GEO accessions (e.g., E13.5 and E14.5 mouse skin from GSE122043) and reformat them as MDIC3-compatible gene-by-cell exp.txt and cell-label metadata.txt files so downstream communication inference can run on published data.

## When to use

Use when an analysis requires reproducing a published single-cell result from raw public data, specifically when a GEO accession (e.g., GSE122043) must be converted into MDIC3's exp.txt/metadata.txt input format.

## When NOT to use

- The expression data is not a gene-by-cell matrix from a public repository — e.g., already a processed feature table or spatial transcriptomics object that needs no GEO retrieval
- Cell labels are unavailable or cannot be matched one-to-one to the expression matrix columns, since MDIC3 requires matching exp.txt and metadata.txt
- A cell type contains fewer than 10 cells, which the MDIC3 README advises against

## Inputs

- GEO accession identifier (e.g., GSE122043) pointing to a published scRNA-seq dataset
- Downloaded raw count matrix or expression table from GEO supplementary files
- Cell-type annotation/labels for the cells in the downloaded dataset

## Outputs

- exp.txt: gene-by-cell expression matrix (.txt, rows = genes, columns = cells, first row contains cell names only)
- metadata.txt: two-column cell-label file whose first column matches exp.txt cell names exactly

## How to apply

Locate the dataset in GEO by accession (e.g., E13.5 and E14.5 mouse skin under GSE122043) and download the raw count or expression matrix. Reshape the expression data so each row is a gene and each column is a cell, saving it as exp.txt with a first row of cell names only (no header field). Build metadata.txt as a two-column file where the first column of cell names matches exactly the first row of exp.txt and the second column holds the cell-type labels. Check the two files carefully before running MDIC3, and ensure each cell type contains at least 10 cells as suggested by the README.

## Related tools

- **MDIC3** (Consumes the exp.txt and metadata.txt files produced by this retrieval/formatting step for cell-cell communication inference) — https://github.com/LYxiaotai/MDIC3
- **GNIPLR** (Downstream GRN inference tool (lucky.GRN_GNIPLR) that operates on the retrieved expression matrix once formatted) — https://github.com/zyllluck/GNIPLR

## Examples

```
AA, gene_exp, cellname = lucky.readexp('exp.txt'); labels, label_index, label_cell = lucky.readlabel('metadata.txt')
```

## Evaluation signals

- First row of exp.txt contains only cell names with no header field, per the README's required format
- First column of metadata.txt matches exactly with the first row of exp.txt (cell name identity check)
- The two output files are valid .txt with rows = genes and columns = cells
- Retrieved GEO accession matches the study (e.g., GSE122043 yields both E13.5 and E14.5 mouse skin data)
- Each cell label in metadata.txt is represented by at least 10 cells, as suggested in the README

## Limitations

- GEO supplementary files may need reformatting (transposition, header removal) before matching MDIC3's required exp.txt schema
- The README truncates before covering all protocol details, so edge cases in data preparation may require consulting the repository protocol_Data examples directly

## Evidence

- [discussion] The E14.5 mouse skin dataset can be obtained via GEO: GSE122043.: "The E14.5 mouse skin dataset can be obtained via GEO: GSE122043."
- [other] Retrieve the E13.5 and E14.5 mouse skin scRNA-seq datasets from GEO accession GSE122043 and format each as a gene-by-cell expression table (exp.txt) plus a cell-label metadata file (metadata.txt) as required by MDIC3.: [evidence span withheld — anchor status: not_found]
- [readme] The single-cell gene expression data must be a *.txt file, while each row represents a gene and each column represents a cell.: "The single-cell gene expression data must be a *.txt file, while each row represents a gene and each column represents a cell."
- [readme] Note that the first column of the scRNAseq metadata should match exactly with the first row of the single-cell gene expression data.: "Note that the first column of the scRNAseq metadata should match exactly with the first row of the single-cell gene expression data."
