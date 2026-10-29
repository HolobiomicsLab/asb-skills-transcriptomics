---
name: simulated-scrna-seq-quality-evaluation
description: Use when you have simulated scRNA-seq cells (e.g., simulated.h5ad from
  GRouNdGAN --generate) and must verify they are realistic and non-collapsed — especially
  on heterogeneous multi-cell-type references like PBMC-All.
when_to_use_negative:
- When the goal is to evaluate a GRN inference method against a ground truth graph
  — use --benchmark_grn instead of the data-quality evaluation
- When you need raw unnormalized count data for downstream comparison — GRouNdGAN
  generates library-size normalized expression values, so count-based quality checks
  are not applicable
- When the reference dataset is very small (e.g., BoneMarrow ~2.7k cells) — simulator
  performance drops on the test set and evaluation results may be uninformative
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GRouNdGAN
  role: 'Primary tool: its --evaluate command computes the t-SNE joint embedding,
    Euclidean/Cosine distances, RF AUROC, MMD, and miLISI metrics with test-set-half
    controls'
  repo: github.com/Emad-COMBINE-lab/GRouNdGAN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:886af286b36a251bde064d8f038c4753d238b8b973d1f8fe1f1a924d5ca0212e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GRouNdGAN
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# simulated-scrna-seq-quality-evaluation

## Summary

Quantitatively and qualitatively assesses whether GRouNdGAN-simulated single-cell RNA-seq data resemble their experimental reference using the --evaluate metric suite (joint t-SNE embedding, Euclidean/Cosine distances of mean profiles, RF AUROC, MMD, miLISI), each also computed against a test-set-half control to detect mode collapse.

## When to use

Use when you have simulated scRNA-seq cells (e.g., simulated.h5ad from GRouNdGAN --generate) and must verify they are realistic and non-collapsed — especially on heterogeneous multi-cell-type references like PBMC-All.

## When NOT to use

- When the goal is to evaluate a GRN inference method against a ground truth graph — use --benchmark_grn instead of the data-quality evaluation
- When you need raw unnormalized count data for downstream comparison — GRouNdGAN generates library-size normalized expression values, so count-based quality checks are not applicable
- When the reference dataset is very small (e.g., BoneMarrow ~2.7k cells) — simulator performance drops on the test set and evaluation results may be uninformative

## Inputs

- Trained GRouNdGAN .pth checkpoint referenced in the [EXPERIMENT]/checkpoint config field
- Preprocessed experimental test split (from --preprocess, e.g., PBMC-All with 1000 highly variable genes)
- Simulated cells in h5ad format (simulated.h5ad from --generate)

## Outputs

- Metric suite results: RF AUROC, MMD, miLISI, Euclidean and Cosine distances between mean expression profiles
- Joint t-SNE plot of experimental and simulated cells
- Control values: same metrics computed between two halves of the experimental test set

## How to apply

Set [EXPERIMENT]/checkpoint to the trained .pth checkpoint, run --generate to simulate cells (e.g., 10000 cells saved as simulated.h5ad), then run --evaluate to compute the metric suite on the jointly embedded experimental and simulated data. Compute each metric (t-SNE of the joint embedding, Euclidean and Cosine distances between mean expression profiles, Random Forest AUROC, MMD, miLISI) between simulated and experimental test cells AND between two halves of the test set as a control, so realistic values have an empirical baseline. Judge success by RF AUROC close to 0.5 (classifier cannot distinguish simulated from experimental cells), miLISI close to 2 (e.g., 1.90 on PBMC-All), MMD ~0.017, and Euclidean/Cosine values comparable to the control; t-SNE should show simulated cells spread across all cell-type clusters, confirming no mode collapse even without cell-type labels.

## Related tools

- **GRouNdGAN** (Primary tool: its --evaluate command computes the t-SNE joint embedding, Euclidean/Cosine distances, RF AUROC, MMD, and miLISI metrics with test-set-half controls) — github.com/Emad-COMBINE-lab/GRouNdGAN

## Evaluation signals

- RF AUROC is close to 0.5, meaning the Random Forest classifier cannot distinguish simulated from experimental cells
- miLISI is close to 2 (PBMC-All test set achieved 1.90), indicating cells cover the heterogeneous cell-type space without mode collapse
- MMD (~0.017 on PBMC-All) and Euclidean/Cosine distances between mean profiles are comparable to the two-halves-of-test-set control values
- The joint t-SNE embedding shows simulated cells intermixed with experimental cells across all clusters/cell types
- Metrics were computed on the test set (not just train set), confirming generalization rather than memorization

## Limitations

- GRouNdGAN generates expression values library-size normalized across the considered genes and TFs, not unnormalized counts, constraining which comparison metrics are meaningful
- Evaluation conclusions are only as good as the trained model; the model must be retrained per new dataset and hyperparameters may need re-tuning via a validation set
- Benchmarking results on heterogeneous datasets like PBMC-All are not conclusive; follow-up studies are required to fully characterize the effect of distinct cell types

## Evidence

- [methods] --evaluate command: "--evaluate       Evaluate the data quality of the simulated dataset"
- [methods] evaluation metrics: "You can evaluate the quality of GRouNdGAN simulations using the following quantitative and qualitative metrics:"
- [abstract] model fidelity claims: "our model captures non-linear TF-gene dependencies and preserves gene identities, cell trajectories, pseudo-time ordering, and technical and biological noise"
- [discussion] normalized output limitation: "the model generates expression values library-size normalized across the considered genes and TFs"
