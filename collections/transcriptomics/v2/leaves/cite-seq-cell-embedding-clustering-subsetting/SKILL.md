---
name: cite-seq-cell-embedding-clustering-subsetting
description: Use when scME joint embeddings from CITE-seq BMNC data must be subdivided
  into cell-type subclusters and the subdivision needs validation from differential
  protein (CDT ADT) markers like CD69 and CD25 rather than genes.
when_to_use_negative:
- Data are raw RNA/protein counts that have not yet been embedded with scME — generate
  the joint embedding first (scme.inference)
- The goal is differential gene expression as the primary signal — in this setting
  no significantly different genes were found across the three clusters
- Data are RNA-only single-cell data without paired protein ADT measurements, since
  the validation relies on the protein modality
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0121
tools:
- name: scME
  role: Dual-modality factor model that produces the joint cell embedding on which
    neighbor-based subclustering is performed
  repo: https://github.com/bucky527/scME
- name: scanpy
  role: Neighbor-based clustering of the joint embedding and rank_genes_groups Wilcoxon
    rank-sum differential expression
- name: anndata
  role: Wraps the RNA and protein CSV count matrices as AnnData objects for scanpy
    DE analysis
- name: pandas
  role: Loads the RNA gene counts and protein ADT counts CSV files (rows = cells,
    columns = genes)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad337
    title: 'scME: a dual-modality factor model for single-cell multiomics embedding'
  source_content_hash: sha256:bc51f958ce0da52ad6120a644d7d9fe607796952e24502a716a719fe2d69f685
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bucky527/scME
  tools:
  - anndata
  - numpy
  - pandas
  - scME
  - scanpy
  - scipy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# cite-seq-cell-embedding-clustering-subsetting

## Summary

Partition scME CITE-seq cell embeddings into fine-grained subclusters (e.g. CD4+ Memory T cells in clusters 3, 22, and 24 of the BMNC embedding) and validate the partition by differential protein (ADT) expression, using differential RNA expression as a cross-modality control.

## When to use

Use when scME joint embeddings from CITE-seq BMNC data must be subdivided into cell-type subclusters and the subdivision needs validation from differential protein (CDT ADT) markers like CD69 and CD25 rather than genes.

## When NOT to use

- Data are raw RNA/protein counts that have not yet been embedded with scME — generate the joint embedding first (scme.inference)
- The goal is differential gene expression as the primary signal — in this setting no significantly different genes were found across the three clusters
- Data are RNA-only single-cell data without paired protein ADT measurements, since the validation relies on the protein modality

## Inputs

- scME BMNC cell embeddings (joint latent representation from scme.inference)
- RNA gene counts CSV (rows = cells, columns = genes, column 0 = cell ids)
- Raw protein ADT counts CSV (rows = cells, columns = proteins, column 0 = cell ids)
- Cluster labels identifying the CD4+ Memory T cell clusters (3, 22, 24)

## Outputs

- Subcluster assignments for CD4+ Memory T cells partitioned into clusters 3, 22, and 24
- protein_de_results.csv containing the combined CD69 and CD25 differential-expression entries from both protein and RNA modalities

## How to apply

Load the scME BMNC cell embeddings and use scanpy neighbor-based clustering on the joint embedding to subset CD4+ Memory T cells into clusters 3, 22, and 24. Load the RNA gene counts and raw protein ADT counts CSV files (rows are cells, columns are genes, with column 0 holding cell ids per the README) with pandas and wrap each as an AnnData object. Run scanpy's rank_genes_groups Wilcoxon rank-sum test on the protein modality, comparing the large cluster against the two tiny clusters. Repeat the identical differential-expression procedure on the RNA modality as a control. Extract the CD69 and CD25 entries from both modality results and write the combined table to protein_de_results.csv.

## Related tools

- **scME** (Dual-modality factor model that produces the joint cell embedding on which neighbor-based subclustering is performed) — https://github.com/bucky527/scME
- **scanpy** (Neighbor-based clustering of the joint embedding and rank_genes_groups Wilcoxon rank-sum differential expression)
- **anndata** (Wraps the RNA and protein CSV count matrices as AnnData objects for scanpy DE analysis)
- **pandas** (Loads the RNA gene counts and protein ADT counts CSV files (rows = cells, columns = genes))

## Examples

```
python scme.py --rna rna_gene_counts.csv --protein protein_adt_counts.csv --output-dir embeddings/ --batch-size 256
```

## Evaluation signals

- The Wilcoxon rank-sum test on the protein modality yields significant differential expression for CD69 and CD25 across clusters 3, 22, and 24
- The identical RNA-modality control returns no significantly different genes, matching the expected protein-driven finding
- protein_de_results.csv is produced and contains combined entries extracted from both modality DE results
- The CD4+ Memory T cells are fully partitioned into the three expected subclusters (3, 22, 24) from neighbor-based clustering on the joint embedding
- Input CSVs follow the README convention where column 0 contains cell ids and columns are genes/proteins

## Limitations

- The contrast compares one large cluster against two tiny clusters, so differential-expression power is limited by the small subcluster sizes
- The discriminating signal was protein-level only — no significantly different genes were found across the three CD4+ Memory T clusters, so gene-based validation will fail
- README notes the protein-data likelihood choice (NB vs mixture negative binomial via --use-mnb) affects the scME embedding that underlies the subclustering

## Evidence

- [readme] You can use scme.py to easily obtain cell embeddings for CITE-seq data: "You can use scme.py to easily obtain cell embeddings for CITE-seq data"
- [abstract] the joint representation of multiple modalities yielded by scME can provide salient information to improve both single-cell clustering and cell-type classification: "the joint representation of multiple modalities yielded by scME can provide salient information to improve both single-cell clustering and cell-type classification"
- [readme] scME accepts as input the RNA gene counts matrix data and raw protein ADTs counts matrix data in the CSV format: "scME accepts as input the  RNA gene counts matrix data and raw protein ADTs counts matrix data in the CSV format"
- [methods] pip3 install numpy scipy pandas scikit-learn pyro-ppl matplotlib scanpy anndata scvi-tools: "pip3 install numpy scipy pandas scikit-learn pyro-ppl matplotlib scanpy anndata scvi-tools"
