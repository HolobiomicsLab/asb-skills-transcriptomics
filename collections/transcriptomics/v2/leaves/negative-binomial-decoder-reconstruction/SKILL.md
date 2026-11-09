---
name: negative-binomial-decoder-reconstruction
description: Use when raw UMI count matrices show noisy gene-gene relationships (e.g.,
  low marker co-expression correlations) and a trained batch-invariant scPhere model
  is available whose decoder can impute denoised negative-binomial mean expression
  values.
when_to_use_negative:
- No trained scPhere model exists for the dataset — the decoder cannot reconstruct
  meaningful expression without a fitted model
- The task is batch correction or latent-space embedding rather than expression imputation/denoising
  (use model.encode / Trainer instead)
- Ambient RNA contamination removal is the goal — the paper explicitly states this
  remains an unvalidated future direction
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPhere
  role: Deep generative model whose negative-binomial decoder imputes a UMI count
    vector for each input cell
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: TensorFlow
  role: Backend dependency (v1.14.0) on which the scPhere decoder runs
- name: NumPy
  role: Computes Pearson correlations (numpy corrcoef) between raw and imputed gene
    expression
- name: pandas
  role: Loads the UMI count matrix and batch vector files (TSV/CSV) prior to decoding
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:e05116956853cb67bd3af3f7625d8deed74f3c99bcdb50f9b8c4a9d45a64281a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klarman-cell-observatory/scPhere
  tools:
  - NumPy
  - TensorFlow
  - pandas
  - scPhere
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# negative-binomial-decoder-reconstruction

## Summary

Reconstructs denoised per-cell gene expression by feeding UMI count vectors through the scPhere decoder, which outputs a negative-binomial mean ('nb' observation_dist) count vector per input cell, enabling imputation and denoising of scRNA-seq data (e.g., raising CD8A–CD8B correlation from 0.27 to 0.81 in UC CD8+ T cells).

## When to use

Use when raw UMI count matrices show noisy gene-gene relationships (e.g., low marker co-expression correlations) and a trained batch-invariant scPhere model is available whose decoder can impute denoised negative-binomial mean expression values.

## When NOT to use

- No trained scPhere model exists for the dataset — the decoder cannot reconstruct meaningful expression without a fitted model
- The task is batch correction or latent-space embedding rather than expression imputation/denoising (use model.encode / Trainer instead)
- Ambient RNA contamination removal is the goal — the paper explicitly states this remains an unvalidated future direction

## Inputs

- Raw UMI count matrix for target cells (e.g., UC CD8+ T cells), loaded with pandas/NumPy
- Previously trained batch-invariant scPhere model checkpoint saved via model.save_sess (observation_dist='nb')
- 0-based batch identifier vector for the cells (or artificial -1 vector when no batch vectors apply)

## Outputs

- Imputed/denoised expression matrix (negative-binomial decoder means) per cell and gene, saved as TSV
- CD8A–CD8B Pearson correlation summary comparing raw (~0.27) vs. imputed (~0.81) values as TSV
- Scatter/summary figure analogous to Supplementary Fig. 11a

## How to apply

Load the raw UMI count matrix (e.g., the UC CD8+ T cell matrix, 0-based batch vectors via pandas) and restore the previously trained batch-invariant scPhere model saved with model.save_sess. Build or load the model with observation_dist='nb' so the decoder emits negative-binomial mean expression, then feed each cell's count vector through the decoder to obtain the imputed expression matrix. Quantify the effect with numpy corrcoef on raw vs. imputed values (expected: CD8A–CD8B Pearson correlation rises from ~0.27 to ~0.81) and verify no false positives appear (CD4, absent in CD8+ T cells, remains low). Save the imputed matrix and correlation summary as TSV files and a scatter/summary figure analogous to Supplementary Fig. 11a.

## Related tools

- **scPhere** (Deep generative model whose negative-binomial decoder imputes a UMI count vector for each input cell) — https://github.com/klarman-cell-observatory/scPhere
- **TensorFlow** (Backend dependency (v1.14.0) on which the scPhere decoder runs)
- **NumPy** (Computes Pearson correlations (numpy corrcoef) between raw and imputed gene expression)
- **pandas** (Loads the UMI count matrix and batch vector files (TSV/CSV) prior to decoding)

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0)
```

## Evaluation signals

- Imputed CD8A–CD8B Pearson correlation reaches approximately 0.81 (up from a raw value of ~0.27)
- No cell shows spurious imputed CD4 expression in CD8+ T cells (no false positives introduced by denoising)
- Saved model checkpoint (model.save_sess) and artifacts (imputed TSV, correlation summary TSV, figure) reproduce Supplementary Fig. 11a analogously
- Decoder run uses observation_dist='nb' so outputs are negative-binomial mean imputed counts, not raw counts or normalized values

## Limitations

- The paper states further studies are required to validate scPhere's ability to impute missing counts and remove ambient RNA contamination — this decoder reconstruction is a demonstration, not a fully validated imputation method
- Requires a pre-trained batch-invariant scPhere model (batch_invariant=True) so gene expression vectors alone can be mapped; a model trained with batch vectors must be retrained for new-batch reconstruction
- Denoising behavior (no false positives) was verified only for markers like CD8A/CD8B/CD4 in UC CD8+ T cells; per-dataset marker checks are needed

## Evidence

- [discussion] further studies are required to explore its abilities in imputing missing counts in scRNA-seq data and removing ambient RNA contamination: "further studies are required to explore its abilities in imputing missing counts in scRNA-seq data and removing ambient RNA contamination"
- [readme] observation_dist: the gene expression distribution, 'nb' for negative binomial: "observation_dist: the gene expression distribution, 'nb' for negative binomial"
- [methods] model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0): "model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0)"
- [readme] The batch vector should be 0-based: "# The batch vector should be 0-based"
