---
name: tabula-muris-count-matrix-ingestion
description: Use when raw Tabula Muris FACS per-tissue count CSVs (e.g., Diaphragm-counts.csv
  from the Figshare deposit) must be loaded into a cell-by-gene count matrix before
  QC, normalization, and PCA.
when_to_use_negative:
- Input is already a preprocessed matrix (e.g., preprocessed_pca50.npz with exactly
  50 PC columns) rather than a raw per-tissue FACS counts CSV.
- Data are Tabula Muris droplet-based 3'-end counting reads or a different modality
  (e.g., 10x PBMC scATAC-seq peak matrices in .h5/fragments.tsv.gz format) with different
  file layouts and filter rules.
- The goal is quality assessment of an existing embedding (EMBEDR p-values, perplexity
  sweeps) on an already-ingested count matrix — no re-ingestion is needed.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: pandas
  role: Read the per-tissue FACS counts CSV (e.g., Diaphragm-counts.csv) into a cell-by-gene
    DataFrame
- name: numpy
  role: Hold the numeric raw count matrix consumed by the downstream QC and PCA steps
- name: czbiohub tabula-muris data_ingest
  role: Reference implementation of the Tabula Muris FACS data ingestion whose processing
    the EMBEDR analysis reproduces
  repo: https://github.com/czbiohub/tabula-muris/tree/master/00_data_ingest
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:e9ef98d09b20a5f40dd40095b8c493d94d0dee4c198f7e795960f17f6f9c0c34
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ejohnson643/EMBEDR
  tools:
  - Python
  - czbiohub tabula-muris data_ingest
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# tabula-muris-count-matrix-ingestion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Loads raw Tabula Muris FACS scRNA-seq counts from per-tissue CSVs (00_facs_raw_data/FACS/{Tissue}-counts.csv in the Figshare deposit) into a cell-by-gene count matrix using pandas/numpy. This ingestion is the entry point of the preprocessing pipeline EMBEDR consumes before cell QC, library-size normalization, HVG scaling, and 50-component PCA.

## When to use

Use when raw Tabula Muris FACS per-tissue count CSVs (e.g., Diaphragm-counts.csv from the Figshare deposit) must be loaded into a cell-by-gene count matrix before QC, normalization, and PCA.

## When NOT to use

- Input is already a preprocessed matrix (e.g., preprocessed_pca50.npz with exactly 50 PC columns) rather than a raw per-tissue FACS counts CSV.
- Data are Tabula Muris droplet-based 3'-end counting reads or a different modality (e.g., 10x PBMC scATAC-seq peak matrices in .h5/fragments.tsv.gz format) with different file layouts and filter rules.
- The goal is quality assessment of an existing embedding (EMBEDR p-values, perplexity sweeps) on an already-ingested count matrix — no re-ingestion is needed.

## Inputs

- Raw Tabula Muris FACS per-tissue counts CSV (e.g., ../data/tabula-muris/00_facs_raw_data/FACS/Diaphragm-counts.csv from the Figshare deposit)
- Tissue name string (e.g., "diaphragm") used to construct the filename via {tissue.title()}-counts.csv

## Outputs

- Cell-by-gene raw count matrix (pandas DataFrame / numpy array) with gene identifiers as columns and cell barcodes as rows
- Downstream persisted artifact preprocessed_pca50.npz (cell-by-50 PC numeric matrix) once QC, normalization, and PCA are applied to the ingested matrix

## How to apply

Resolve the per-tissue file using the paper's path pattern: file_path = "../data/tabula-muris/" and data_name = f"00_facs_raw_data/FACS/{tissue.title()}-counts.csv" (e.g., Diaphragm-counts.csv for tissue = "diaphragm"), with raw data coming from the Tabula Muris Consortium Figshare deposit. Read the CSV into a pandas DataFrame and convert it to a numeric numpy cell-by-gene count matrix, keeping gene identifiers as columns and cell barcodes as rows; leave counts raw and untransformed at this stage. This mirrors the czbiohub tabula-muris data_ingest code that the EMBEDR paper cites as the source of its Tabula Muris processing. The ingested raw matrix is the substrate for the paper's subsequent filters (>=500 detected genes, >=50,000 total reads, and caps on spike-in, ribosomal, and Rn45s read fractions per the Experimental Procedures), library-size normalization, log transform, HVG selection/scaling, and scikit-learn PCA retaining 50 components saved as preprocessed_pca50.npz.

## Related tools

- **pandas** (Read the per-tissue FACS counts CSV (e.g., Diaphragm-counts.csv) into a cell-by-gene DataFrame)
- **numpy** (Hold the numeric raw count matrix consumed by the downstream QC and PCA steps)
- **czbiohub tabula-muris data_ingest** (Reference implementation of the Tabula Muris FACS data ingestion whose processing the EMBEDR analysis reproduces) — https://github.com/czbiohub/tabula-muris/tree/master/00_data_ingest

## Examples

```
tissue = "diaphragm"; counts = pd.read_csv(f"../data/tabula-muris/00_facs_raw_data/FACS/{tissue.title()}-counts.csv")
```

## Evaluation signals

- Loaded matrix is numeric and non-negative (raw integer counts), with one row per cell barcode and one column per gene; no log-transformed or scaled values present at this stage.
- Constructed filename matches f"00_facs_raw_data/FACS/{tissue.title()}-counts.csv" (e.g., Diaphragm-counts.csv) under ../data/tabula-muris/ from the Figshare deposit.
- Per-cell QC is computable on the ingested matrix: cells can be checked against >=500 detected genes and >=50,000 total reads, and spike-in, ribosomal, and Rn45s read fractions can be capped per the Experimental Procedures thresholds.
- After the full pipeline, preprocessed_pca50.npz exists, loads as a numeric matrix with exactly 50 columns, and its row count equals the number of cells passing the stated gene-count and read-count thresholds.

## Limitations

- Pattern applies only to FACS-based full-length transcript data; Tabula Muris also contains droplet-based 3'-end counting data (thousands of cells per organ at low coverage) with different per-organ files, so the FACS path pattern does not transfer.
- Ingestion alone performs no QC or normalization — skipping the subsequent filter steps (>=500 genes, >=50,000 reads, spike-in/ribosomal/Rn45s fraction caps) leaves low-quality cells in the matrix destined for PCA.
- File resolution depends on title-casing the tissue name (tissue.title()); naming mismatches against the Figshare deposit layout (00_facs_raw_data/FACS/) will break loading.

## Evidence

- [readme] FACS-based full length transcript analysis, which provides higher sensitivity and coverage.: [evidence span withheld — anchor status: external_doc]
- [readme] The Tabula muris project is a a compendium of single cell transcriptomic data from the mouse containing nearly 100,000 cells from 20 organs and tissues.: [evidence span withheld — anchor status: external_doc]
