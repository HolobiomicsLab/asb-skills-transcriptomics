---
name: spatial-transcriptomics-gene-imputation
description: Use when a low-plex spatial assay (e.g., 153-gene MERFISH hypothalamus
  data) needs prediction of unmeasured genes from a matched scRNA-seq reference across
  random training/test gene splits.
when_to_use_negative:
- When the spatial and scRNA datasets profile the same genes and require only joint
  embedding rather than prediction of unmeasured genes — plain integration (uniPort
  out='latent') suffices.
- When the spatial and scRNA data come from the same cells (vertical integration),
  the gene-imputation benchmark design with random gene splits is not the appropriate
  evaluation.
- When no matched scRNA-seq reference exists for the spatial tissue — imputation from
  scRNA cannot be performed.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: uniPort
  role: Trains the coupled-VAE with minibatch unbalanced optimal transport (uniport.Run)
    and predicts held-out gene expression through the decoder (out='predict'); also
    provides load_file, filter_data, and batch_scale preprocessing
  repo: https://github.com/caokai1073/uniport
- name: scanpy
  role: Normalizes total counts per cell (sc.pp.normalize_total) and log-normalizes
    (sc.pp.log1p) both MERFISH and scRNA data before scaling
- name: scikit-learn
  role: MaxAbsScaler scales each gene's values to the 0-1 range within each dataset
- name: gimVI
  role: Baseline method run on the identical 12 gene splits for imputation-accuracy
    comparison
- name: Tangram
  role: Baseline method run on the identical 12 gene splits for imputation-accuracy
    comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:ac97a73d626646971bf8bfa8cf7f9118951ef16f60ecbbe9e31e96a980145a79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/caokai1073/uniport
  tools:
  - Tangram
  - gimVI
  - scanpy
  - scikit-learn
  - uniPort
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# spatial-transcriptomics-gene-imputation

## Summary

Impute held-out gene expression in spatial transcriptomics data (e.g., MERFISH) by training a uniPort coupled-VAE with minibatch unbalanced optimal transport on a paired scRNA-seq reference, and benchmark imputation accuracy against gimVI and Tangram.

## When to use

Use when a low-plex spatial assay (e.g., 153-gene MERFISH hypothalamus data) needs prediction of unmeasured genes from a matched scRNA-seq reference across random training/test gene splits.

## When NOT to use

- When the spatial and scRNA datasets profile the same genes and require only joint embedding rather than prediction of unmeasured genes — plain integration (uniPort out='latent') suffices.
- When the spatial and scRNA data come from the same cells (vertical integration), the gene-imputation benchmark design with random gene splits is not the appropriate evaluation.
- When no matched scRNA-seq reference exists for the spatial tissue — imputation from scRNA cannot be performed.

## Inputs

- MERFISH hypothalamus spatial dataset and cell annotations (Dryad doi:10.5061/dryad.8t8s248)
- Mouse hypothalamus scRNA-seq reference (GEO GSE113576)
- Gene split definitions: 122 training genes and 31 held-out test genes per split

## Outputs

- Imputed expression of 31 held-out MERFISH genes per spatial spot via the trained decoder
- Per-split imputation metrics (mSCC, aSCC, mPCC, aPCC) for uniPort, gimVI, and Tangram
- Aggregated metrics table merfish_imputation_metrics.csv

## How to apply

Download the MERFISH hypothalamus data from Dryad (doi:10.5061/dryad.8t8s248) and the mouse hypothalamus scRNA-seq reference (GSE113576), loading both with uniport.load_file. Remove cells labeled 'Ambiguous' or 'Unstable', preprocess with scanpy (normalize_total, log1p), and scale each gene to 0-1 with uniport.filter_data/uniport.batch_scale using MaxAbsScaler. For each of 12 random splits, use 80% of the 153 MERFISH genes (122) as training genes and hold out 20% (31) for testing; train the coupled-VAE with uniport.Run (batch_size 512, iteration 30000, lambda_ot 1.0, lambda_s 0.5, out='predict') and predict held-out gene expression in spatial spots via the trained decoder. Compute mean and averaged Spearman (mSCC, aSCC) and Pearson (mPCC, aPCC) correlations between imputed and measured spatial expression on the held-out genes, repeat identically for gimVI and Tangram on the same splits, and aggregate into a metrics table.

## Related tools

- **uniPort** (Trains the coupled-VAE with minibatch unbalanced optimal transport (uniport.Run) and predicts held-out gene expression through the decoder (out='predict'); also provides load_file, filter_data, and batch_scale preprocessing) — https://github.com/caokai1073/uniport
- **scanpy** (Normalizes total counts per cell (sc.pp.normalize_total) and log-normalizes (sc.pp.log1p) both MERFISH and scRNA data before scaling)
- **scikit-learn** (MaxAbsScaler scales each gene's values to the 0-1 range within each dataset)
- **gimVI** (Baseline method run on the identical 12 gene splits for imputation-accuracy comparison)
- **Tangram** (Baseline method run on the identical 12 gene splits for imputation-accuracy comparison)

## Examples

```
import uniport as up; adata = up.Run(adatas=[merfish, scrna], adata_cm=adata_cm, batch_size=512, iteration=30000, lambda_ot=1.0, lambda_s=0.5, mode='v', out='predict')
```

## Evaluation signals

- uniPort's aggregated MERFISH row reproduces approximately mSCC 0.259, aSCC 0.26, mPCC 0.249, aPCC 0.294
- uniPort exceeds gimVI (mSCC 0.221, aSCC 0.24, mPCC 0.201, aPCC 0.242) and Tangram (mSCC 0.188, aSCC 0.206, mPCC 0.202, aPCC 0.231) across the same 12 splits
- Each split contains exactly 122 training genes (80% of 153) and 31 held-out genes (20%), and all three methods are evaluated on identical splits
- No cells labeled 'Ambiguous' or 'Unstable' remain in either dataset, and gene values fall within the 0-1 MaxAbsScaler range
- Output table merfish_imputation_metrics.csv exists with per-split metrics for all three methods

## Limitations

- uniPort's scATAC-style integration relies on gene activity scores rather than directly modeling regulatory signals
- Storing a dense full OT plan (save_OT=True) requires substantial memory; computing capacity must be ascertained beforehand
- The exact hyperparameter combination selected per dataset from the searched grids (λ ∈ {0.5, 1.0, 5.0}, γ ∈ {0.5, 1.0}, λs ∈ {0.5, 1.0}) is not stated

## Evidence

- [results] uniPort achieved the highest imputation correlations on the MERFISH data: [evidence span withheld — anchor status: not_found]
- [discussion] All cells labeled as Ambiguous or Unstable were removed from both datasets: "All cells labeled as ‘Ambiguous’ or ‘Unstable’ were removed from both datasets"
- [discussion] MaxAbsScaler normalization to 0-1: "We normalized values of each gene to the range of 0-1 within each dataset by the MaxAbsScaler function"
- [readme] out='predict' mode documentation: "If out=='predict', project data into the latent space and output cell embeddings through a specified decoder. Default: 'latent'"
- [abstract] Scalable coupled-VAE plus Minibatch-UOT framework: "uniPort, a unified single-cell data integration framework that combines a coupled variational autoencoder (coupled-VAE) and minibatch unbalanced optimal transport (Minibatch-UOT)."
