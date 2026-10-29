---
name: random-forest-classifier-training
description: Use when assessing whether augmenting a sparse or downsampled scRNA-seq
  cell population (e.g., cluster 2 from Fresh 68k PBMCs) improves downstream classification
  robustness, with F1 score as the metric.
when_to_use_negative:
- The single-cell data has no cell-type/cluster labels, so a supervised Random Forest
  classifier cannot be trained
- The goal is to evaluate the realism of generated cells directly (e.g., t-SNE validation
  plots) rather than the downstream effect of augmentation on classifier robustness
- The population of interest is not sparse or downsampled, so no augmentation-vs-degradation
  comparison can be made
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: cscGAN
  role: Conditional GAN used to generate cluster-2 cells that form the augmented condition
    for classifier robustness testing
  repo: github.com/imsb-uke/scGAN
- name: Python
  role: Implements Random Forest training, downsampling/upsampling of cluster 2, and
    F1-score computation over five random seeds
  repo: github.com/imsb-uke/scGAN
- name: TensorFlow
  role: Backend for training the cscGAN model whose generated cells supply the augmentation
    condition
  repo: github.com/imsb-uke/scGAN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:fecc4453f62d6d3fadaffffa84b71b38fbc6fb762491b3e38fe607e5e04b94b2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - Jupyter
  - Python
  - TensorFlow
  - cscGAN
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# random-forest-classifier-training

## Summary

Trains Random Forest classifiers on Fresh 68k PBMCs (Donor A) cluster-2 populations under downsampled, randomly upsampled, and cscGAN-augmented conditions to quantify how well generative augmentation restores classifier robustness, measured via F1 score.

## When to use

Use when assessing whether augmenting a sparse or downsampled scRNA-seq cell population (e.g., cluster 2 from Fresh 68k PBMCs) improves downstream classification robustness, with F1 score as the metric.

## When NOT to use

- The single-cell data has no cell-type/cluster labels, so a supervised Random Forest classifier cannot be trained
- The goal is to evaluate the realism of generated cells directly (e.g., t-SNE validation plots) rather than the downstream effect of augmentation on classifier robustness
- The population of interest is not sparse or downsampled, so no augmentation-vs-degradation comparison can be made

## Inputs

- Fresh 68k PBMCs (Donor A) dataset in .h5 or .h5ad format, pre-processed and split into TFRecord format
- cscGAN-generated cluster-2 cells (e.g., .h5ad output from --generate with --cells_no, --model_path, --save_path)
- Cluster labels for cluster 2 vs. other populations for supervised classification

## Outputs

- f1_scores.csv with one row per downsampling level (50% to 0.5%), strategy (downsampled/upsampled/cscGAN-augmented), and random seed
- F1-score curves comparing classifier robustness across the three augmentation strategies

## How to apply

After pre-processing the Fresh 68k PBMCs (Donor A) dataset (.h5/.h5ad) with `python main.py --param parameters. --process` and generating cluster-2 cells with a trained cscGAN via `--generate`, build three population conditions per retention level: artificially downsampled (eight levels from 50% down to 0.5%), upsampled to the original size by random duplication, and cscGAN-augmented. Train a Random Forest classifier on each condition, repeating over five random seeds to capture seed-level variability. Compute the F1 score per retention level, strategy, and seed, and write results to f1_scores.csv. The rationale is that the paper found augmenting sparse cell populations improves classifier robustness: the mean F1 at the 0.5% downsampling rate should reach ~0.45 (downsampled), ~0.6 (upsampled), and ~0.93 (cscGAN-augmented), demonstrating that augmentation almost completely mitigates the downsampling effect.

## Related tools

- **cscGAN** (Conditional GAN used to generate cluster-2 cells that form the augmented condition for classifier robustness testing) — github.com/imsb-uke/scGAN
- **Python** (Implements Random Forest training, downsampling/upsampling of cluster 2, and F1-score computation over five random seeds) — github.com/imsb-uke/scGAN
- **TensorFlow** (Backend for training the cscGAN model whose generated cells supply the augmentation condition) — github.com/imsb-uke/scGAN

## Evaluation signals

- Mean F1 scores at the 0.5% downsampling rate match the reported values of 0.45 (downsampled), 0.6 (upsampled), and 0.93 (cscGAN-augmented) within seed-level variability
- cscGAN-augmented F1 at severe downsampling approaches the classifier performance at the original population size
- F1 score decreases monotonically as the retention rate drops from 50% to 0.5% in the downsampled condition
- Results file f1_scores.csv contains entries for all eight retention levels, three strategies, and five random seeds

## Limitations

- Results depend on five random seeds; seed-level variability means reported F1 values are means, not exact reproductions
- The mechanism enabling cscGAN to generate cells from very small subpopulations (e.g. 16 cells) is only speculated, not established
- Augmentation benefit is demonstrated for classification (F1 robustness); other applications such as clustering, cell type detection, and data denoising are proposed but not demonstrated

## Evidence

- [abstract] Augmenting sparse cell populations with cscGAN generated cells improves downstream analyses such as the detection of marker genes, the robustness and reliability of classifiers: "Augmenting sparse cell populations with cscGAN generated cells improves downstream analyses such as the detection of marker genes, the robustness and reliability of classifiers"
- [other] Compute F1 scores per level, strategy, and seed, and write the results to f1_scores.csv.: [evidence span withheld — anchor status: not_found]
- [other] verify that the mean F1 scores at 0.5% downsampling match the reported values of 0.45 (downsampled), 0.6 (upsampled), and 0.93 (cscGAN-augmented) within seed-level variability: [evidence span withheld — anchor status: not_found]
- [readme] The main.py script is used to start the pre-processing of the files, to start (or resume) the training, and generate cells using a trained model.: "The `main.py` script is used to start the pre-processing of the files, to start (or resume) the training, and generate cells using a trained model."
