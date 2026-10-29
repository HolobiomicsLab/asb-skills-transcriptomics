---
name: atac-peak-matrix-preprocessing
description: Use when raw scATAC-seq peak-by-cell count matrices (AnnData) must be
  normalized and structured (peaks sorted per chromosome, intra-chromosomal pruning)
  before feeding scButterfly's ATAC encoder with 30% input masking.
when_to_use_negative:
- Input is scRNA-seq gene count data rather than an ATAC peak matrix — use data_processing.RNA_data_preprocessing
  instead
- The ATAC matrix is already TF-IDF/CLR normalized and chromosome-sorted with connections
  pruned; re-running default preprocessing may double-transform
- Translation involves modalities other than scRNA-seq/scATAC-seq under the paired
  basic model, e.g. scADT-seq, where ATAC-specific peak pruning does not apply
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0769
tools:
- name: scButterfly
  role: Provides data_preprocessing and data_processing.ATAC_data_preprocessing APIs
    that load, TF-IDF-normalize, and prune the ATAC peak matrix before model construction
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python 3.9
  role: Runtime environment for the scButterfly package and AnnData/scanpy single-cell
    data containers
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:48be3d0e2ae14a49cb94638e75dc2e712e5c38f32bfe87757807da0622bb0754
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - PyTorch
  - Python
  - Python 3.9
  - scButterfly
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# atac-peak-matrix-preprocessing

## Summary

Preprocess raw scATAC-seq peak count matrices for scButterfly cross-modality translation: TF-IDF-based normalization with intra-chromosomal connection pruning and inverse TF-IDF/CLR-style transformations handled inside Butterfly.data_preprocessing / data_processing.ATAC_data_preprocessing so peak profiles are denoised and comparable to translated RNA profiles.

## When to use

Use when raw scATAC-seq peak-by-cell count matrices (AnnData) must be normalized and structured (peaks sorted per chromosome, intra-chromosomal pruning) before feeding scButterfly's ATAC encoder with 30% input masking.

## When NOT to use

- Input is scRNA-seq gene count data rather than an ATAC peak matrix — use data_processing.RNA_data_preprocessing instead
- The ATAC matrix is already TF-IDF/CLR normalized and chromosome-sorted with connections pruned; re-running default preprocessing may double-transform
- Translation involves modalities other than scRNA-seq/scATAC-seq under the paired basic model, e.g. scADT-seq, where ATAC-specific peak pruning does not apply

## Inputs

- Raw scATAC-seq peak-by-cell count matrix as an AnnData object (n_obs cells × n_vars peaks)
- Train/test cell ID lists (train_id, test_id) and optional validation_id
- chrom_list: a list of peaks count for each chromosome, with peaks sorted by chromosome

## Outputs

- Normalized, connection-pruned ATAC peak matrix ready for the scButterfly ATAC encoder (30% input masking)
- Per-chromosome peak count list (chrom_list) for construct_model
- Optionally saved processed data files and preprocessing logs (save_data/file_path/logging_path)

## How to apply

Load the raw scATAC-seq peak count matrix as an AnnData object with butterfly.load_data, passing train/test (and optional validation) cell IDs. Run butterfly.data_preprocessing, which internally calls data_processing.ATAC_data_preprocessing: it normalizes the peak matrix (including TF-IDF handling via inverse_TFIDF and CLR_transform variants) and prunes inter-chromosomal connections for the encoder and decoder of scATAC-seq data to reduce noise and parameters. Sort peaks by chromosome and build the chrom_list of per-chromosome peak counts, since construct_model(chrom_list) requires chromosome-sorted peaks. Keep default preprocessing settings — the README states the defaults give the best model results — and optionally save processed data or logs via save_data/file_path/logging_path. The processed peak matrix then feeds the ATAC variational encoder with 30% input masking during pretraining and the dual-aligned translation phase.

## Related tools

- **scButterfly** (Provides data_preprocessing and data_processing.ATAC_data_preprocessing APIs that load, TF-IDF-normalize, and prune the ATAC peak matrix before model construction) — https://github.com/BioX-NKU/scButterfly
- **Python 3.9** (Runtime environment for the scButterfly package and AnnData/scanpy single-cell data containers)

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id); butterfly.data_preprocessing(); butterfly.construct_model(chrom_list)
```

## Evaluation signals

- Peaks are sorted by chromosome and chrom_list correctly lists per-chromosome peak counts so construct_model runs without errors
- butterfly.data_preprocessing completes end-to-end and, if save_data=True, writes the processed matrix to file_path
- The preprocessed ATAC matrix has no inter-chromosomal connections remaining in the encoder/decoder structure (connection pruning applied)
- Subsequent pretraining with 30% ATAC input masking and Sigmoid decoder output converges, and test_model yields translated R2A profiles whose cell-type structure is assessable with calculate_cluster_index

## Limitations

- README strongly recommends default preprocessing settings; deviating parameters risk worse model results
- chrom_list requires peaks sorted by chromosome — unsorted peak matrices will misbuild the pruned ATAC network
- The current method does not incorporate prior knowledge of gene regulatory mechanisms or spatial/Hi-C modalities; cross-cell-type and cross-organ translation performance is flagged as needing improvement

## Evidence

- [methods] ATAC preprocessing prunes inter-chromosomal connections: [evidence span withheld — anchor status: not_found]
- [readme] Load raw count matrix before preprocessing: [evidence span withheld — anchor status: external_doc]
- [readme] chrom_list must be chromosome-sorted peaks: [evidence span withheld — anchor status: external_doc]
- [readme] Default preprocessing settings recommended: [evidence span withheld — anchor status: external_doc]
- [intro] Dropout noise requires tailored denoising: "dropout events due to the loss of DNA material during library preparation require tailored denoising approaches for the large number of false zeros"
