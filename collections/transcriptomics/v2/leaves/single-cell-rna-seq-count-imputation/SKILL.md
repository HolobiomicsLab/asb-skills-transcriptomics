---
name: single-cell-rna-seq-count-imputation
description: Use when raw scRNA-seq UMI counts (e.g., UC mucosa cells) need denoised/imputed
  expression values via the scPhere decoder, and when dropouts must be reduced without
  introducing false-positive expression of marker genes.
when_to_use_negative:
- The article's discussion explicitly flags imputation of missing counts and ambient-RNA
  removal as requiring further validation, so do not treat scPhere decoder outputs
  as a validated general-purpose imputation or ambient-RNA-correction tool beyond
  demonstrated denoising cases.
- Input is not raw UMI counts (e.g., already log-normalized or TPM data) — the model
  is built for UMI count matrices with a negative-binomial observation distribution
  ('nb').
- No trained scPhere model exists for the tissue/batch structure at hand; training
  a model just for ad hoc imputation without checking the batch-invariant setting
  is not supported by the demonstrated workflow.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPhere
  role: Deep generative model whose decoder outputs the negative-binomial mean imputed
    count vector per input cell; installed via 'python setup.py install' from the
    cloned repository.
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: TensorFlow
  role: Backend (v1.14.0) on which the scPhere VAE decoder runs to compute imputed
    expression.
- name: NumPy
  role: Computes the Pearson correlation between CD8A and CD8B (raw vs. imputed) via
    numpy corrcoef and writes the imputed matrix with np.savetxt.
- name: pandas
  role: Loads the UMI count matrix and batch vectors prior to decoder imputation and
    handles TSV output of the imputed expression.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:3009cfdd779c96b5e01ee1ad7a721b23e36d7db3baed89498ad3ca2246071460
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klarman-cell-observatory/scPhere
  tools:
  - NumPy
  - TensorFlow
  - pandas
  - scPhere
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# single-cell-rna-seq-count-imputation

## Summary

Denoise/impute UMI count matrices by passing cell count vectors through a trained batch-invariant scPhere decoder (negative-binomial observation model, observation_dist='nb'), producing mean imputed expression per cell and gene; validated on UC CD8+ T cells where the CD8A–CD8B Pearson correlation rises from ~0.27 (raw counts) to ~0.81 (imputed) without inducing false-positive CD4 expression.

## When to use

Use when raw scRNA-seq UMI counts (e.g., UC mucosa cells) need denoised/imputed expression values via the scPhere decoder, and when dropouts must be reduced without introducing false-positive expression of marker genes.

## When NOT to use

- The article's discussion explicitly flags imputation of missing counts and ambient-RNA removal as requiring further validation, so do not treat scPhere decoder outputs as a validated general-purpose imputation or ambient-RNA-correction tool beyond demonstrated denoising cases.
- Input is not raw UMI counts (e.g., already log-normalized or TPM data) — the model is built for UMI count matrices with a negative-binomial observation distribution ('nb').
- No trained scPhere model exists for the tissue/batch structure at hand; training a model just for ad hoc imputation without checking the batch-invariant setting is not supported by the demonstrated workflow.

## Inputs

- Raw scRNA-seq UMI count matrix (e.g., UC CD8+ T cell count matrix, loadable as a dense matrix via pandas/numpy or scPhere's read_mtx on an .mtx file)
- Previously trained batch-invariant scPhere model checkpoint (saved via model.save_sess), built with observation_dist='nb' and latent_dist='vmf'
- Optional batch vector (0-based, e.g., [30, 3, 2] batch components for patient/health/location) if the model is not batch-invariant

## Outputs

- Imputed (negative-binomial mean) expression matrix, cells × genes, saved as TSV
- CD8A–CD8B Pearson correlation summary (raw ~0.27 vs. imputed ~0.81) as TSV
- Scatter/summary figure analogous to Supplementary Fig. 11a

## How to apply

Load the UMI count matrix with pandas/numpy and restore the previously trained batch-invariant scPhere model saved via model.save_sess. Feed the count vectors through the model decoder with the negative-binomial observation model (observation_dist='nb') to obtain the NB-mean imputed expression for each cell and gene. Quantify denoising by computing the Pearson correlation (numpy corrcoef) of a known co-expressed marker pair (CD8A and CD8B in CD8+ T cells) before and after imputation; expect a rise from ~0.27 to ~0.81. Guard against false positives by checking that a gene not expressed in the population (CD4 in CD8+ T cells) remains lowly expressed in the imputed matrix. Save the imputed matrix and raw-vs-imputed correlation summary as TSV files and generate a scatter/summary figure analogous to Supplementary Fig. 11a.

## Related tools

- **scPhere** (Deep generative model whose decoder outputs the negative-binomial mean imputed count vector per input cell; installed via 'python setup.py install' from the cloned repository.) — https://github.com/klarman-cell-observatory/scPhere
- **TensorFlow** (Backend (v1.14.0) on which the scPhere VAE decoder runs to compute imputed expression.)
- **NumPy** (Computes the Pearson correlation between CD8A and CD8B (raw vs. imputed) via numpy corrcoef and writes the imputed matrix with np.savetxt.)
- **pandas** (Loads the UMI count matrix and batch vectors prior to decoder imputation and handles TSV output of the imputed expression.)

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=True, observation_dist='nb', seed=0); model.save_sess(model_name)
```

## Evaluation signals

- Imputed CD8A–CD8B Pearson correlation reaches approximately 0.81, up from a raw-count value of ~0.27 (computed with numpy corrcoef).
- No cell shows spurious imputed CD4 expression — CD4, which is not expressed in CD8+ T cells, remains lowly expressed after denoising.
- The imputed values correspond to negative-binomial means (observation_dist='nb') from the decoder, not raw or normalized counts.
- Imputed expression matrix and the raw-vs-imputed correlation summary are saved as TSV files, and a scatter/summary figure analogous to Supplementary Fig. 11a is produced.
- The model used was previously saved with model.save_sess and trained with a scPhere configuration (latent_dist='vmf', z_dim, seed) that is reproducible.

## Limitations

- The authors state that 'further studies are required to explore its abilities in imputing missing counts in scRNA-seq data and removing ambient RNA contamination' — decoder denoising is demonstrated, not broadly validated.
- Imputation quality depends on training a batch-invariant model (batch_invariant=True) appropriate to the tissue; the demonstrated validation is limited to UC CD8+ T cells and the CD8A/CD8B vs. CD4 marker panel.
- Batch vectors must be 0-based, and reproducing the exact model requires fixing the seed parameter; mismatched gene sets (n_gene) between training and inference data will break decoder imputation.

## Evidence

- [discussion] further studies are required to explore its abilities in imputing missing counts in scRNA-seq data and removing ambient RNA contamination: "further studies are required to explore its abilities in imputing missing counts in scRNA-seq data and removing ambient RNA contamination"
- [methods] model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0): "model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0)"
- [readme] observation_dist: the gene expression distribution, 'nb' for negative binomial: "observation_dist: the gene expression distribution, 'nb' for negative binomial"
- [methods] model.save_sess(model_name): "save_path = '/Users/jding/work/scdata/UC-out/'

model_name = save_path + 'uc_epi_model_250epoch'
model.save_sess(model_name)"
