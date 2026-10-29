---
name: hyperparameter-sensitivity-analysis
description: Use when a pipeline contains a tunable clustering/embedding parameter
  (e.g. Leiden resolution) whose default needs empirical justification across multiple
  benchmark datasets via ARI/NMI sweeps.
when_to_use_negative:
- The goal is tuning a generative-model training hyperparameter (e.g. learning rate
  or epochs), not a downstream clustering parameter — this skill only varies the clustering
  resolution.
- No ground-truth reference labels exist, so ARI/NMI cannot be computed and the sweep
  cannot be scored.
- The analysis targets a single dataset only; the scME validation relies on consistency
  across all four benchmark datasets.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scanpy
  role: Provides the Leiden clustering implementation used to derive modality-specific
    labels at each resolution
- name: scikit-learn
  role: Computes ARI and NMI between Leiden labels and reference labels
- name: scME
  role: Produces the joint cell embeddings (zm) that the resolution sweep is applied
    to
  repo: https://github.com/bucky527/scME
- name: pandas
  role: Aggregates ARI/NMI scores across resolutions and datasets into ari_nmi_resolution_sweep.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad337
    title: 'scME: a dual-modality factor model for single-cell multiomics embedding'
  source_content_hash: sha256:8ca0e18927f3cc15d4ac665ba35296864ae616a420ed7049792df3c77535f882
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bucky527/scME
  tools:
  - PyTorch
  - anndata
  - pandas
  - scME
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# hyperparameter-sensitivity-analysis

## Summary

Sweeps a clustering hyperparameter (Leiden resolution) over a defined range and measures its effect on downstream performance (ARI/NMI) to justify a default parameter choice, as done in scME to show resolution 1 is optimal.

## When to use

Use when a pipeline contains a tunable clustering/embedding parameter (e.g. Leiden resolution) whose default needs empirical justification across multiple benchmark datasets via ARI/NMI sweeps.

## When NOT to use

- The goal is tuning a generative-model training hyperparameter (e.g. learning rate or epochs), not a downstream clustering parameter — this skill only varies the clustering resolution.
- No ground-truth reference labels exist, so ARI/NMI cannot be computed and the sweep cannot be scored.
- The analysis targets a single dataset only; the scME validation relies on consistency across all four benchmark datasets.

## Inputs

- Joint cell embeddings (zm) from scME inference on RNA gene counts and raw protein ADT counts CSVs
- Reference cell-type labels for each CITE-seq dataset (CBMC, BMNC, BM2, SLN)
- Sweep range of Leiden resolution values (0.1 to 5)

## Outputs

- Leiden cluster labels per resolution value per dataset
- ARI and NMI scores per resolution per dataset
- Aggregated results table (ari_nmi_resolution_sweep.csv) supporting a default resolution choice (resolution 1)

## How to apply

Fix all model parameters (here: scME trained per dataset with protein_dist='NB', rna_latent_dim=24, protein_latent_dim=20, latent_dim=32, max_epochs=200, batch_size=256, lr=1e-4) and vary only the target hyperparameter. For each Leiden resolution value from 0.1 to 5, derive modality-specific cluster labels from the joint cell embeddings (zm from scme.inference) using Leiden clustering implemented in scanpy. Compute ARI and NMI between each resolution's labels and the reference cell-type labels with scikit-learn metrics, then aggregate all scores per dataset (CBMC, BMNC, BM2, SLN) into a single table such as ari_nmi_resolution_sweep.csv. Select the resolution with consistently highest ARI/NMI across all datasets; in scME this validated resolution 1, reproducing the article's Fig. 5 (Section 3.4) result.

## Related tools

- **scanpy** (Provides the Leiden clustering implementation used to derive modality-specific labels at each resolution)
- **scikit-learn** (Computes ARI and NMI between Leiden labels and reference labels)
- **scME** (Produces the joint cell embeddings (zm) that the resolution sweep is applied to) — https://github.com/bucky527/scME
- **pandas** (Aggregates ARI/NMI scores across resolutions and datasets into ari_nmi_resolution_sweep.csv)

## Examples

```
zm = scme.inference(rna_data, protein_data); for res in [0.1,1,5]: labels = sc.tl.leiden(adata, resolution=res, key_added=f'leiden_{res}')
```

## Evaluation signals

- ARI and NMI at Leiden resolution 1 are the highest across the full 0.1–5 sweep on all four datasets (CBMC, BMNC, BM2, SLN)
- The aggregated ari_nmi_resolution_sweep.csv contains one ARI/NMI row per (dataset, resolution) pair with no missing values
- Monotonic or near-monotonic score degradation away from the optimum indicates a stable sweep (no noisy isolated peaks)
- Reference labels and cluster labels have matching cell counts and ordering (cells as rows)

## Limitations

- The sweep is validated only on the four CITE-seq benchmark datasets (CBMC, BMNC, BM2, SLN); the optimal resolution may differ on other data.
- Leiden clustering is stochastic unless a random seed is fixed, so scores can vary between runs.
- The article evaluates resolution in the 0.1–5 range; behavior outside this range was not characterized.

## Evidence

- [results] Across all four benchmark datasets (CBMC, BMNC, BM2, SLN), sweeping the Leiden resolution from 0.1 to 5 showed that a resolution of 1 consistently generated the best clustering performance as measured by ARI and NMI.: [evidence span withheld — anchor status: not_found]
- [readme] scME input format: "scME accepts as input the  RNA gene counts matrix data and raw protein ADTs counts matrix data in the CSV format"
- [readme] Embedding inference: "#Inference cell embedding

zm=scme.inference(rna_data,protein_data)"
- [readme] Model training parameters: "scme=train_model(model,
                max_epochs=200,
                batchsize=256,
                lr=1e-4,"
- [methods] Dependency stack including clustering/metric libraries: "pip3 install numpy scipy pandas scikit-learn pyro-ppl matplotlib scanpy anndata scvi-tools"
