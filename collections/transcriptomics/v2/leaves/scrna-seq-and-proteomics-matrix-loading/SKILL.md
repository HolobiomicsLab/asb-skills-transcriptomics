---
name: scrna-seq-and-proteomics-matrix-loading
description: Use when the task requires ingesting a cell-by-gene scRNA-seq matrix
  or an experiment-by-protein proteomics matrix (tsv/csv, hundreds of thousands of
  rows) as input for FAVA network inference or a PCC baseline on the same data.
when_to_use_negative:
- Input is not an expression matrix but a pre-computed interaction or co-expression
  network — FAVA infers networks from raw high-dimensional omics data, not from existing
  networks
- Data are single-sample or very small (FAVA targets high-dimensional data with sparseness/redundancy;
  datasets can exceed 0.5 million conditions)
- You need R/Bioconductor or non-tabular formats (e.g. raw mzML) — favapy reads tsv/csv
  expression matrices, not raw proteomics spectra files
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3557
- http://edamontology.org/topic_0121
tools:
- name: favapy
  role: Installs the FAVA pipeline and provides load_data plus the command-line entry
    point that reads the tsv/csv expression matrix
  repo: https://github.com/mikelkou/fava
- name: FAVA (favapy)
  role: Uses AnnData as its input source for the loaded expression matrix within the
    scverse ecosystem
  repo: https://github.com/mikelkou/fava
- name: AnnData
  role: Data structure into which the loaded scRNA-seq/proteomics matrix is wrapped
    before VAE compression
  repo: https://github.com/scverse/ecosystem-packages/tree/main/packages/favapy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:41068558a338c27b30cd086ec507be4e86344f3d61c16f2626ecad129bae4398
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - AnnData
  - FAVA (favapy)
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3557
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# scrna-seq-and-proteomics-matrix-loading

## Summary

Load large-scale single-cell RNA-seq and bulk proteomics expression matrices (e.g. Human Protein Atlas ~566,109 cells; PRIDE 633 experiments / 18,846 proteins) as the input layer for FAVA-based functional association network inference, using AnnData-compatible, tsv/csv tabular input.

## When to use

Use when the task requires ingesting a cell-by-gene scRNA-seq matrix or an experiment-by-protein proteomics matrix (tsv/csv, hundreds of thousands of rows) as input for FAVA network inference or a PCC baseline on the same data.

## When NOT to use

- Input is not an expression matrix but a pre-computed interaction or co-expression network — FAVA infers networks from raw high-dimensional omics data, not from existing networks
- Data are single-sample or very small (FAVA targets high-dimensional data with sparseness/redundancy; datasets can exceed 0.5 million conditions)
- You need R/Bioconductor or non-tabular formats (e.g. raw mzML) — favapy reads tsv/csv expression matrices, not raw proteomics spectra files

## Inputs

- Human Protein Atlas scRNA-seq expression matrix (tsv/csv, 566,109 cells, genes as protein identifiers)
- PRIDE bulk proteomics matrix (tsv/csv, 633 experiments, 18,846 proteins)
- Path to data file passed to favapy

## Outputs

- AnnData-wrapped dense expression matrix consumed by the FAVA VAE
- Protein pair list (create_protein_pairs) covering both directions proteinA-proteinB and proteinB-proteinA

## How to apply

Prepare the expression matrix as a tsv/csv file — rows are conditions (cells for Human Protein Atlas scRNA-seq, experiments for PRIDE bulk proteomics) and columns are proteins — sized up to 566,109 cells (scRNA-seq) or 633 experiments x 18,846 proteins (proteomics). FAVA consumes this file directly from the command line via favapy, which internally uses load_data to read the matrix and wraps it in AnnData (FAVA functions within the scverse ecosystem), so no manual AnnData conversion is required. The same loaded matrix must be reused for the PCC baseline so both methods are benchmarked on identical inputs. After loading, FAVA auto-scales hidden/latent dimensions to the input size, so the loader must preserve the full dimensionality rather than pre-filtering proteins.

## Related tools

- **favapy** (Installs the FAVA pipeline and provides load_data plus the command-line entry point that reads the tsv/csv expression matrix) — https://github.com/mikelkou/fava
- **FAVA (favapy)** (Uses AnnData as its input source for the loaded expression matrix within the scverse ecosystem) — https://github.com/mikelkou/fava
- **AnnData** (Data structure into which the loaded scRNA-seq/proteomics matrix is wrapped before VAE compression) — https://github.com/scverse/ecosystem-packages/tree/main/packages/favapy

## Examples

```
favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- Loaded matrix dimensions match expectation: 566,109 cells for the HPA scRNA-seq file and 633 experiments x 18,846 proteins for the PRIDE file
- favapy accepts the tsv/csv file without a manual AnnData conversion step (AnnData is FAVA's input source)
- Output contains 100,000 top interactions, each listed in both directions (proteinA-proteinB and proteinB-proteinA)
- The same loaded matrix yields a PCC baseline sweep over the full cutoff range 1 to -1 with no re-loading differences between methods
- Network derived from the loaded HPA matrix benchmarks against KEGG with >5000 true positives and <500 false positives (precision >90%)

## Limitations

- Input must be tabular tsv/csv; favapy's load_data expects this format and AnnData wrapping is handled internally, so other formats need prior conversion
- Simple co-expression analysis on such data fails due to high sparseness (many transcripts/proteins are not consistently observed in each cell/sample) and redundancy (many similar cells/samples), which is why VAE compression precedes correlation computation
- Existing networks derived from such data can be biased by data availability, so loading alone does not fix literature bias — the benefit comes from FAVA's latent-space correlations

## Evidence

- [intro] large-scale scRNA-seq and proteomics data from the Human Protein Atlas, and from PRIDE, respectively: "large-scale scRNA-seq and proteomics data from the Human Protein Atlas, and from PRIDE, respectively"
- [methods] favapy CLI usage: "Run FAVA from the command line as follows:

.. code-block:: bash

   favapy <path-to-data-file> <path-to-save-output>"
- [abstract] AnnData as input source: "FAVA functions within the scverse ecosystem, employing AnnData as its input source"
- [intro] sparseness/redundancy rationale: "this approach does not work well due to high sparseness (many transcripts/proteins are not consistently observed in each cell/sample) and redundancy"
- [methods] load_data member: ":exclude-members: argument_parser, custom_formatwarning, main, load_data, create_protein_pairs"
