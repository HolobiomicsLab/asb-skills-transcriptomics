---
name: scrnaseq-dataset-retrieval-geo-scp
description: Use when a benchmark or method comparison requires raw scRNA-seq count
  matrices with ground-truth cell labels and the paper lists source repositories (GEO/SCP
  accession numbers, e.g. Table 1 of the scDHA article) to fetch them.
when_to_use_negative:
- The task is clustering a single user-provided dataset that is already locally available
  — no retrieval step is needed.
- The analysis needs datasets without published cell-type labels, since ARI benchmarking
  against known labels is impossible.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDHA
  role: Downstream consumer of the retrieved labeled datasets; its clustering output
    is scored against the known cell labels via adjusted Rand index
  repo: https://github.com/duct317/scDHA
- name: CIDR
  role: Comparator clustering method run on the same retrieved datasets (second-best
    method, average ARI 0.5)
  repo: https://github.com/VCCRI/CIDR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:fd81799f412033a28a3e5f94f86a4879e986f7aaba46565a3279842a3a50cc1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scDHA
  tools:
  - CIDR
  - SC3
  - SCANPY
  - SEURAT
  - SINCERA
  - scDHA
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# scrnaseq-dataset-retrieval-geo-scp

## Summary

Retrieve scRNA-seq benchmark datasets with known cell-type labels from GEO/SCP using accession numbers, as required to reproduce the 34-dataset clustering benchmark (ARI comparison) from the scDHA paper (Nature Communications, 2021).

## When to use

Use when a benchmark or method comparison requires raw scRNA-seq count matrices with ground-truth cell labels and the paper lists source repositories (GEO/SCP accession numbers, e.g. Table 1 of the scDHA article) to fetch them.

## When NOT to use

- The task is clustering a single user-provided dataset that is already locally available — no retrieval step is needed.
- The analysis needs datasets without published cell-type labels, since ARI benchmarking against known labels is impossible.

## Inputs

- GEO or SCP accession numbers for the 34 scRNA-seq datasets listed in Table 1 of the scDHA article
- Expected cell-type label annotations accompanying each dataset

## Outputs

- 34 downloaded scRNA-seq count matrices (one per dataset)
- Paired ground-truth cell-type label vectors for ARI evaluation
- Manifest linking each dataset to its accession number and source repository

## How to apply

Collect the accession numbers listed in the paper's dataset table (Table 1 of the scDHA article enumerates 34 scRNA-seq data sets with known cell types, each with a GEO or Single Cell Portal accession). For each accession, download the raw count matrix (cells x genes or genes x cells) and the published cell-type labels. Keep the datasets paired with their labels so downstream methods can be scored; the benchmark workflow computes the adjusted Rand index of each method's clusters against these known cell labels. Log-transform downloaded counts (e.g. log2(counts + 1)) before feeding them into scDHA, consistent with the package's Goolam example.

## Related tools

- **scDHA** (Downstream consumer of the retrieved labeled datasets; its clustering output is scored against the known cell labels via adjusted Rand index) — https://github.com/duct317/scDHA
- **CIDR** (Comparator clustering method run on the same retrieved datasets (second-best method, average ARI 0.5)) — https://github.com/VCCRI/CIDR

## Evaluation signals

- All 34 datasets listed in Table 1 are retrieved and each has a matching cell-type label vector.
- Every downloaded matrix has a defined cell x gene orientation suitable for the clustering tools (scDHA expects rows as samples, columns as genes).
- Reproduced benchmark means match the reported values: scDHA average ARI of 0.81 and CIDR average ARI of 0.5 across the 34 datasets.
- Each dataset's accession in the manifest resolves to the same study described in the paper (cell counts and cell types consistent with Table 1).

## Limitations

- SCP/GEO hosting may change or restrict bulk download; accession resolution is not guaranteed stable over time.
- The article's discussion section provides no code or data availability statement, so retrieval instructions rely on the accession numbers in Table 1.
- Retrieved labels must be trusted as ground truth; any relabeling or exclusion of cell types (e.g. excluding hybrid or undefined cells, as in CIDR examples) would shift ARI values.

## Evidence

- [full_text] Retrieve the 34 scRNA-seq datasets listed in Table 1 using their GEO/SCP accession numbers.: [evidence span withheld — anchor status: not_found]
- [full_text] scDHA achieves an average ARI of 0.81 across all 34 data sets with very low variability, while the second best method, CIDR, has an average ARI of only 0.5.: [evidence span withheld — anchor status: not_found]
- [readme] Log transform the data: data <- log2(data + 1): "- Log transform the data: `data <- log2(data + 1)`"
- [readme] The input is a matrix with rows as samples and columns as genes.: [evidence span withheld — anchor status: external_doc]
