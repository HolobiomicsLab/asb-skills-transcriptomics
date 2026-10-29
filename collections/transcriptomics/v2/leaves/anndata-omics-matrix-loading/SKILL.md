---
name: anndata-omics-matrix-loading
description: Use when the input is a raw protein-by-condition abundance matrix in
  TSV or CSV from scRNA-seq (Human Protein Atlas) or proteomics (PRIDE) data and it
  must be ingested into a scverse-compatible workflow for VAE-based network inference.
when_to_use_negative:
- The data are already an AnnData object or another tool's processed feature table
  — this skill is only the ingestion step for raw TSV/CSV files
- The goal is gene-level or cell-level single-cell analysis rather than protein-by-condition
  matrices for functional association network inference
- The downstream task does not use FAVA/favapy — a generic AnnData loader (e.g. scanpy.read_*
  ) may be more appropriate
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3520
tools:
- name: favapy
  role: Provides the load_data routine that parses TSV/CSV omics files into an AnnData-compatible
    container for the FAVA pipeline
  repo: https://github.com/mikelkou/fava
- name: AnnData
  role: Standard data container used by favapy load_data to hold the loaded matrix
    within the scverse ecosystem
- name: FAVA
  role: Downstream tool whose VAE stage consumes the loaded matrix; this loading step
    is its required first stage
  repo: https://github.com/mikelkou/fava
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:95aa181905dbc4d2325cc1f94f7d4d9533da55307c4b181ad73903980dcea74f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - AnnData
  - FAVA
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# anndata-omics-matrix-loading

## Summary

Loads high-dimensional omics matrices (scRNA-seq or proteomics, TSV or CSV) into FAVA's pipeline via the favapy load_data routine with AnnData-compatible handling, so that downstream VAE compression and network inference operate on a validated feature-by-condition matrix.

## When to use

Use when the input is a raw protein-by-condition abundance matrix in TSV or CSV from scRNA-seq (Human Protein Atlas) or proteomics (PRIDE) data and it must be ingested into a scverse-compatible workflow for VAE-based network inference.

## When NOT to use

- The data are already an AnnData object or another tool's processed feature table — this skill is only the ingestion step for raw TSV/CSV files
- The goal is gene-level or cell-level single-cell analysis rather than protein-by-condition matrices for functional association network inference
- The downstream task does not use FAVA/favapy — a generic AnnData loader (e.g. scanpy.read_* ) may be more appropriate

## Inputs

- Protein-by-condition abundance matrix in TSV or CSV format (scRNA-seq from Human Protein Atlas or proteomics from PRIDE)
- Path to the input data file (per the favapy command line argument)

## Outputs

- AnnData-compatible in-memory object holding the omics matrix, ready for the FAVA VAE stage

## How to apply

Pass the input data file to the favapy command line entry point, which resolves the format via the -t option (TSV or CSV) and invokes the load_data routine with AnnData-compatible handling, placing the data in the scverse ecosystem's standard container. Verify that rows correspond to proteins and columns to conditions/samples before proceeding, since FAVA compresses this high-dimensional matrix (supporting over 0.5 million conditions) into a dense latent space. The loaded AnnData object then feeds directly into FAVA's VAE member function for dimensionality reduction; correct loading is the precondition for producing one latent vector per input protein. Because favapy is installed via pip and is part of the scverse ecosystem, the loaded object can be interoperated with other AnnData-based tools.

## Related tools

- **favapy** (Provides the load_data routine that parses TSV/CSV omics files into an AnnData-compatible container for the FAVA pipeline) — https://github.com/mikelkou/fava
- **AnnData** (Standard data container used by favapy load_data to hold the loaded matrix within the scverse ecosystem)
- **FAVA** (Downstream tool whose VAE stage consumes the loaded matrix; this loading step is its required first stage) — https://github.com/mikelkou/fava

## Examples

```
favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- The input file parses without format errors when the TSV/CSV type is declared via the -t option
- The loaded matrix has proteins as features and conditions as observations, matching FAVA's expectation of one latent vector per input protein downstream
- Row/column dimensions match the raw file exactly (no dropped proteins or conditions) before VAE compression
- The loaded object is AnnData-compatible and interoperable with the scverse ecosystem
- Datasets with very large condition counts (over 0.5 million, as FAVA is reported to process) load without memory failure

## Limitations

- Only TSV and CSV input formats are described; other formats (e.g. .h5ad) are not part of the documented load path
- Loading alone does not validate data quality — sparse or redundant matrices (many transcripts/proteins not consistently observed) must still be addressed by the downstream VAE compression
- Scaling to very large datasets depends on available memory; FAVA's reported capability is over 0.5 million conditions

## Evidence

- [methods] Load the input data file (TSV or CSV, per the -t option) with the favapy load_data routine using AnnData-compatible handling.: [evidence span withheld — anchor status: not_found]
- [abstract] FAVA functions within the scverse ecosystem, employing AnnData as its input source: "FAVA functions within the scverse ecosystem, employing AnnData as its input source"
- [intro] large-scale scRNA-seq and proteomics data from the Human Protein Atlas, and from PRIDE, respectively: "large-scale scRNA-seq and proteomics data from the Human Protein Atlas, and from PRIDE, respectively"
- [methods] You can install FAVA using pip: "You can install FAVA using pip:"
- [methods] Run FAVA from the command line: "Run FAVA from the command line as follows:"
