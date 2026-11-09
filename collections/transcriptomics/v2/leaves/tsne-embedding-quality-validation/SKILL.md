---
name: tsne-embedding-quality-validation
description: Use when generated or augmented scRNA-seq cells (e.g. from scGAN/cscGAN)
  must be checked for realism against held-out real cells before downstream analyses
  such as marker gene detection or classifier training.
when_to_use_negative:
- When only the test set is available — the README states the test set is not used
  and validation t-SNE plots must be computed on the validation set
- When quantitative benchmarks alone suffice and no visual overlap check of generated
  vs. real cells is required
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: t-SNE
  role: Produces the embedding of validation cells plus generated cells used as the
    qualitative realism check
- name: Tensorboard
  role: Monitors the evolution of generator/discriminator losses and summary operations
    to confirm stable training before trusting t-SNE plots
- name: scGAN
  role: Source of the generated cells being validated against the real validation
    set
  repo: https://github.com/imsb-uke/scGAN
- name: cscGAN
  role: Conditional model whose generated cells are additionally assessed in conditional
    t-SNE plots
  repo: https://github.com/imsb-uke/scGAN
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:79a148b916da50ac01d4a5866b516f45cc1169505118f3a99c851fed3ce5b1a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/imsb-uke/scGAN
  tools:
  - Docker
  - Python
  - Tensorboard
  - cscGAN
  - scGAN
  - t-SNE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# tsne-embedding-quality-validation

## Summary

Validates GAN-generated single-cell RNA-seq data by producing t-SNE embeddings computed on the whole validation set plus an equal number of generated cells, and confirming the generated cells visually overlap the real validation cells.

## When to use

Use when generated or augmented scRNA-seq cells (e.g. from scGAN/cscGAN) must be checked for realism against held-out real cells before downstream analyses such as marker gene detection or classifier training.

## When NOT to use

- When only the test set is available — the README states the test set is not used and validation t-SNE plots must be computed on the validation set
- When quantitative benchmarks alone suffice and no visual overlap check of generated vs. real cells is required

## Inputs

- Trained scGAN/cscGAN model checkpoint (path given via --model_path)
- Pre-processed validation set stored in TFRecord format from .h5/.h5ad scRNA-seq data
- Generated cells file (.h5ad, e.g. produced with --generate --cells_no)
- parameters. configuration with plotting frequency and output directories

## Outputs

- Non-conditional t-SNE validation plots embedding validation cells and generated cells
- Conditional t-SNE plots for cscGAN models
- Tensorboard loss summaries confirming training stability

## How to apply

During or after scGAN/cscGAN training, use the validation set (not the test set, which is reserved for custom experiments) to produce t-SNE plots; each embedding is computed using the whole validation set plus an equal number of generated cells. Set the t-SNE plotting frequency in the parameters. file so plots are produced at regular intervals. For non-conditional models, judge success qualitatively by generated cells overlapping the real validation cells in the embedding, matching the reference non-conditional t-SNE figure in the repository. Complement the visual check with Tensorboard loss curves to confirm stable GAN training convergence, and optionally with quantitative metrics such as MMD and classifier AUC used in the paper's comparisons (e.g. SUGAR vs. scGAN).

## Related tools

- **t-SNE** (Produces the embedding of validation cells plus generated cells used as the qualitative realism check)
- **Tensorboard** (Monitors the evolution of generator/discriminator losses and summary operations to confirm stable training before trusting t-SNE plots)
- **scGAN** (Source of the generated cells being validated against the real validation set) — https://github.com/imsb-uke/scGAN
- **cscGAN** (Conditional model whose generated cells are additionally assessed in conditional t-SNE plots) — https://github.com/imsb-uke/scGAN

## Examples

```
python main.py --param parameters. --generate --cells_no 1000 500 0 200 --model_path path/to/my/model --save_path where_to_save.h5ad
```

## Evaluation signals

- Generated cells visually overlap the real validation-set cells in the t-SNE embedding, matching the paper's reference non-conditional t-SNE figure
- Each embedding is computed on the whole validation set plus an equal number of generated cells (balanced embedding sizes)
- Tensorboard loss curves show stable, converged GAN training without mode collapse
- MMD and AUC metrics align with reported ranges (e.g. scGAN MMD 0.87, AUC 0.65 vs. real-data MMD 0.03, AUC 0.52 in the PBMC comparison)

## Limitations

- The t-SNE overlap criterion is qualitative; quantitative realism must be supported with metrics such as MMD and AUC
- The test set is not used for validation plots — misuse of test data would leak evaluation data
- Each t-SNE embedding must include an equal number of generated cells and validation cells; unbalanced counts distort the visual comparison

## Evidence

- [readme] Note that the validation set is used to produce the t-SNE plots (each embedding is computed using the whole validation set and an equal number of generated cells).: "Note that the validation set is used to produce the t-SNE plots (each embedding is computed using the whole validation set and an equal number of generated cells)."
- [readme] you can use the Tensorboard to monitor the losses (for instance) evolve.: "you can use the Tensorboard to monitor the losses (for instance) evolve."
- [methods] the validation set is used to produce the t-SNE plots: "the validation set is used to produce the t-SNE plots"
- [results] SUGAR's metrics are significantly higher than the MMD (0.87) and AUC (0.65) of the scGAN and the MMD (0.03) and AUC (0.52) of the real data: [evidence span withheld — anchor status: not_found]
