---
name: tcga-data-handling
description: Use when TCGA RNA-Seq gene expression matrices for a cancer type (e.g.,
  KIRC, LIHC, PRAD) must be retrieved, metadata-queried, and preprocessed (zFPKM filtering,
  gene-type mapping) before network or DE analysis.
when_to_use_negative:
- Data already comes as a preprocessed, filtered expression matrix (the skill would
  duplicate preprocessing such as zFPKM > −3.0 selection)
- Analysis targets non-TCGA or non-RNA-Seq data sources; GMQL retrieval against the
  TCGA repository is specific to this setting
- Single-gene or single-sample interrogation without the normal/cancer sample structure
  needed for downstream comparison
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GMQL
  role: Query language used to retrieve TCGA genomic data and metadata from the TCGA
    repository
  repo: http://gmql.eu
- name: PyGMQL
  role: Python interface through which the TCGA gene expression data are retrieved
    into the Datasets/Datasets_created folders
  repo: https://pygmql.readthedocs.io/en/latest/
- name: Python
  role: Implements preprocessing.py and the main pipeline steps applied to the retrieved
    TCGA data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:6b2bf5dab0e2ca2e9429f42446de26494fe13d938906bd6f1a80fffb41e3bac0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - GMQL
  - PyGMQL
  - Python
  - R/Bioconductor
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# tcga-data-handling

## Summary

Retrieves and preprocesses TCGA gene expression data for specific cancer types (KIRC, LIHC, PRAD) using the GenoMetric Query Language (GMQL) and PyGMQL, applying zFPKM-based RNA-Seq data selection so that downstream co-expression network construction and differential expression analysis operate on consistently filtered counts.

## When to use

Use when TCGA RNA-Seq gene expression matrices for a cancer type (e.g., KIRC, LIHC, PRAD) must be retrieved, metadata-queried, and preprocessed (zFPKM filtering, gene-type mapping) before network or DE analysis.

## When NOT to use

- Data already comes as a preprocessed, filtered expression matrix (the skill would duplicate preprocessing such as zFPKM > −3.0 selection)
- Analysis targets non-TCGA or non-RNA-Seq data sources; GMQL retrieval against the TCGA repository is specific to this setting
- Single-gene or single-sample interrogation without the normal/cancer sample structure needed for downstream comparison

## Inputs

- TCGA RNA-Seq gene count/expression data with metadata for KIRC, LIHC, or PRAD cancer types
- Gene symbol-to-type correspondence table (matrix_GTypeGName.xlsx)
- GMQL query definitions for genomic data and metadata retrieval

## Outputs

- Preprocessed TCGA gene expression count matrices per cancer type, split by normal and cancer samples (written to Datasets_created)
- Preprocessed gene/miRNA lists annotated with gene type (protein-coding/lncRNA vs. miRNA)

## How to apply

Retrieve TCGA gene expression datasets from the TCGA repository through GMQL queries executed with PyGMQL, storing raw retrieved data in a Datasets folder and generated matrices in a Datasets_created folder. Select RNA-Seq data with a mean normalized value of fragments per kilobase of transcript per million mapped reads (zFPKM) higher than −3.0. Run the preprocessing step (preprocessing.py) as the first stage of the pipeline, producing preprocessed TCGA RNA-Seq gene count matrices for normal and cancer samples that feed DESeq2 differential expression and co-expression network construction. Map gene symbols to gene types (protein-coding/long non-coding RNAs vs. miRNAs) using a gene-type correspondence table (matrix_GTypeGName.xlsx). Expect on average 20% of RNA genes and 81% of miRNAs to be retained after pre-processing; validate outputs as CSV/expression tables containing only genes passing the zFPKM threshold for each cancer type.

## Related tools

- **GMQL** (Query language used to retrieve TCGA genomic data and metadata from the TCGA repository) — http://gmql.eu
- **PyGMQL** (Python interface through which the TCGA gene expression data are retrieved into the Datasets/Datasets_created folders) — https://pygmql.readthedocs.io/en/latest/
- **Python** (Implements preprocessing.py and the main pipeline steps applied to the retrieved TCGA data)

## Evaluation signals

- Retrieved/preprocessed TCGA data files exist in the Datasets and Datasets_created folders after running preprocessing
- All retained RNA-Seq data have mean zFPKM > −3.0
- Pre-processed matrices are split into normal and cancer samples for each cancer type (KIRC, LIHC, PRAD)
- Approximately 20% of RNA genes and 81% of miRNAs are retained after pre-processing, consistent with the paper's reported averages
- Each retained feature is correctly annotated with gene type via the matrix_GTypeGName.xlsx correspondence table

## Limitations

- The GMQL/PyGMQL retrieval depends on the external TCGA repository and the version of data hosted there; no accession IDs or data versions are specified in the paper
- The zFPKM > −3.0 selection discards ~80% of RNA genes on average, which may exclude low-expression but potentially relevant genes from downstream networks
- Analysis in the released repository is restricted to the LIHC tumor example; results for KIRC and PRAD are only provided as extracted gene symbol lists

## Evidence

- [methods] GMQL retrieval of TCGA data: "from the TCGA repository17 using the GenoMetric Query Language (GMQL), an innovative query language for genomic data and metadata25"
- [methods] zFPKM selection threshold: "The selected RNA-Seq data have a mean value of normalized fragments per kilobase of transcript per million mapped reads (zFPKM) higher than −3.0"
- [intro] Preprocessing as first pipeline step: "preprocessing.py ... python files which contain the main steps of the process"
- [results] Post-preprocessing retention rates: "On average, 20% of RNA (protein-coding or long non-coding) genes and 81% of miRNAs are extracted"
- [readme] Repository data organization: [evidence span withheld — anchor status: not_found]
