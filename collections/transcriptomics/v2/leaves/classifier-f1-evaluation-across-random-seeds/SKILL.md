---
name: classifier-f1-evaluation-across-random-seeds
description: Use when testing whether augmenting a sparse or artificially downsampled
  cell population (e.g. PBMC cluster 2 at retention levels from 50% to 0.5%) improves
  classifier performance relative to downsampled or randomly duplicated baselines.
when_to_use_negative:
- When the original population is already large and not sparse — augmentation gains
  for classifier robustness are demonstrated specifically for sparse/downsampled subpopulations
- When the classifier task is not cell-type/subpopulation classification (e.g. trajectory
  inference or clustering), since downstream benefits there are proposed but not demonstrated
  in the article
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: cscGAN
  role: Conditional GAN used to generate realistic cells for the target cluster to
    augment the downsampled training population
  repo: github.com/imsb-uke/scGAN
- name: Python
  role: Language in which downsampling, Random Forest training, F1 computation, and
    CSV export are implemented (tested with Python 3.5)
- name: TensorFlow
  role: Framework backing cscGAN training and cell generation prior to classifier
    evaluation
  repo: github.com/imsb-uke/scGAN
- name: Docker
  role: Containerized environment (scgan image) to reproduce the training and evaluation
    setup
  repo: github.com/imsb-uke/scGAN
- name: Jupyter
  role: Notebook used to convert source data to .h5/.h5ad before preprocessing
  repo: github.com/imsb-uke/scGAN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-14018-z
    title: Realistic in silico generation and augmentation of single-cell RNA-seq
      data using generative adversarial networks
  source_content_hash: sha256:30fbb89a57b8b31f7bbabb6292142cb11475626a7ad523469659a48fffc937ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - Jupyter
  - Python
  - TensorFlow
  - cscGAN
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Classifier F1 evaluation across random seeds

## Summary

Protocol for quantifying how data augmentation (e.g. with cscGAN-generated cells) affects downstream classifier robustness by training a Random Forest classifier per downsampling level, augmentation strategy, and random seed, and comparing mean F1 scores.

## When to use

Use when testing whether augmenting a sparse or artificially downsampled cell population (e.g. PBMC cluster 2 at retention levels from 50% to 0.5%) improves classifier performance relative to downsampled or randomly duplicated baselines.

## When NOT to use

- When the original population is already large and not sparse — augmentation gains for classifier robustness are demonstrated specifically for sparse/downsampled subpopulations
- When the classifier task is not cell-type/subpopulation classification (e.g. trajectory inference or clustering), since downstream benefits there are proposed but not demonstrated in the article

## Inputs

- Preprocessed single-cell RNA-seq data in .h5/.h5ad format (e.g. Fresh 68k PBMCs Donor A)
- cscGAN-generated cells for the target cluster (e.g. cluster 2), saved as .h5ad
- Downsampling retention levels (50% to 0.5%) and upsampled (random duplication) baseline populations
- Random seed definitions (e.g. five seeds)

## Outputs

- f1_scores.csv with F1 scores per downsampling level, augmentation strategy, and seed
- Mean F1 score comparisons across strategies (e.g. 0.45 vs 0.6 vs 0.93 at 0.5% retention)

## How to apply

After generating augmented cells (e.g. cscGAN output saved as .h5ad), downsample the target subpopulation (cluster 2 of the Fresh 68k PBMCs Donor A dataset) to eight retention levels from 50% down to 0.5%. For each level, train a Random Forest classifier under three strategies: downsampled-only, upsampled by random duplication to original size, and cscGAN-augmented. Repeat the entire procedure over five random seeds to capture seed-level variability, compute the F1 score per level-strategy-seed combination, and write the results to f1_scores.csv. Judge correctness by checking that mean F1 scores at 0.5% downsampling approximate the reported values of 0.45 (downsampled), 0.6 (upsampled), and 0.93 (cscGAN-augmented), within seed-level variability, demonstrating that conditional GAN augmentation nearly fully restores classifier robustness.

## Related tools

- **cscGAN** (Conditional GAN used to generate realistic cells for the target cluster to augment the downsampled training population) — github.com/imsb-uke/scGAN
- **Python** (Language in which downsampling, Random Forest training, F1 computation, and CSV export are implemented (tested with Python 3.5))
- **TensorFlow** (Framework backing cscGAN training and cell generation prior to classifier evaluation) — github.com/imsb-uke/scGAN
- **Docker** (Containerized environment (scgan image) to reproduce the training and evaluation setup) — github.com/imsb-uke/scGAN
- **Jupyter** (Notebook used to convert source data to .h5/.h5ad before preprocessing) — github.com/imsb-uke/scGAN

## Examples

```
python main.py --param parameters. --generate --cells_no 1000 500 0 200 --model_path path/to/my/model --save_path where_to_save.h5ad
```

## Evaluation signals

- Mean F1 at 0.5% downsampling is ~0.45 for downsampled, ~0.6 for upsampled, and ~0.93 for cscGAN-augmented, within seed-level variability
- F1 curves are computed for all 8 retention levels × 3 strategies × 5 seeds and complete rows are present in f1_scores.csv
- cscGAN-augmented F1 at low retention approaches the undownsampled baseline (augmentation 'almost completely mitigates' downsampling)
- Per-seed spread is reported so variance across the five seeds can be inspected rather than a single point estimate

## Limitations

- Results depend on the trained generative model; the mechanism enabling cscGAN to generate cells from very small subpopulations (e.g. 16 cells) is only speculated, not established
- Evaluation is specific to Random Forest classification of PBMC cluster 2; other downstream tasks (clustering, cell type detection, denoising) are proposed but not demonstrated
- Seed-level variability means single-seed F1 values may deviate from reported means

## Evidence

- [abstract] Augmenting sparse cell populations with cscGAN generated cells improves downstream analyses such as the detection of marker genes, the robustness and reliability of classifiers: "Augmenting sparse cell populations with cscGAN generated cells improves downstream analyses such as the detection of marker genes, the robustness and reliability of classifiers"
- [methods] most of the experiments reported in our paper use the Fresh 68k PBMCs (Donor A) dataset: "most of the experiments reported in our paper use the [Fresh 68k PBMCs (Donor A) dataset](http://cf.10xgenomics.com/samples/cell-exp/1.1.0/fresh_68k_pbmc_donor_a/fresh_68k_pbmc_donor_a_filtered_gene_b"
- [readme] To generate cells and write them to a file, you can use the --generate flag: "To generate cells and write them to a file, you can use the `--generate` flag. In that case, also use the `--cells_no` flag with a list of integers corresponding to the number of cells"
- [readme] The code was tested with Python 3.5: "The code was tested with Python 3.5 and the packages listed in [requirements.txt](requirements.txt)."
