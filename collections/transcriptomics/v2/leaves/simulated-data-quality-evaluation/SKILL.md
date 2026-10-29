---
name: simulated-data-quality-evaluation
description: Use when a .pth-trained GRouNdGAN checkpoint has produced simulated.h5ad
  cells and you must verify resemblance to held-out experimental cells (e.g., PBMC-CTL,
  Dahlin, Tumor-ALL) via cosine, Euclidean, MMD, RF AUROC, and miLISI metrics.
when_to_use_negative:
- When evaluating how well a GRN inference method reconstructs a ground truth network
  — use the --benchmark_grn command instead, which evaluates the reconstructed GRN
  against the ground truth GRN
- When performing in silico TF perturbation or knockout experiments — use the --perturb
  command, not data-quality evaluation
- When the goal is unnormalized raw count simulation — GRouNdGAN only generates expression
  values library-size normalized across the considered genes and TFs, so count-level
  quality assessment is out of scope
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GRouNdGAN
  role: Provides the --evaluate command that computes Euclidean, cosine, MMD, RF AUROC,
    and miLISI metrics between simulated and real data, with optional t-SNE plotting
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers the causal GRN on the preprocessed train set upstream of training;
    the quality of the imposed GRN constrains how closely simulated data resembles
    the reference, which the evaluation metrics detect
- name: Docker (yazdanz/groundgan:4b98686)
  role: Container runtime environment in which the --evaluate command is executed
    reproducibly
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:7df52342a8abaca460990ef7d4035b00073470deffec38f6dae15902b757df1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - Docker
  - Docker (yazdanz/groundgan:4b98686)
  - GRNBoost2
  - GRouNdGAN
  - SPARSIM
  - TensorBoard
  - scDESIGN2
  - scGAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# simulated-data-quality-evaluation

## Summary

Quantitatively validates GRouNdGAN-simulated scRNA-seq data against held-out real cells by running the repository's --evaluate command, which computes cosine distance, Euclidean distance, MMD, RF AUROC, and miLISI between simulated and experimental reference profiles.

## When to use

Use when a .pth-trained GRouNdGAN checkpoint has produced simulated.h5ad cells and you must verify resemblance to held-out experimental cells (e.g., PBMC-CTL, Dahlin, Tumor-ALL) via cosine, Euclidean, MMD, RF AUROC, and miLISI metrics.

## When NOT to use

- When evaluating how well a GRN inference method reconstructs a ground truth network — use the --benchmark_grn command instead, which evaluates the reconstructed GRN against the ground truth GRN
- When performing in silico TF perturbation or knockout experiments — use the --perturb command, not data-quality evaluation
- When the goal is unnormalized raw count simulation — GRouNdGAN only generates expression values library-size normalized across the considered genes and TFs, so count-level quality assessment is out of scope

## Inputs

- simulated.h5ad produced by --generate from a trained .pth checkpoint (1000–10000 cells, 1000 highly variable genes, library-size normalized to 20000)
- Held-out preprocessed reference test split (e.g., 1000 test cells set aside by --preprocess)
- causal_gan.cfg configuration file with checkpoint path and evaluation flags (plot tsne, Euclidean distance, cosine distance, RF AUROC, MMD, miLISI)
- GRouNdGAN Docker image yazdanz/groundgan:4b98686 or locally cloned repository environment

## Outputs

- evaluation_metrics.csv with cosine distance, Euclidean distance, MMD, RF AUROC, and miLISI for simulated-vs-real and real-real control comparisons across training and test splits
- t-SNE plot overlaying simulated and real cells
- Real-real control values computed between two halves of the reference test set

## How to apply

Populate the [EXPERIMENT]/checkpoint field of causal_gan.cfg with the final .pth checkpoint, generate cells (e.g., 1000 for PBMC-CTL or 10000 for the Dahlin/Tumor datasets), then run the --evaluate command with plot tsne, Euclidean distance, cosine distance, RF AUROC, MMD, and miLISI all enabled. The command computes distances and MMD between mean profiles of simulated cells and 1000 held-out test cells, trains a random forest to distinguish real from simulated (AUROC near 0.5 is good), and computes miLISI (near 2 is good) alongside a t-SNE plot. Always compute the internal real-real control between two halves of the reference test set, since acceptability is judged by whether simulated-vs-real values are comparable to or lower than this control (demo: Euclidean 171.2 vs 242.8 control; MMD 0.030 vs 0.062 control; miLISI 1.897 vs 1.890 control). Collect cosine, Euclidean, MMD, RF AUROC, and miLISI values for training and test splits into evaluation_metrics.csv, accepting results when they mirror the paper's expected output (PBMC-CTL reference: cosine 0.00057, Euclidean 182, MMD 0.026, RF AUROC 0.54, miLISI 1.891).

## Related tools

- **GRouNdGAN** (Provides the --evaluate command that computes Euclidean, cosine, MMD, RF AUROC, and miLISI metrics between simulated and real data, with optional t-SNE plotting) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers the causal GRN on the preprocessed train set upstream of training; the quality of the imposed GRN constrains how closely simulated data resembles the reference, which the evaluation metrics detect)
- **Docker (yazdanz/groundgan:4b98686)** (Container runtime environment in which the --evaluate command is executed reproducibly)

## Examples

```
docker pull yazdanz/groundgan:4b98686 && python . --evaluate causal_gan.cfg
```

## Evaluation signals

- RF AUROC for distinguishing real from simulated cells is close to 0.5 (PBMC-CTL reference: 0.54)
- miLISI is close to 2 (PBMC-CTL reference: 1.891; demo 1.897 vs 1.890 real-real control)
- Euclidean, cosine, and MMD distances between mean real and simulated profiles are comparable to or lower than the real-real control computed between two halves of the test set (demo: Euclidean 171.2 vs 242.8 control; MMD 0.030 vs 0.062 control; cosine 0.00057)
- Test-split metrics show only a small degradation relative to training-split metrics (Table 3 pattern across Dahlin, Tumor-All, and Tumor-malignant datasets)
- evaluation_metrics.csv is populated with all five metrics for both training and test splits of each dataset

## Limitations

- On small reference datasets (BoneMarrow, ~2.7k cells), performance of all simulators dropped on the test set compared to the training set, so train-split metrics alone overstate quality
- The model only generates library-size normalized expression values across the considered genes and TFs; unnormalized count generation is not supported, limiting count-level fidelity checks
- Hyperparameters may need re-tuning via a validation set for new datasets or datasets with very different gene/TF counts, which changes the expected metric ranges
- GRouNdGAN is generative, not predictive — the model must be retrained for each new dataset before evaluation is meaningful

## Evidence

- [methods] GRouNdGAN evaluate command: "--evaluate       Evaluate the data quality of the simulated dataset"
- [methods] Evaluation metrics listed: "You can evaluate the quality of GRouNdGAN simulations using the following quantitative and qualitative metrics:"
- [abstract] Model scope statement: "We introduce GRouNdGAN, a gene regulatory network (GRN)-guided reference-based causal implicit generative model for simulating single-cell RNA-seq data"
- [discussion] Small-dataset caveat: "In the BoneMarrow dataset (which had only ~2.7k cells), the performance of all simulators dropped on the test set"
- [discussion] Normalization restriction: "the model generates expression values library-size normalized across the considered genes and TFs"
