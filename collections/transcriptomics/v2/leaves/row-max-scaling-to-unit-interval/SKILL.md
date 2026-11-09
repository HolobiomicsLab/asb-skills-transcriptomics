---
name: row-max-scaling-to-unit-interval
description: Use when raw gene/protein count matrices (TSV or CSV) must be transformed
  to the 0–1 range expected by a sigmoid-output VAE, or when reproducing the FAVA
  preprocessed_matrix.tsv input format.
when_to_use_negative:
- Data is already log2-normalized and 0–1 row-max scaled — re-applying log2 to log-scaled
  or compressed values is wrong
- Data are continuous omics values not on a count scale (e.g. already-normalized expression
  or proteomics intensities in a different range) where log2 + row-max scaling does
  not match the intended VAE input distribution
- The target model does not use a sigmoid-bounded output layer — row-max scaling to
  [0,1] only makes sense when outputs are 0–1 scaled
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3308
tools:
- name: FAVA (favapy)
  role: Downstream consumer of the preprocessed_matrix.tsv produced by this scaling
    step; its VAE's sigmoid output layer motivates the [0,1] row-max scaling
  repo: https://github.com/mikelkou/fava
- name: favapy
  role: Command-line entry point (favapy <path-to-data-file> <path-to-save-output>)
    that runs the FAVA VAE on the scaled matrix
  repo: https://github.com/mikelkou/fava
- name: Python
  role: Implementation language; pandas loads the TSV/CSV count matrix for log2 +
    row-max scaling
- name: AnnData
  role: Input source data structure used by FAVA within the scverse ecosystem
  repo: https://github.com/scverse/ecosystem-packages/tree/main/packages/favapy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:43e3a78229e9d97cb895de3aebb36414b61e5996575738f8dd7aada130cea706
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - AnnData
  - FAVA (favapy)
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# row-max-scaling-to-unit-interval

## Summary

Reconstruct FAVA's count pre-processing stage: log2-normalize the raw count matrix, then divide each row's values by that row's maximum so every value lands in [0,1], matching the sigmoid (0–1) output layer of the FAVA variational autoencoder.

## When to use

Use when raw gene/protein count matrices (TSV or CSV) must be transformed to the 0–1 range expected by a sigmoid-output VAE, or when reproducing the FAVA preprocessed_matrix.tsv input format.

## When NOT to use

- Data is already log2-normalized and 0–1 row-max scaled — re-applying log2 to log-scaled or compressed values is wrong
- Data are continuous omics values not on a count scale (e.g. already-normalized expression or proteomics intensities in a different range) where log2 + row-max scaling does not match the intended VAE input distribution
- The target model does not use a sigmoid-bounded output layer — row-max scaling to [0,1] only makes sense when outputs are 0–1 scaled

## Inputs

- Raw count matrix in TSV (default) or CSV format
- AnnData object (FAVA's scverse-ecosystem input source)

## Outputs

- Log2-normalized, row-max-scaled matrix with all values in [0,1]
- preprocessed_matrix.tsv — the command-line data file consumed by favapy

## How to apply

Load the raw count matrix with pandas, accepting TSV (default) or CSV. Apply log2 normalization to the count values. Then scale each value into [0,1] by dividing it by the maximum of its row — this row-max scaling matches the 0–1 scaled sigmoid output activation of the FAVA VAE, so the network input and output ranges are consistent. Write the result to preprocessed_matrix.tsv, the data file FAVA consumes on the command line (favapy <path-to-data-file> <path-to-save-output>).

## Related tools

- **FAVA (favapy)** (Downstream consumer of the preprocessed_matrix.tsv produced by this scaling step; its VAE's sigmoid output layer motivates the [0,1] row-max scaling) — https://github.com/mikelkou/fava
- **favapy** (Command-line entry point (favapy <path-to-data-file> <path-to-save-output>) that runs the FAVA VAE on the scaled matrix) — https://github.com/mikelkou/fava
- **Python** (Implementation language; pandas loads the TSV/CSV count matrix for log2 + row-max scaling)
- **AnnData** (Input source data structure used by FAVA within the scverse ecosystem) — https://github.com/scverse/ecosystem-packages/tree/main/packages/favapy

## Examples

```
favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- All values in the output matrix fall within [0,1], with at least one value equal to 1.0 per row (the row maximum)
- Each row's max equals the max of the log2-normalized values for that row (scaling is per-row, not global)
- Applying log2 then row-max scaling is idempotent in range: min ≥ 0 and max ≤ 1 after the transform
- Output file is named/formatted as preprocessed_matrix.tsv and is accepted by favapy on the command line without format errors
- Log2 normalization precedes row-max scaling — values scaled before log2 will not match FAVA's expected input distribution

## Limitations

- Scaling ties each row to its maximum, so a single very large count in a row dominates that row's scale
- Row-max scaling assumes the sigmoid output layer is 0–1 scaled; if the VAE architecture changes, the scaling target range must change too
- FAVA handles TSV (default) and CSV; other formats need conversion before this step
- The article documents this as a pre-processing layer inside FAVA; standalone use outside the FAVA VAE pipeline is not validated by the paper

## Evidence

- [methods] In FAVA's pre-processing layer, after log2 normalization of the count matrices, values are normalized to the [0,1] range by dividing each value by the maximum of the row it belongs to, matching the 0-1 scaled sigmoid output layer of the VAE.: [evidence span withheld — anchor status: not_found]
- [methods] FAVA workflow step 1: [evidence span withheld — anchor status: not_found]
- [methods] FAVA workflow step 4: [evidence span withheld — anchor status: not_found]
- [readme] README command-line usage: "Run FAVA from the command line as follows:

.. code-block:: bash

   favapy <path-to-data-file> <path-to-save-output>"
- [abstract] AnnData as input: "FAVA functions within the scverse ecosystem, employing AnnData as its input source"
