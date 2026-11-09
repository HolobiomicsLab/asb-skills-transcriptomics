---
name: generated-cell-quality-evaluation-rf-auc-mmd
description: Use when you have cscGAN/scGAN-generated scRNA-seq cells (e.g. .h5ad
  output for specific PBMC clusters) and need to judge whether they are realistic
  and indistinguishable from real cells of the target clusters.
when_to_use_negative:
- You want to evaluate biological utility (e.g. marker gene detection or classifier
  augmentation gains) rather than realism of the synthetic cells themselves
- Input data are not single-cell RNA-seq count/expression matrices — the RF and MMD
  evaluation is calibrated on scRNA-seq representations (e.g. PBMC clusters)
- No cluster-annotated real cells are available for the target clusters, so the real-vs-synthetic
  comparison cannot be constructed
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: cscGAN
  role: Conditional single-cell GAN whose generated PBMC cluster cells are the object
    being evaluated for realism
  repo: github.com/imsb-uke/scGAN
- name: Tensorboard
  role: Used to monitor training losses and where validation t-SNE plots embedding
    real validation cells and generated cells are produced for visual quality checking
- name: t-SNE
  role: Embedding used to visually verify that generated cells overlap their target
    clusters in the validation set
- name: TensorFlow
  role: Framework in which the scGAN/cscGAN models and their validation plots are
    implemented
  repo: github.com/imsb-uke/scGAN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:1cda5e16c844ebd3cfdcd92a31f04af905f6740883296b1a83ce23d004046de5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - TensorFlow
  - Tensorboard
  - cscGAN
  - t-SNE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Generated-cell quality evaluation via Random Forest AUC and MMD

## Summary

Quantitative realism check for GAN-generated scRNA-seq cells: train a Random Forest classifier to distinguish synthetic from real cells (AUC near 0.5 = hard to distinguish) and compute maximum mean discrepancy (MMD) between synthetic and real distributions, complemented by conditional t-SNE inspection.

## When to use

Use when you have cscGAN/scGAN-generated scRNA-seq cells (e.g. .h5ad output for specific PBMC clusters) and need to judge whether they are realistic and indistinguishable from real cells of the target clusters.

## When NOT to use

- You want to evaluate biological utility (e.g. marker gene detection or classifier augmentation gains) rather than realism of the synthetic cells themselves
- Input data are not single-cell RNA-seq count/expression matrices — the RF and MMD evaluation is calibrated on scRNA-seq representations (e.g. PBMC clusters)
- No cluster-annotated real cells are available for the target clusters, so the real-vs-synthetic comparison cannot be constructed

## Inputs

- Generated cells saved as .h5ad from the cscGAN --generate step (e.g. generated_cells.h5ad)
- Real cluster-annotated PBMC scRNA-seq cells from the held-out validation set in .h5/.h5ad or TFRecord format
- Cluster indices used for conditioning (e.g. cluster 2 and cluster 6 of Fresh 68k PBMCs Donor A)

## Outputs

- Random Forest classifier AUC for real-vs-generated discrimination per cluster (reference: 0.62 for cluster 2, 0.55 for cluster 6)
- MMD distance between generated and real cells per cluster (reference: 0.286 for cluster 2, 0.238 for cluster 6)
- Conditional t-SNE embedding of the validation set plus an equal number of generated cells

## How to apply

After generating cluster-conditioned cells with the cscGAN (--generate flag, e.g. cluster 2 and cluster 6 of the Fresh 68k PBMCs Donor A dataset), compare them against real cells of the corresponding clusters using two complementary metrics. First, train a Random Forest classifier on the real-vs-synthetic discrimination task: an AUC near 0.5 indicates the generator fools the classifier, while values approaching 1.0 indicate obvious artifacts — the reference cscGAN results are AUC 0.62 (cluster 2) and 0.55 (cluster 6). Second, compute the maximum mean discrepancy (MMD) distance between generated and real cells of each cluster; lower MMD means closer distributions (reference values 0.286 and 0.238 for clusters 2 and 6 respectively). Finally, inspect the conditional t-SNE produced during training, which embeds the whole validation set together with an equal number of generated cells, and confirm that generated cells overlap their target cluster. A generated set is judged successful when RF AUC is near chance, MMD is low, and the conditional t-SNE shows overlap with target clusters.

## Related tools

- **cscGAN** (Conditional single-cell GAN whose generated PBMC cluster cells are the object being evaluated for realism) — github.com/imsb-uke/scGAN
- **Tensorboard** (Used to monitor training losses and where validation t-SNE plots embedding real validation cells and generated cells are produced for visual quality checking)
- **t-SNE** (Embedding used to visually verify that generated cells overlap their target clusters in the validation set)
- **TensorFlow** (Framework in which the scGAN/cscGAN models and their validation plots are implemented) — github.com/imsb-uke/scGAN

## Examples

```
python main.py --param parameters. --generate --cells_no 1000 1000 --model_path path/to/model --save_path generated_cells.h5ad
```

## Evaluation signals

- Random Forest real-vs-synthetic AUC close to chance (reference targets: ~0.62 for PBMC cluster 2, ~0.55 for cluster 6)
- MMD distance between generated and real cells of the same cluster falls near reference values 0.286 (cluster 2) and 0.238 (cluster 6); substantially higher MMD indicates distribution mismatch
- Conditional t-SNE shows generated cells overlapping their target clusters (plot embeds the whole validation set plus an equal number of generated cells)
- Generator/discriminator losses monitored in Tensorboard are stable and no mode collapse is visible across checkpoints

## Limitations

- The reported AUC values (0.62 and 0.55) are dataset- and cluster-specific to Fresh 68k PBMCs Donor A clusters 2 and 6 and should be re-established for other datasets
- The mechanism enabling cscGAN to generate cells from very small subpopulations (e.g. 16 cells) is only speculated, not established, so realism metrics for tiny subpopulations may not transfer
- Visual t-SNE overlap is qualitative and can be misleading without the quantitative RF AUC and MMD counterparts

## Evidence

- [results] RF classifier AUC and MMD targets: [evidence span withheld — anchor status: not_found]
- [abstract] cscGAN realism claim: "cscGAN learns non-linear gene-gene dependencies from complex, multiple cell type samples and uses this information to generate realistic cells of defined types"
- [readme] t-SNE validation protocol: "the validation set is used to produce the t-SNE plots (each embedding is computed using the whole validation set and an equal number of generated cells)"
- [methods] generation command: "To generate cells and write them to a file, you can use the --generate flag."
- [methods] Tensorboard loss monitoring: "you can use the Tensorboard to monitor the losses (for instance) evolve."
