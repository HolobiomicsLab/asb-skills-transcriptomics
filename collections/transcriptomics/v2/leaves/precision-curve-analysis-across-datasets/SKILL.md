---
name: precision-curve-analysis-across-datasets
description: Use when you have ranked predicted interaction lists (e.g. top 100000
  pairs from a latent-space or correlation method) and a gold-standard set like KEGG,
  and need to quantify precision across both scRNA-seq and bulk proteomics data.
when_to_use_negative:
- The method output is unranked (e.g. raw VAE embeddings without computed protein
  pairs) — precision curves need an ordering to sweep a cut-off over
- There is no gold-standard reference (e.g. KEGG) to benchmark predictions against,
  since precision cannot be defined without known positives
- The goal is functional validation rather than benchmarking ranked predictions against
  curated pathway associations
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3308
tools:
- name: FAVA (favapy)
  role: Produces the ranked interaction network (top 100000 pairs) whose precision
    is benchmarked against KEGG; provides VAE, cook, and pairs_after_cutoff functions
    for the cut-off sweep
  repo: https://github.com/mikelkou/fava
- name: Python
  role: Used to compute the PCC baseline, sweep the Pearson cut-off, benchmark against
    KEGG, and plot precision curves
- name: AnnData
  role: Input data structure through which FAVA reads the scRNA-seq matrix for benchmarking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:cb00e2fdb633c6a1d02d659cfb752a14fa948c4ae27ada3963c713f594245a9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - AnnData
  - FAVA (favapy)
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# precision-curve-analysis-across-datasets

## Summary

Benchmark ranked protein-protein interaction predictions (e.g. from FAVA vs a Pearson correlation baseline) against KEGG functional associations by sweeping the top-ranked pairs and plotting precision curves, on multiple input datasets (HPA scRNA-seq and PRIDE proteomics).

## When to use

Use when you have ranked predicted interaction lists (e.g. top 100000 pairs from a latent-space or correlation method) and a gold-standard set like KEGG, and need to quantify precision across both scRNA-seq and bulk proteomics data.

## When NOT to use

- The method output is unranked (e.g. raw VAE embeddings without computed protein pairs) — precision curves need an ordering to sweep a cut-off over
- There is no gold-standard reference (e.g. KEGG) to benchmark predictions against, since precision cannot be defined without known positives
- The goal is functional validation rather than benchmarking ranked predictions against curated pathway associations

## Inputs

- Ranked protein-protein interaction lists (top 100000 pairs with both directions, proteinA-proteinB and proteinB-proteinA) from FAVA
- Ranked PCC interaction lists produced by sweeping the Pearson correlation cut-off from 1 to -1
- KEGG functional association pairs as gold-standard benchmark
- Human Protein Atlas scRNA-seq matrix (566109 cells, tsv/csv input)
- PRIDE bulk proteomics dataset (633 experiments, 18846 proteins)

## Outputs

- Precision curves (TP vs FP as cut-off varies) for FAVA and PCC on each dataset
- Counts of true positives and false positives at each ranking cut-off
- Precision summary (e.g. >90% precision, >5000 TP with <500 FP on HPA scRNA-seq)
- Wall-clock runtime comparison of FAVA vs PCC per dataset

## How to apply

Run each network-inference method on the same input matrix — for FAVA, run 'favapy <path-to-data-file> <path-to-save-output>' with defaults (epochs=50, batch size=32, auto-scaled hidden/latent dimensions), and compute a plain PCC baseline sweeping the Pearson cut-off from +1 (high correlation) to -1 (high anti-correlation). Rank the inferred pairs and benchmark both ranked lists against KEGG functional associations, using the favapy 'pairs_after_cutoff' step to select pairs at each cut-off. Plot precision curves (true positives vs false positives as the cut-off is relaxed) for each dataset — here the Human Protein Atlas scRNA-seq matrix (566109 cells) and the PRIDE bulk proteomics dataset (633 experiments, 18846 proteins). Also record wall-clock runtime per method per dataset so accuracy gains can be weighed against cost. Correct application is judged by checking that the scRNA-seq precision curve exceeds 5000 true positives with fewer than 500 false positives (>90% precision) and that FAVA outperforms PCC by a wide margin.

## Related tools

- **FAVA (favapy)** (Produces the ranked interaction network (top 100000 pairs) whose precision is benchmarked against KEGG; provides VAE, cook, and pairs_after_cutoff functions for the cut-off sweep) — https://github.com/mikelkou/fava
- **Python** (Used to compute the PCC baseline, sweep the Pearson cut-off, benchmark against KEGG, and plot precision curves)
- **AnnData** (Input data structure through which FAVA reads the scRNA-seq matrix for benchmarking)

## Examples

```
favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- HPA scRNA-seq precision curve shows more than 5000 true positives with fewer than 500 false positives (precision >90%) when benchmarked against KEGG
- FAVA precision curve lies above the PCC baseline curve on both HPA scRNA-seq and PRIDE proteomics datasets
- Both directions (proteinA-proteinB and proteinB-proteinA) are included in the evaluated top-100000 pair set
- Recorded runtime shows FAVA at least 9x faster than PCC (~1 day vs ~9 days) on both datasets

## Limitations

- Precision against KEGG is limited to interactions covered by KEGG; the gold standard itself is biased by data availability, so results partly reflect that bias
- Runtime benchmarking is hardware-dependent; the reported 9x speedup (1 day vs ~9 days) is specific to these dataset sizes (566109 cells and 633 experiments)
- PCC precision depends on the correlation cut-off direction: anti-correlation (cut-off toward -1) as well as correlation must be swept to fairly compare with FAVA

## Evidence

- [results] FAVA outperforms PCC with >90% precision on both HPA and PRIDE datasets: [evidence span withheld — anchor status: not_found]
- [abstract] FAVA benchmarked against KEGG outperforms PCC: "FAVA infers networks from high-dimensional omics data with much higher accuracy than existing methods, across a diverse collection of real as well as simulated datasets"
- [methods] Command-line invocation of favapy used to generate ranked networks: "Run FAVA from the command line as follows:

.. code-block:: bash

   favapy <path-to-data-file> <path-to-save-output>"
- [intro] Latent-space correlations produce improved networks that are then benchmarked: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
