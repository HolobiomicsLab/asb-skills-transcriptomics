---
name: tpfp-ratio-evaluation-of-network-predictions
description: Use when you have ranked protein interaction lists (e.g., from FAVA,
  scLink, hdWGCNA, PCC) on scRNA-seq datasets and need a comparable accuracy metric
  against curated references such as KEGG or BioGRID.
when_to_use_negative:
- When the compared ranked lists are truncated to different lengths or count protein
  pairs with different direction conventions, since TP/FP ratios would not be comparable
- When the goal is network inference itself (VAE latent-space compression, pair ranking)
  rather than post-hoc accuracy assessment of already-ranked predictions
- When input is bulk omics data without single-cell sparseness/redundancy, the scRNA-seq-specific
  benchmarking context of this skill does not apply
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: favapy
  role: Generates the FAVA ranked interaction list being evaluated; the -n option
    controls the number of output interactions and -c ranks pairs by latent-space
    Pearson correlation
  repo: https://github.com/mikelkou/fava
- name: scLink
  role: Baseline network-inference method whose top-10000 ranked list is benchmarked;
    expected to be unable to process the large PBMC dataset
- name: hdWGCNA
  role: Baseline co-expression network method benchmarked against KEGG and BioGRID
    on the same five datasets
- name: PCC
  role: Plain Pearson correlation baseline producing a comparable ranked interaction
    list for TP/FP benchmarking
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:c007aa2e414ad11f543a628cb3725d07f2b48272743248521a3bfc2fd858c4b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - FAVA
  - PCC
  - Python
  - favapy
  - hdWGCNA
  - scLink
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# TP/FP ratio evaluation of network predictions

## Summary

Benchmark inferred functional association networks by truncating each method's ranked interaction list to the top 10,000 pairs and counting true positives and false positives against KEGG Release 95 and BioGRID Release 4.4.217, summarized as TP/FP ratios per method, dataset, and reference.

## When to use

Use when you have ranked protein interaction lists (e.g., from FAVA, scLink, hdWGCNA, PCC) on scRNA-seq datasets and need a comparable accuracy metric against curated references such as KEGG or BioGRID.

## When NOT to use

- When the compared ranked lists are truncated to different lengths or count protein pairs with different direction conventions, since TP/FP ratios would not be comparable
- When the goal is network inference itself (VAE latent-space compression, pair ranking) rather than post-hoc accuracy assessment of already-ranked predictions
- When input is bulk omics data without single-cell sparseness/redundancy, the scRNA-seq-specific benchmarking context of this skill does not apply

## Inputs

- Ranked protein interaction lists (top 10,000) from FAVA (favapy), scLink, hdWGCNA, and plain Pearson correlation (PCC)
- scRNA-seq expression matrices for GBM, lung SCC, small PBMC, pancreatic tumor, and large PBMC datasets (tsv/csv)
- KEGG Release 95 reference interaction set
- BioGRID Release 4.4.217 reference interaction set

## Outputs

- True positive and false positive counts per method, dataset, and reference
- TP/FP ratio table (method x dataset x reference) written to benchmark_tpfp_ratios.csv

## How to apply

Run each network-inference method on the same scRNA-seq expression matrices (tsv/csv), ranking protein pairs by Pearson correlation score in FAVA's latent space (-c, 1 = high correlation, -1 = high anti-correlation). Truncate every method's ranked list to the top 10,000 interactions using the favapy -n option, remembering that favapy counts both directions (proteinA-proteinB and proteinB-proteinB) so this covers 5,000 unique pairs. Count true positives as top-listed pairs present in KEGG Release 95 or BioGRID Release 4.4.217 and all others as false positives, then compute the TP/FP ratio per method x dataset x reference. In the FAVA study this yielded, e.g., a small PBMC/KEGG ratio of 3.9 for FAVA (4588 TP / 1168 FP) versus 1.0 for scLink (1878/1860), and 0.55 (KEGG) and 0.04 (BioGRID) for FAVA on large PBMC. The ratio is comparable across methods only because all lists share the same truncation length and direction-counting convention.

## Related tools

- **favapy** (Generates the FAVA ranked interaction list being evaluated; the -n option controls the number of output interactions and -c ranks pairs by latent-space Pearson correlation) — https://github.com/mikelkou/fava
- **scLink** (Baseline network-inference method whose top-10000 ranked list is benchmarked; expected to be unable to process the large PBMC dataset)
- **hdWGCNA** (Baseline co-expression network method benchmarked against KEGG and BioGRID on the same five datasets)
- **PCC** (Plain Pearson correlation baseline producing a comparable ranked interaction list for TP/FP benchmarking)

## Examples

```
pip install favapy; favapy <path-to-data-file> <path-to-save-output> -c -n 10000
```

## Evaluation signals

- On small PBMC benchmarked against KEGG, FAVA must reproduce TP/FP = 4588/1168 = 3.9, roughly 3.9x the second-best scLink at 1878/1860 = 1.0
- On large PBMC, FAVA should yield TP/FP ratios of 0.55 (KEGG) and 0.04 (BioGRID), and scLink should be absent from the comparison
- Each method's ranked list is truncated to exactly 10,000 interactions before counting, with both directions (proteinA-proteinB and proteinB-proteinA) counted
- The final output is a complete method x dataset x reference table written to benchmark_tpfp_ratios.csv
- Reference versions must match KEGG Release 95 and BioGRID Release 4.4.217 for counts to be reproducible

## Limitations

- scLink cannot process the large PBMC dataset, leaving that comparison incomplete for this baseline
- On large PBMC, even FAVA's BioGRID TP/FP ratio is very low (0.04), so ratio interpretation depends heavily on the dataset and reference chosen
- Truncating to the top 10,000 interactions (counting both directions) discards lower-ranked predictions, so the metric reflects precision at a fixed depth only

## Evidence

- [abstract] FAVA infers networks with higher accuracy; benchmark context: "FAVA infers networks from high-dimensional omics data with much higher accuracy than existing methods, across a diverse collection of real as well as simulated datasets"
- [intro] Latent-space correlation is the basis of the ranked list being evaluated: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [methods] CLI entry point used to produce the ranked predictions: "Run FAVA from the command line as follows:

.. code-block:: bash

   favapy <path-to-data-file> <path-to-save-output>"
- [methods] Installation command for reproducing the tooling: "You can install FAVA using pip:

.. code-block:: bash

   pip install favapy"
