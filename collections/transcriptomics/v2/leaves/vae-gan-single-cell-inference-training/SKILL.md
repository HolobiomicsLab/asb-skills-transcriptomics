---
name: vae-gan-single-cell-inference-training
description: Use when a cohort has bulk sequencing data for all samples but single-cell
  data for only a few representatives, and single-cell-resolution profiles must be
  generated for the remaining target samples.
when_to_use_negative:
- The input bulk data has no accompanying single-cell reference — single-cell-level
  deconvolution via the deep generative model is only possible 'provided there is
  a single-cell reference'
- The goal is proteomics or another non-scRNA-seq modality — extension beyond single-cell
  RNA sequencing is future work only
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: scSemiProfiler
  role: Implements the VAE-GAN pretraining (Pretrain 1 SCVI + GAN epochs, Pretrain
    2 bulk loss) and in silico inference fine-tuning used in semi-profiling
  repo: https://github.com/mcgilldinglab/scSemiProfiler
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:59072db173bada6771e1a83d8dc962dfc299f68f1c58cd9738b191b1da771358
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - scSemiProfiler
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# vae-gan-single-cell-inference-training

## Summary

Train and fine-tune the scSemiProfiler VAE-GAN deep generative model to reconstruct representative single-cell data and infer single-cell profiles for target samples from bulk RNA-seq, enabling cost-effective semi-profiling of large cohorts.

## When to use

Use when a cohort has bulk sequencing data for all samples but single-cell data for only a few representatives, and single-cell-resolution profiles must be generated for the remaining target samples.

## When NOT to use

- The input bulk data has no accompanying single-cell reference — single-cell-level deconvolution via the deep generative model is only possible 'provided there is a single-cell reference'
- The goal is proteomics or another non-scRNA-seq modality — extension beyond single-cell RNA sequencing is future work only

## Inputs

- Preprocessed bulk RNA-seq matrix for the cohort (normalized, log1p, top 6000 HVGs + markers, PCA to 100 PCs)
- Preprocessed single-cell data (h5ad) for selected representative samples
- Gene set resources: MSigDB curated gene sets and Ernst et al. gene sets for score augmentation

## Outputs

- Reconstructed single-cell data for each representative sample
- Inferred (in silico) single-cell profiles for each non-representative target sample
- Assembled semi-profiled cohort (semiprofiled_cohort.h5ad)

## How to apply

After preprocessing representative scRNA-seq (zero out expression below 0.1% of library size, drop cells expressing fewer than 200 genes, normalize to 10,000 counts with log1p, augment with MSigDB/Ernst gene set scores and variance-based feature weights), run two pretraining phases per representative: Pretrain 1 trains SCVI at learning rate 1e-3 for ~100 epochs followed by alternating generator-discriminator training 3 epochs each, then Pretrain 2 in full-batch mode with representative bulk loss at learning rate 1e-4. Fine-tune per target sample at learning rate 2e-4 using mini-stage gradient thresholds at max_expr, max_expr/2, max_expr/4, max_expr/8, and max_expr/(8*sqrt(2)), quadrupling the bulk-loss weight per mini-stage, with conversion ratio alpha = (B_T + 0.1)/(B_R + 0.1). The rationale is that the VAE-GAN learns the representative's single-cell distribution, and bulk information steers generation to match each target sample's bulk profile.

## Related tools

- **scSemiProfiler** (Implements the VAE-GAN pretraining (Pretrain 1 SCVI + GAN epochs, Pretrain 2 bulk loss) and in silico inference fine-tuning used in semi-profiling) — https://github.com/mcgilldinglab/scSemiProfiler

## Examples

```
from scSemiProfiler import main; main.pipeline('cohort/', '01', '001', bulktype='bulkdata')  # scSemiProfiler workflow (see example.ipynb)
```

## Evaluation signals

- Reconstructed representative single-cell data closely matches the real representative sequencing (pretraining reconstruction loss converged)
- Fine-tuned target-sample inferred data matches the target bulk profile under the alpha-scaled bulk loss with quadrupled mini-stage weights
- Downstream UMAP visualization and cell type proportions of semi-profiled vs real-profiled data show strong resemblance (per the COVID-19 124-sample validation)
- For the COVID-19 cohort with 28 representatives, total cost falls within $60,000-$66,000, i.e. cost ratio 0.30-0.37 of the $186,000 full single-cell cost

## Limitations

- Extension to proteomics and other modalities is future work; algorithm adjustments and comprehensive benchmarking against established standards are required for new modalities
- The deep generative model requires a single-cell reference to perform single-cell level deconvolution
- No software versions for scSemiProfiler or dependencies are stated in the evaluated sections

## Evidence

- [methods] pretrain generator and discriminator alternately: [evidence span withheld — anchor status: not_found]
- [methods] fine-tuning parameters: [evidence span withheld — anchor status: not_found]
- [discussion] VAE-GAN role: "employs a deep generative learning model, VAE-GAN, to infer the single-cell data of the remainder of the cohort."
- [readme] cost savings: [evidence span withheld — anchor status: external_doc]
