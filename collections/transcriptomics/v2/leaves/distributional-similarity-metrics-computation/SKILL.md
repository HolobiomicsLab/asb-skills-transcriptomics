---
name: distributional-similarity-metrics-computation
description: Use when GRouNdGAN-simulated scRNA-seq cells (e.g., 10,000 generated
  from a Dahlin, Tumor-All, or Tumor-malignant checkpoint) must be compared against
  the real reference data on both training and test splits.
when_to_use_negative:
- No trained checkpoint exists yet — run --preprocess, --create_grn (GRNBoost2, top
  k=15 TFs per gene), and --train (batch size 1024, critic iterations 5, latent dim
  128, lambda 10) before evaluating.
- Evaluating on unnormalized count data — GRouNdGAN only generates expression values
  library-size normalized across the considered genes and TFs.
- Small reference datasets (~2.7k cells like BoneMarrow), where simulator performance
  is known to drop on the test set, so test-split metrics may be uninformative.
edam_topics:
- http://edamontology.org/topic_3384
- http://edamontology.org/topic_0203
tools:
- name: GRouNdGAN
  role: Provides the --generate command (simulated cells from a .pth checkpoint) and
    the --evaluate command that computes the resemblance metrics, RF AUROC, and miLISI
    with t-SNE plotting
  repo: github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Used upstream to infer the causal GRN (top 15 TFs per gene) that guides the
    trained model whose simulations are then evaluated
- name: TensorBoard
  role: Monitors training progress so evaluation is only run once a final checkpoint
    is available
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:6eb404eefbac830177b17d7e849c8efc7abb256d854494485362c28656b57a50
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GRNBoost2
  - GRouNdGAN
  - TensorBoard
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3384
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# distributional-similarity-metrics-computation

## Summary

Quantifies how closely GRouNdGAN-simulated single-cell RNA-seq data resembles its experimental reference using five metrics — Euclidean and cosine distance between mean cell profiles, MMD, random-forest AUROC for distinguishing real from fake cells, and miLISI — mirroring the paper's Table 3 evaluation.

## When to use

Use when GRouNdGAN-simulated scRNA-seq cells (e.g., 10,000 generated from a Dahlin, Tumor-All, or Tumor-malignant checkpoint) must be compared against the real reference data on both training and test splits.

## When NOT to use

- No trained checkpoint exists yet — run --preprocess, --create_grn (GRNBoost2, top k=15 TFs per gene), and --train (batch size 1024, critic iterations 5, latent dim 128, lambda 10) before evaluating.
- Evaluating on unnormalized count data — GRouNdGAN only generates expression values library-size normalized across the considered genes and TFs.
- Small reference datasets (~2.7k cells like BoneMarrow), where simulator performance is known to drop on the test set, so test-split metrics may be uninformative.

## Inputs

- 10,000 GRouNdGAN-simulated cells (.csv) generated from a trained .pth checkpoint on the 1000 highly variable genes
- Preprocessed experimental reference data (library size 20000, min cells 3, min genes 10) split into train and test sets
- Evaluation config file with [EXPERIMENT]/checkpoint path and evaluation metrics settings

## Outputs

- evaluation_metrics.csv with cosine distance, Euclidean distance, MMD, RF AUROC, and miLISI for training and test splits
- t-SNE plots colored by real vs simulated cells (qualitative data-quality check)

## How to apply

After generating 10,000 simulated cells per dataset with the --generate command and the final .pth checkpoint, run the --evaluate command with t-SNE plotting enabled to compute Euclidean distance, cosine distance, and MMD between the mean profiles of real vs simulated cells, the RF AUROC for how well a random forest can distinguish real from simulated cells, and miLISI. Compute each metric on both the training and the held-out test split of the reference dataset (e.g., Dahlin GSE107727, Tumor-ALL from Han et al. 2022, Tumor-malignant from cellxgene follicular lymphoma biopsies) and collect the values into evaluation_metrics.csv. Judgment is relative to an internal control computed between two halves of the reference test set: simulated-vs-real values should approach that control, i.e., low distances/MMD, AUROC near 0.5, and miLISI near 2, as reported in the paper's expected output.

## Related tools

- **GRouNdGAN** (Provides the --generate command (simulated cells from a .pth checkpoint) and the --evaluate command that computes the resemblance metrics, RF AUROC, and miLISI with t-SNE plotting) — github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Used upstream to infer the causal GRN (top 15 TFs per gene) that guides the trained model whose simulations are then evaluated)
- **TensorBoard** (Monitors training progress so evaluation is only run once a final checkpoint is available)

## Evaluation signals

- Real-vs-fake Euclidean/cosine distance and MMD are comparable to the internal control computed between two halves of the reference test set (low distances/MMD).
- RF AUROC for distinguishing real from simulated cells is near 0.5.
- miLISI value is near 2.
- Only a small gap between training-split and test-split metrics, as reported in the paper's Table 3 for Dahlin (n=44,802), Tumor-All (n=136,147), and Tumor-malignant (n=89,203).
- All five metric values per dataset and split are present in evaluation_metrics.csv.

## Limitations

- Evaluation is on library-size-normalized expression across the 1000 HVGs only; unnormalized count generation is planned for future iterations.
- Simulated data resemble the reference only if the imposed TF-gene relationships are consistent with reference data patterns; resemblance deteriorates otherwise.
- Hyperparameters may need re-tuning via a validation set for new datasets or datasets with very different gene/TF counts, which changes what 'good' metric values look like.
- Metrics must be interpreted per split: all simulators dropped on the test set for the small BoneMarrow dataset.

## Evidence

- [methods] evaluate command: "--evaluate       Evaluate the data quality of the simulated dataset"
- [methods] metrics list: "You can evaluate the quality of GRouNdGAN simulations using the following quantitative and qualitative metrics:"
- [methods] checkpoint setup: "One training is done, populate the ``[EXPERIMENT]/checkpoint`` field with the path of the ``.pth`` checkpoint"
- [discussion] small-dataset caveat: "In the BoneMarrow dataset (which had only ~2.7k cells), the performance of all simulators dropped on the test set"
- [discussion] normalization caveat: "the model generates expression values library-size normalized across the considered genes and TFs"
