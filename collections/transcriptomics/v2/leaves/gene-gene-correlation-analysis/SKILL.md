---
name: gene-gene-correlation-analysis
description: Use when you need to validate denoised/imputed single-cell expression
  by comparing gene-gene Pearson correlations before and after imputation, or to detect
  spurious marker expression in a specific cell population.
when_to_use_negative:
- The article notes further studies are required to validate scPhere's abilities in
  imputing missing counts and removing ambient RNA contamination, so do not treat
  imputed correlations as a substitute for validated imputation in those settings
- Raw counts are adequate and the goal is simply a descriptive correlation between
  markers, with no denoised/imputed values available
- The trained scPhere model's gene set does not match the input matrix gene dimensionality
  (n_gene must equal x.shape[1])
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scPhere
  role: Deep generative model whose decoder outputs a negative-binomial UMI count
    vector for each input cell, used to produce imputed/denoised expression values
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: NumPy
  role: Compute Pearson correlations (numpy corrcoef) between gene pairs on raw and
    imputed matrices
- name: pandas
  role: Load the UMI count matrix and save imputed expression and correlation summaries
    as TSV files
- name: TensorFlow
  role: Backend for running the trained scPhere model decoder to generate imputed
    expression
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:bec9f92040d2fcf0bc2ce934ca50d16a064f85c12759b5aca5307ce6c9caefab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klarman-cell-observatory/scPhere
  tools:
  - NumPy
  - TensorFlow
  - pandas
  - scPhere
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# gene-gene-correlation-analysis

## Summary

Compute Pearson correlations between gene pairs (e.g., CD8A and CD8B) on raw scRNA-seq UMI counts versus scPhere decoder-imputed negative-binomial expression, to quantify denoising effects and verify that imputation strengthens expected co-expression while avoiding false positives.

## When to use

Use when you need to validate denoised/imputed single-cell expression by comparing gene-gene Pearson correlations before and after imputation, or to detect spurious marker expression in a specific cell population.

## When NOT to use

- The article notes further studies are required to validate scPhere's abilities in imputing missing counts and removing ambient RNA contamination, so do not treat imputed correlations as a substitute for validated imputation in those settings
- Raw counts are adequate and the goal is simply a descriptive correlation between markers, with no denoised/imputed values available
- The trained scPhere model's gene set does not match the input matrix gene dimensionality (n_gene must equal x.shape[1])

## Inputs

- Raw UMI count matrix for the cell population of interest (e.g., UC CD8+ T cells), loaded with pandas/numpy
- Previously trained batch-invariant scPhere model checkpoint saved via model.save_sess (e.g., uc_epi_model_250epoch)
- Gene name/index mapping to locate genes such as CD8A, CD8B, and CD4

## Outputs

- Imputed (decoder negative-binomial mean) expression matrix saved as TSV
- CD8A-CD8B Pearson correlation summary (raw ~0.27 vs. imputed ~0.81) saved as TSV
- Scatter/summary figure analogous to Supplementary Fig. 11a

## How to apply

Load the raw UMI count matrix (e.g., UC CD8+ T cells) with pandas/numpy and load the previously trained batch-invariant scPhere model saved via model.save_sess. Pass the count vectors through the model decoder with observation_dist='nb' to obtain the negative-binomial mean imputed expression for each cell and gene. Compute the Pearson correlation for a biologically linked gene pair (here CD8A and CD8B) on raw counts and imputed values using numpy corrcoef, expecting the correlation to rise from ~0.27 to ~0.81. As a negative control, inspect imputed CD4 expression across CD8+ T cells to confirm no false-positive signal is introduced. Save the imputed matrix and the raw-vs-imputed correlation summary as TSV files, and produce a scatter/summary figure analogous to Supplementary Fig. 11a.

## Related tools

- **scPhere** (Deep generative model whose decoder outputs a negative-binomial UMI count vector for each input cell, used to produce imputed/denoised expression values) — https://github.com/klarman-cell-observatory/scPhere
- **NumPy** (Compute Pearson correlations (numpy corrcoef) between gene pairs on raw and imputed matrices)
- **pandas** (Load the UMI count matrix and save imputed expression and correlation summaries as TSV files)
- **TensorFlow** (Backend for running the trained scPhere model decoder to generate imputed expression)

## Examples

```
corr_raw = np.corrcoef(x[:, cd8a_idx], x[:, cd8b_idx])[0,1]; imputed = model.decode(x, batch); corr_imp = np.corrcoef(imputed[:, cd8a_idx], imputed[:, cd8b_idx])[0,1]
```

## Evaluation signals

- Imputed CD8A-CD8B Pearson correlation reaches approximately 0.81, up from a raw value of ~0.27
- No cell shows spurious imputed CD4 expression in CD8+ T cells (no false positives introduced by denoising)
- Decoder outputs are valid UMI count vectors matching the input gene dimensionality (n_gene = x.shape[1])
- Imputed expression matrix and raw-vs-imputed correlation summary files are produced as TSV with consistent cell/gene ordering
- Figure analogous to Supplementary Fig. 11a can be regenerated from the saved outputs

## Limitations

- The paper states that further studies are required to explore scPhere's abilities in imputing missing counts in scRNA-seq data and removing ambient RNA contamination, so this denoising use is not fully validated generally
- Imputation quality depends on a previously trained batch-invariant scPhere model; training used max_epoch=250, mb_size=128, learning_rate=0.001, observation_dist='nb'
- Correlation improvement is demonstrated for CD8A/CD8B in UC CD8+ T cells; other gene pairs or populations may behave differently

## Evidence

- [results] CD8A-CD8B correlation rise: [evidence span withheld — anchor status: not_found]
- [results] no false positives: [evidence span withheld — anchor status: not_found]
- [discussion] imputation future work: "further studies are required to explore its abilities in imputing missing counts in scRNA-seq data and removing ambient RNA contamination"
- [readme] model construction parameters: "model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0)"
- [methods] save trained model: "save_path = '/Users/jding/work/scdata/UC-out/'

model_name = save_path + 'uc_epi_model_250epoch'
model.save_sess(model_name)"
