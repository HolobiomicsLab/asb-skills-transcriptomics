---
name: gene-set-score-feature-augmentation
description: Use when preparing representative single-cell reference data for scSemiProfiler's
  deep generative model, where raw normalized expression alone under-constrains in
  silico inference of target samples.
when_to_use_negative:
- When running pure bulk deconvolution tools (CIBERSORTx, Bisque, DWLS, MuSiC, NNLS)
  that only need a signature gene matrix, not augmented single-cell features
- When no gene set reference collections (MSigDB / Ernst et al.) are available or
  the data are from a modality (e.g. proteomics) not covered by these gene sets
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: scSemiProfiler
  role: Implements gene set score computation and variance-based feature weighting
    as part of the representative single-cell data processing step of the semi-profiling
    pipeline
  repo: https://github.com/mcgilldinglab/scSemiProfiler
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:9ee77a1cc6078272180318d9b95dac3e67c5ac95f1bb9b96ba059691303e6858
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - scSemiProfiler
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Gene-set score feature augmentation

## Summary

Augments preprocessed single-cell expression matrices with gene set scores computed from all curated MSigDB and Ernst et al. gene sets, and reweights features by variance, to give the scSemiProfiler VAE-GAN biologically informative inputs beyond raw expression values.

## When to use

Use when preparing representative single-cell reference data for scSemiProfiler's deep generative model, where raw normalized expression alone under-constrains in silico inference of target samples.

## When NOT to use

- When running pure bulk deconvolution tools (CIBERSORTx, Bisque, DWLS, MuSiC, NNLS) that only need a signature gene matrix, not augmented single-cell features
- When no gene set reference collections (MSigDB / Ernst et al.) are available or the data are from a modality (e.g. proteomics) not covered by these gene sets

## Inputs

- Quality-controlled representative single-cell count matrix (AnnData/.h5ad) normalized to 10,000 counts per cell with log1p transformation
- Curated gene set collections: MSigDB (all curated gene sets) and Ernst et al. gene sets

## Outputs

- Gene-set-score-augmented single-cell feature matrix (original genes plus MSigDB/Ernst gene set scores) with variance-based feature importance weights, used as VAE-GAN training input

## How to apply

After quality-controlling the representative single-cell data (zeroing expression below 0.1% of library size, dropping cells expressing fewer than 200 genes, normalizing to 10,000 counts per cell with log1p), compute gene set scores using all curated gene sets from MSigDB and the gene sets from Ernst et al., appending them as extra features in the AnnData matrix. In parallel, adopt a feature weighting scheme that gives more weight to features with higher variance so that informative genes and gene set scores dominate the reconstruction loss. These augmented, weighted matrices then feed the VAE-GAN pretraining (SCVI, learning rate 1e-3, ~100 epochs) and the subsequent in silico fine-tuning against target-sample bulk profiles. The rationale is that pathway-level scores capture biological programs (e.g. immune activation in the COVID-19 cohort) that pure per-gene expression may represent too sparsely for the generative model.

## Related tools

- **scSemiProfiler** (Implements gene set score computation and variance-based feature weighting as part of the representative single-cell data processing step of the semi-profiling pipeline) — https://github.com/mcgilldinglab/scSemiProfiler

## Examples

```
import scSemiProfiler; scSemiProfiler.main.preprocess representatives and compute gene set scores via the documented preprocessing workflow (see example.ipynb) before scSemiProfiler.main.pretrain
```

## Evaluation signals

- Augmented AnnData contains the original ~6000 highly variable genes plus marker genes and additional columns matching the number of MSigDB/Ernst curated gene sets
- Gene set scores are finite and computed after the 10,000-count normalization and log1p transformation, not on raw counts
- Feature importance weights are strictly positive and monotonically increasing with gene/gene-set variance
- Downstream VAE-GAN pretraining on augmented features converges (~100 epochs at learning rate 1e-3) and semi-profiled output passes UMAP and cell-type-proportion comparison against real single-cell data
- Semi-profiled COVID-19 cohort cost stays within the $60,000-$66,000 validation window (0.30-0.37 of the $186,000 full cost), indicating the augmented features did not degrade inference

## Limitations

- Feature augmentation depends on the coverage and relevance of MSigDB/Ernst gene sets; biological programs outside these collections are not represented
- The article notes extensions to modalities beyond scRNA-seq (e.g. proteomics) are future work and would require algorithm adjustments, so this augmentation is currently scRNA-seq specific
- Gene set scores add dimensions to the generative model and presuppose a single-cell reference; without one, scSemiProfiler's single-cell level deconvolution cannot run

## Evidence

- [methods] gene set scores are calculated using all curated gene sets from MSigDB59 and Ernst et al.91: "gene set scores are calculated using all curated gene sets from MSigDB59 and Ernst et al.91"
- [methods] we adopt a feature weighting method, giving more weight to features with higher variance: "we adopt a feature weighting method, giving more weight to features with higher variance"
- [methods] the removal of low-quality cells (expressing fewer than 200 genes, per SCANPY's default settings): [evidence span withheld — anchor status: not_found]
- [readme] a deep generative model first learns the distribution of the reference single-cell data, generating reconstructions of the reference cells: [evidence span withheld — anchor status: external_doc]
