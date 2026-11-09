---
name: min-max-normalization-of-integration-metrics
description: Use when comparing multiple integration methods on shared benchmark metrics
  that have different scales/ranges, and composite scores (bio-conservation, batch-correction,
  combined) are needed to rank methods.
when_to_use_negative:
- Metrics with inverted direction (e.g. runtime, kBET acceptance) are normalized naively
  without first accounting for whether lower is better — direction must be handled
  before min-max scaling
- Only a single method is evaluated with no set of competing methods — min-max normalization
  requires a comparison pool across methods
- Downstream analysis needs raw metric magnitudes or statistical tests rather than
  method-ranking composite scores
tools:
- name: scDREAMER
  role: Method whose integration embedding is clustered and scored, then min-max normalized
    against competing methods
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: Scanpy
  role: Provides the Louvain clustering and preprocessing whose outputs (NMI, ARI,
    ASW, PCR batch, graph connectivity, kBET) feed the normalization step
- name: scVI
  role: Deep-learning baseline included in the metric pool that defines the min/max
    normalization range (~154 s/epoch on 1M cells)
- name: INSCT
  role: Deep-learning baseline included in the metric pool that defines the min/max
    normalization range (~172 s/epoch on 1M cells)
  repo: http://github.com/lkmklsmn/insct
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:1d0f3f43531732fc6e8f35a9e36cbac4a5f98a42f51552d0a8a402cb40754773
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - DESC
  - INSCT
  - Scanpy
  - iMAP
  - scANVI
  - scDML
  - scDREAMER
  - scGEN
  - scVI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# min-max normalization of integration metrics

## Summary

Converts heterogeneous single-cell integration metrics (NMI, ARI, cell-type ASW, batch ASW, PCR batch, graph connectivity, kBET, runtime) onto a common [0, 1] scale by min-max normalization across competing methods, enabling computation of composite bio-conservation, batch-correction, and combined scores for benchmarking methods like scDREAMER against scVI, INSCT, DESC, iMAP, scANVI, scGEN, and scDML.

## When to use

Use when comparing multiple integration methods on shared benchmark metrics that have different scales/ranges, and composite scores (bio-conservation, batch-correction, combined) are needed to rank methods.

## When NOT to use

- Metrics with inverted direction (e.g. runtime, kBET acceptance) are normalized naively without first accounting for whether lower is better — direction must be handled before min-max scaling
- Only a single method is evaluated with no set of competing methods — min-max normalization requires a comparison pool across methods
- Downstream analysis needs raw metric magnitudes or statistical tests rather than method-ranking composite scores

## Inputs

- Per-method raw metric table (NMI, ARI, cell-type ASW, batch ASW, PCR batch, graph connectivity, kBET k=50) from Louvain clustering of each method's integrated embedding
- Runtime measurements in seconds for each method on datasets subsampled to 10k, 100k, 500k, and 1M cells
- Supplementary Table 5 run configurations for deep-learning baselines (scVI, DESC, iMAP, scANVI, scGEN, scDML, INSCT)

## Outputs

- Min-max normalized metric matrix with every scaled value in [0, 1]
- Composite bio-conservation score, composite batch-correction score, and combined composite score per method
- integration_metrics_1M_cross_species.csv with per-method metric scores and runtime values across 10k/100k/500k/1M down-sampled sizes

## How to apply

After computing per-method raw metrics — NMI and ARI from Louvain clusters at the resolution maximizing NMI, cell-type ASW and batch ASW, PCR batch, graph connectivity, and kBET with k=50 kNN graphs — min-max normalize each metric across all competing methods so each scaled value lies in [0, 1]. Then average the normalized bio-conservation metrics into a composite bio-conservation score, the normalized batch-correction metrics into a composite batch-correction score, and combine them into a combined composite score. In the 1M-cell cross-species runtime task, apply the same normalization to runtime values (scDREAMER ~130 s/epoch vs scVI ~154 s/epoch vs INSCT ~172 s/epoch on one Nvidia Quadro RTX 5000 GPU) and write per-method normalized scores and runtime values to integration_metrics_1M_cross_species.csv.

## Related tools

- **scDREAMER** (Method whose integration embedding is clustered and scored, then min-max normalized against competing methods) — https://github.com/Zafar-Lab/scDREAMER
- **Scanpy** (Provides the Louvain clustering and preprocessing whose outputs (NMI, ARI, ASW, PCR batch, graph connectivity, kBET) feed the normalization step)
- **scVI** (Deep-learning baseline included in the metric pool that defines the min/max normalization range (~154 s/epoch on 1M cells))
- **INSCT** (Deep-learning baseline included in the metric pool that defines the min/max normalization range (~172 s/epoch on 1M cells)) — http://github.com/lkmklsmn/insct

## Examples

```
from sklearn.preprocessing import MinMaxScaler; normalized = MinMaxScaler().fit_transform(metrics_df[['NMI','ARI','ASW_celltype','ASW_batch','PCR_batch','graph_conn','kBET']])
```

## Evaluation signals

- Every scaled metric value lies in [0, 1] across all methods and metrics
- The metric with the maximum raw value maps to exactly 1 and the minimum maps to exactly 0 within each metric column
- The runtime table covers all down-sampled sizes (10k, 100k, 500k, 1M cells) for each method
- All per-method metric scores and runtime values are present in integration_metrics_1M_cross_species.csv
- Composite bio-conservation, batch-correction, and combined scores are computed from the normalized (not raw) metric values

## Limitations

- Min-max composite scores are tied to the specific pool of competing methods: adding or removing a baseline changes the min/max range and shifts every method's normalized score
- kBET could not be computed for some methods on the healthy heart atlas dataset due to computational limitations in handling large numbers of cells and batches, so the normalized metric pool may be incomplete
- scDREAMER being a deep learning-based method may require fine-tuning of some parameters for extracting the best performance, which affects the raw metrics entering normalization

## Evidence

- [other] Compute the composite bio-conservation score, composite batch-correction score, and combined composite score by min-max normalizing each metric across all competing methods.: [evidence span withheld — anchor status: not_found]
- [methods] We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value.: "We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value."
- [results] we measured the runtime of scDREAMER on different datasets subsampled from the atlas-integration dataset and compared the runtime against that of scVI and INSCT: "we measured the runtime of scDREAMER on different datasets subsampled from the atlas-integration dataset and compared the runtime against that of scVI and INSCT"
- [methods] for the healthy heart atlas dataset, kBET could not be computed for some of the methods due to computational limitations in handling large number of cells and batches: "for the healthy heart atlas dataset, kBET could not be computed for some of the methods due to computational limitations in handling large number of cells and batches"
