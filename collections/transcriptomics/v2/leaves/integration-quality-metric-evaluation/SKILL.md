---
name: integration-quality-metric-evaluation
description: Use when you have latent embeddings from integrating scATAC-seq and scRNA-seq
  (e.g. uniPort coupled-VAE + Minibatch-UOT output) and must quantify alignment via
  Silhouette, FOSCTTM, and ARI/NMI/F1.
when_to_use_negative:
- Input is raw unnormalized count matrices rather than trained latent embeddings —
  metrics must be computed on post-uniport.Run embeddings
- The datasets share no meaningful cell-type annotation for the reference modality,
  making kNN-based ARI/NMI/F1 label transfer inapplicable
- Storing a dense global OT plan (save_OT=True) when memory capacity is insufficient
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: uniPort
  role: Provides uniport.metrics.silhouette and uniport.metrics.label_transfer functions
    for computing Silhouette, ARI, NMI, and F1 on integration embeddings
  repo: https://github.com/caokai1073/uniport
- name: scikit-learn
  role: KNeighborsClassifier trains the kNN classifier on reference cell-type annotations
    and UMAP coordinates for label-transfer metric evaluation
- name: scanpy
  role: Preprocessing (normalize_total, log1p with offset 1) of scRNA data upstream
    of embedding and metric computation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:a761f025385510ce0bfbf803253b237fc356ea49eaf8a1aa80e243bebc6af4ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/caokai1073/uniport
  tools:
  - scanpy
  - scikit-learn
  - uniPort
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# integration-quality-metric-evaluation

## Summary

Evaluates single-cell multi-omics integration quality by computing Silhouette coefficient, average FOSCTTM, and ARI/NMI/F1 label-transfer scores on uniPort latent embeddings, benchmarked against baselines like GLUE on the 10x paired PBMC scATAC/scRNA dataset.

## When to use

Use when you have latent embeddings from integrating scATAC-seq and scRNA-seq (e.g. uniPort coupled-VAE + Minibatch-UOT output) and must quantify alignment via Silhouette, FOSCTTM, and ARI/NMI/F1.

## When NOT to use

- Input is raw unnormalized count matrices rather than trained latent embeddings — metrics must be computed on post-uniport.Run embeddings
- The datasets share no meaningful cell-type annotation for the reference modality, making kNN-based ARI/NMI/F1 label transfer inapplicable
- Storing a dense global OT plan (save_OT=True) when memory capacity is insufficient

## Inputs

- AnnData object(s) with uniPort latent embeddings in adata.obs['latent'] from up.Run on paired PBMC scATAC/scRNA data
- Cell-type annotations for the reference modality (for kNN label transfer)
- Baseline integration scores for comparison (e.g. GLUE FOSCTTM 0.0441, total 2.514)

## Outputs

- Silhouette coefficient (expected ~0.64 on paired PBMC data)
- Average FOSCTTM (expected ~0.0694 for uniPort, 0.0441 for GLUE)
- ARI, NMI, and F1 label-transfer scores (A+NMI+F1 total expected ~2.321 for uniPort, 2.514 for GLUE)
- Comparison table exported as pbmc_integration_scores.csv

## How to apply

After running uniport.Run on preprocessed AnnData objects (scanpy normalize_total + log1p with offset 1 for scRNA; TFIDF_LSI for scATAC; k=2000 highly variable common genes plus kx=ky=2000 dataset-specific HVGs), take the latent embeddings stored in adata.obs['latent']. Compute the Silhouette coefficient with uniport.metrics.silhouette, and perform label transfer by training a kNN classifier (sklearn.neighbors.KNeighborsClassifier) on reference cell-type annotations to derive ARI, NMI, and F1 via uniport.metrics.label_transfer. Compute average FOSCTTM as the mean fraction of cross-modality nearest neighbors that map to true paired cells. Assemble all metrics into a comparison table alongside baseline scores (GLUE: FOSCTTM 0.0441, total 2.514) and export as CSV. Correct application is judged by reproducing the published uniPort values (Silhouette 0.64, average FOSCTTM 0.0694, ARI+NMI+F1 total 2.321) within a small tolerance.

## Related tools

- **uniPort** (Provides uniport.metrics.silhouette and uniport.metrics.label_transfer functions for computing Silhouette, ARI, NMI, and F1 on integration embeddings) — https://github.com/caokai1073/uniport
- **scikit-learn** (KNeighborsClassifier trains the kNN classifier on reference cell-type annotations and UMAP coordinates for label-transfer metric evaluation)
- **scanpy** (Preprocessing (normalize_total, log1p with offset 1) of scRNA data upstream of embedding and metric computation)

## Examples

```
adata = up.Run(adatas=[adata_atac, adata_rna], adata_cm=adata_cm, lambda_ot=1.0, lambda_kl=0.5, lambda_s=0.5); sil = uniport.metrics.silhouette(adata)
```

## Evaluation signals

- Reproduced uniPort metrics on paired PBMC fall within a small tolerance of published values: Silhouette 0.64, average FOSCTTM 0.0694, ARI+NMI+F1 total 2.321
- Comparison table contains both uniPort and GLUE baseline rows (GLUE FOSCTTM 0.0441, total 2.514) and is written as pbmc_integration_scores.csv
- kNN label transfer uses sklearn.neighbors.KNeighborsClassifier fit on reference cell-type annotations, matching the paper's method
- FOSCTTM is computed as an average across cells from cross-modality nearest-neighbor fractions, not a single direction
- Silhouette coefficient for uniPort exceeds GLUE on this dataset (0.64, highest among methods) even though FOSCTTM is second best

## Limitations

- Which hyperparameter combination from the searched grids (lambda_ot in {0.5,1.0,5.0}, lambda_kl/gamma in {0.5,1.0}, lambda_s in {0.5,1.0}) was selected for each dataset is not stated, so exact reproduction may require grid search
- uniPort's scATAC integration is based on gene activity scores rather than directly modeling regulatory signals, unlike GLUE, which can affect metric comparisons
- Storing a dense full OT plan T requires substantial memory; computing capacity should be ascertained before using save_OT

## Evidence

- [methods] kNN classifier for label transfer metrics: "we trained a k-Nearest-Neighbor (kNN) classifier by the sklearn.neighbors.KNeighborsClassifier function based on cell-type annotations and UMAP coordinates"
- [discussion] Published PBMC benchmark scores: [evidence span withheld — anchor status: not_found]
- [discussion] HVG selection for embedding input: "We identified k = 2000 highly variable common genes across cells of all datasets, and identified kx = ky = 2000 highly variable genes for each dataset"
- [readme] Training budget default: "iteration**: max iterations for training. Training one batch_size samples is one iteration. Default: 30000"
