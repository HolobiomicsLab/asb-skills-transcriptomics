---
name: integration-metric-computation-and-composite-scoring
description: Use when you have integrated embeddings from multiple single-cell integration
  methods (e.g. scDREAMER, scVI, INSCT) and need quantitative batch-mixing, bio-conservation,
  and runtime comparisons across atlas-scale datasets.
when_to_use_negative:
- Embeddings have not yet been generated — this skill evaluates integrated representations,
  not raw count matrices
- Data contain a single batch with no batch effects to correct, so batch-correction
  metrics like PCR batch and kBET are meaningless
- Datasets with extremely large numbers of cells and batches (e.g. the healthy heart
  atlas) where kBET is computationally infeasible
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Scanpy
  role: Provides AnnData handling and the Louvain clustering used to compute clusters
    at the NMI-maximizing resolution for bio-conservation metrics
- name: scDREAMER
  role: Method under evaluation; its batch-mixed latent embeddings are scored against
    baselines
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: scVI
  role: Deep-learning baseline whose runtime (~154 s/epoch on the 1M-cell cross-species
    dataset) is compared against scDREAMER (~130 s/epoch)
- name: INSCT
  role: Deep-learning baseline whose runtime (~172 s/epoch on the 1M-cell cross-species
    dataset) is compared in the subsampled runtime benchmark
  repo: https://github.com/lkmklsmn/insct
- name: scDREAMER-reproducibility
  role: Repository containing the analysis notebooks and results for all metric computations
    presented in the manuscript
  repo: https://github.com/Zafar-Lab/scDREAMER-reproducibility
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:c2fa3ffcbe9de26deaa0593ce50e32116eeec9bdac3acadb02497779aacee7ad
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
  - scDREAMER-reproducibility
  - scGEN
  - scVI
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# integration-metric-computation-and-composite-scoring

## Summary

Computes batch-correction and bio-conservation metrics (NMI, ARI, cell-type/batch ASW, PCR batch, graph connectivity, kBET with k=50 kNN graphs) on integrated single-cell embeddings, plus per-epoch runtimes, then min-max normalizes them into composite scores for method comparison.

## When to use

Use when you have integrated embeddings from multiple single-cell integration methods (e.g. scDREAMER, scVI, INSCT) and need quantitative batch-mixing, bio-conservation, and runtime comparisons across atlas-scale datasets.

## When NOT to use

- Embeddings have not yet been generated — this skill evaluates integrated representations, not raw count matrices
- Data contain a single batch with no batch effects to correct, so batch-correction metrics like PCR batch and kBET are meaningless
- Datasets with extremely large numbers of cells and batches (e.g. the healthy heart atlas) where kBET is computationally infeasible

## Inputs

- Integrated latent embeddings (AnnData/.h5ad) from each integration method (scDREAMER, scVI, DESC, iMAP, scANVI, scGEN, scDML, INSCT)
- Cell-type annotation labels per cell for NMI/ARI/cell-type ASW
- Batch labels per cell for batch ASW, PCR batch, kBET
- Runtime measurements in seconds per epoch or per run at 10k/100k/500k/1M cell subsamples

## Outputs

- Per-method metric table: NMI, ARI, cell-type ASW, batch ASW, PCR batch, graph connectivity, kBET (k=50)
- Composite bio-conservation score, composite batch-correction score, and combined composite score per method (min-max normalized to [0, 1])
- Runtime comparison table covering all down-sampled sizes (e.g. integration_metrics_1M_cross_species.csv)

## How to apply

For each method's integrated embedding, run Louvain clustering at the resolution that maximizes NMI against cell-type labels, then compute NMI, ARI, cell-type ASW, batch ASW, PCR batch, graph connectivity, and kBET using k=50 nearest-neighbor graphs. Measure runtime in seconds per method on datasets subsampled to 10k, 100k, 500k, and 1M cells. Compute the composite bio-conservation score, composite batch-correction score, and combined composite score by min-max normalizing each metric across all competing methods so every scaled value lies in [0, 1]. Write all per-method metric scores and runtime values to a summary table (e.g. integration_metrics_1M_cross_species.csv).

## Related tools

- **Scanpy** (Provides AnnData handling and the Louvain clustering used to compute clusters at the NMI-maximizing resolution for bio-conservation metrics)
- **scDREAMER** (Method under evaluation; its batch-mixed latent embeddings are scored against baselines) — https://github.com/Zafar-Lab/scDREAMER
- **scVI** (Deep-learning baseline whose runtime (~154 s/epoch on the 1M-cell cross-species dataset) is compared against scDREAMER (~130 s/epoch))
- **INSCT** (Deep-learning baseline whose runtime (~172 s/epoch on the 1M-cell cross-species dataset) is compared in the subsampled runtime benchmark) — https://github.com/lkmklsmn/insct
- **scDREAMER-reproducibility** (Repository containing the analysis notebooks and results for all metric computations presented in the manuscript) — https://github.com/Zafar-Lab/scDREAMER-reproducibility

## Examples

```
sc.tl.louvain(adata, resolution=res_best, key_added='louvain'); nmi = normalized_mutual_info_score(adata.obs['celltype'], adata.obs['louvain']); ari = adjusted_rand_score(adata.obs['celltype'], adata.obs['louvain'])
```

## Evaluation signals

- After min-max normalization, every scaled metric value lies in [0, 1]
- The runtime table covers all down-sampled sizes (10k, 100k, 500k, 1M cells) for each competing method
- Louvain clustering resolution is chosen as the value that maximizes NMI, not an arbitrary default
- kBET is computed with k=50 nearest-neighbor graphs on the kNN graph of the integrated embedding
- Composite scores reproduce scDREAMER's reported runtime advantage (~130 s/epoch vs ~154 s for scVI and ~172 s for INSCT on one Nvidia Quadro RTX 5000 GPU)

## Limitations

- For the healthy heart atlas dataset, kBET could not be computed for some methods due to computational limitations in handling large numbers of cells and batches
- Composite scores depend on min-max normalization across the set of competing methods, so values are relative to the methods included in the comparison
- Runtime comparisons are hardware-dependent (measured on a single Nvidia Quadro RTX 5000 GPU) and apply to the deep-learning baselines compared

## Evidence

- [methods] Louvain resolution maximizes NMI: "We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value."
- [results] Runtime benchmark design: "we measured the runtime of scDREAMER on different datasets subsampled from the atlas-integration dataset and compared the runtime against that of scVI and INSCT"
- [discussion] kBET computational limitation: "for the healthy heart atlas dataset, kBET could not be computed for some of the methods due to computational limitations in handling large number of cells and batches"
- [supplementary] Reproducibility materials location: "All analysis and results presented in the manuscript are available at https://github.com/Zafar-Lab/scDREAMER-reproducibility"
