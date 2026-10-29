---
name: integration-benchmark-metric-computation
description: Use when comparing multiple single-cell batch-integration methods (e.g.,
  scDREAMER vs. scVI, Harmony, Seurat v3) on the same AnnData embedding and you need
  quantitative, composite-scored benchmark metrics.
when_to_use_negative:
- You only have raw count matrices with no integrated embeddings — this skill evaluates
  integration results, it does not produce them
- You need a metric that requires full kBET on a very large, many-batch dataset (e.g.,
  the 147-batch heart atlas), where kBET computation fails due to memory/time limits
  and must be excluded
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: Scanpy
  role: Provides AnnData handling, preprocessing, and Louvain clustering used to generate
    cluster assignments for metric computation
- name: Louvain
  role: Cluster each integrated embedding at the resolution maximizing NMI; those
    clusters feed NMI, ARI, and ASW calculations
- name: scDREAMER
  role: Reference unsupervised integration method whose embedding is benchmarked against
    eleven competitors using the composite scores
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: scVI
  role: Competing deep variational-autoencoder integration method scored in the metric
    table
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:62757726b08d127edacebc0a2da2a62cd3e85a5a848fcc91092f1ef0f02eba8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - ADAM
  - BBKNN
  - Harmony
  - INSCT
  - LIGER
  - Louvain
  - Scanorama
  - Scanpy
  - Seurat
  - iMAP
  - scANVI
  - scDML
  - scDREAMER
  - scDREAMER-Sup
  - scGEN
  - scVI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 22
---

# integration-benchmark-metric-computation

## Summary

Computes single-cell integration benchmark metrics — biological conservation (NMI, ARI, cell-type ASW), batch correction (batch ASW, PCR batch, graph connectivity, kBET with k=50), and min-max normalized composite scores — to compare integration methods on a shared dataset.

## When to use

Use when comparing multiple single-cell batch-integration methods (e.g., scDREAMER vs. scVI, Harmony, Seurat v3) on the same AnnData embedding and you need quantitative, composite-scored benchmark metrics.

## When NOT to use

- You only have raw count matrices with no integrated embeddings — this skill evaluates integration results, it does not produce them
- You need a metric that requires full kBET on a very large, many-batch dataset (e.g., the 147-batch heart atlas), where kBET computation fails due to memory/time limits and must be excluded

## Inputs

- Integrated embedding (latent space) per method, e.g., scDREAMER, scVI, Scanorama, Harmony, Seurat v3, BBKNN, INSCT, LIGER, iMAP, scANVI, scGEN, scDML
- Preprocessed Scanpy AnnData object with batch and cell-type annotations (top 2000 HVGs, normalized and log-transformed counts)
- Louvain cluster assignments per embedding at the NMI-maximizing resolution

## Outputs

- Per-method metric table (e.g., heart_atlas_integration_metrics.csv or immune_integration_metrics.csv) with NMI, ARI, cell-type ASW, batch ASW, PCR batch, graph connectivity, kBET, and composite scores
- Min-max normalized composite bio-conservation, composite batch-correction, and combined composite scores per method

## How to apply

For each method's integrated embedding, run Louvain clustering at the resolution that maximizes NMI and reuse those cluster assignments across metrics. Compute biological conservation metrics (NMI, ARI, cell-type ASW scaled as (ASW+1)/2) and batch-correction metrics (batch ASW scaled as 1−abs(ASW), PCR batch, graph connectivity, and kBET with k=50 on the kNN graph; the immune task also adds isolated-label F1/ASW and graph iLISI/cLISI). Min-max normalize each metric across all competing methods, then average into composite bio-conservation, composite batch-correction, and combined composite scores. Where a metric fails computationally — e.g., kBET on the 147-batch healthy heart atlas — exclude it explicitly for that method rather than dropping the method. Finally, write the per-method metric table (e.g., heart_atlas_integration_metrics.csv) and verify the expected ranking against the paper's finding.

## Related tools

- **Scanpy** (Provides AnnData handling, preprocessing, and Louvain clustering used to generate cluster assignments for metric computation)
- **Louvain** (Cluster each integrated embedding at the resolution maximizing NMI; those clusters feed NMI, ARI, and ASW calculations)
- **scDREAMER** (Reference unsupervised integration method whose embedding is benchmarked against eleven competitors using the composite scores) — https://github.com/Zafar-Lab/scDREAMER
- **scVI** (Competing deep variational-autoencoder integration method scored in the metric table)

## Examples

```
sc.pp.neighbors(adata_integrated, use_rep='embedding'); sc.tl.louvain(adata_integrated, resolution=res_max_nmi); scores = minmax_scale(metrics).mean(axis=1)
```

## Evaluation signals

- Output table contains one row for each of the twelve methods with all metric and composite-score columns populated (or explicitly marked as excluded for kBET)
- Each metric is min-max normalized across all competing methods before averaging into composite scores
- Composite scores reproduce the paper's reported ranking, e.g., scDREAMER best unsupervised combined composite score on the heart atlas (25% improvement over second best)
- Louvain resolution is selected to maximize NMI and the same clusters are reused for other metrics
- kBET is run with k=50 on the kNN graph and failures are excluded per-method rather than silently zeroed

## Limitations

- kBET could not be computed for some methods on the healthy heart atlas dataset due to computational limitations in handling large number of cells and batches
- Louvain resolution search for max NMI adds a tuning step and cluster assignments are metric-specific choices reused across metrics
- Min-max normalization makes composite scores relative to the set of methods compared; adding or removing a method changes all scores

## Evidence

- [methods] Louvain clustering for NMI-maximizing resolution: "We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value."
- [methods] kBET exclusion on heart atlas: "for the healthy heart atlas dataset, kBET could not be computed for some of the methods due to computational limitations in handling large number of cells and batches"
- [results] scDREAMER best on heart atlas composite score: [evidence span withheld — anchor status: not_found]
- [other] Validation that table is complete and scDREAMER ranks first: [evidence span withheld — anchor status: not_found]
- [readme] README confirms scope: reproducibility of manuscript analyses: "All analysis and results presented in the manuscript are available at https://github.com/Zafar-Lab/scDREAMER-reproducibility"
