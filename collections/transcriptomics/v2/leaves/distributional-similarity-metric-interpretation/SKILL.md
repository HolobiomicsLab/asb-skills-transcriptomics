---
name: distributional-similarity-metric-interpretation
description: Use when evaluating GRouNdGAN-simulated single-cell RNA-seq data (e.g.,
  10000 cells from PBMC-All) and you need thresholds for deciding if RF AUROC ~0.5,
  miLISI ~2, and MMD/Cosine values comparable to the two-halves test-set control indicate
  realistic generation.
when_to_use_negative:
- Comparing simulated data to training-set metrics instead of the held-out test set,
  which overstates realism
- Benchmarking a GRN inference method's accuracy against ground truth — use the --benchmark_grn
  evaluation instead
- Assessing interventional (knockout) sampling outcomes, where resemblance to the
  observational reference is intentionally altered by imposed TF-gene perturbations
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: GRouNdGAN
  role: 'Computes the metric suite via --evaluate: t-SNE joint embedding, Euclidean
    and Cosine distances between mean expression profiles, Random Forest AUROC, MMD,
    and miLISI, each also computed between two halves of the test set as a control'
  repo: github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers the GRN used to train the model being evaluated, but is not itself
    part of metric interpretation
  repo: github.com/Emad-COMBINE-lab/GRouNdGAN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:78b9af030ab0c0112f50d65502b8af279e6e55187b8cf6bd013e5209071e799f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GRNBoost2
  - GRouNdGAN
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Distributional similarity metric interpretation

## Summary

Interprets GRouNdGAN's simulated-data quality metric suite — Random Forest AUROC, MMD, miLISI, and Euclidean/Cosine distances between mean expression profiles — against test-set controls to judge whether simulated scRNA-seq cells realistically recapitulate the heterogeneous reference distribution without mode collapse.

## When to use

Use when evaluating GRouNdGAN-simulated single-cell RNA-seq data (e.g., 10000 cells from PBMC-All) and you need thresholds for deciding if RF AUROC ~0.5, miLISI ~2, and MMD/Cosine values comparable to the two-halves test-set control indicate realistic generation.

## When NOT to use

- Comparing simulated data to training-set metrics instead of the held-out test set, which overstates realism
- Benchmarking a GRN inference method's accuracy against ground truth — use the --benchmark_grn evaluation instead
- Assessing interventional (knockout) sampling outcomes, where resemblance to the observational reference is intentionally altered by imposed TF-gene perturbations

## Inputs

- GRouNdGAN --evaluate output: metric suite (t-SNE joint embedding, Euclidean distance, Cosine distance, Random Forest AUROC, MMD, miLISI) for simulated.h5ad vs test split
- Control metric values computed between two halves of the experimental test set

## Outputs

- Verdict on simulation realism: whether RF AUROC ~0.5, miLISI ~2, and Euclidean/Cosine/MMD comparable to the control indicate realistic generation without mode collapse
- Flagging of mode collapse when simulated cells fail to cover distinct cell types/clusters

## How to apply

After running GRouNdGAN's --evaluate command on simulated.h5ad against the experimental test split, compare each metric to the control comparison computed between two halves of the test set, which represents the best achievable agreement between real samples. Interpret a Random Forest AUROC close to 0.5 as evidence the classifier cannot distinguish simulated from experimental cells; miLISI close to 2 as evidence of mixing across cell types (critical for heterogeneous datasets like PBMC-All with eleven cell types); and Euclidean, Cosine, and MMD values comparable to the control as evidence the simulated mean expression profile matches the reference. High RF AUROC, low miLISI, or outlier MMD/Cosine values indicate mode collapse or unrealistic generation. On heterogeneous inputs, the key validation is the no-mode-collapse result: GRouNdGAN trained without cell-type or cluster information achieved miLISI 1.90 and MMD 0.017 with RF AUROC 0.53 on PBMC-All, consistent with cells being produced across all distinct clusters.

## Related tools

- **GRouNdGAN** (Computes the metric suite via --evaluate: t-SNE joint embedding, Euclidean and Cosine distances between mean expression profiles, Random Forest AUROC, MMD, and miLISI, each also computed between two halves of the test set as a control) — github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers the GRN used to train the model being evaluated, but is not itself part of metric interpretation) — github.com/Emad-COMBINE-lab/GRouNdGAN

## Examples

```
python groundgan.py --evaluate  # computes t-SNE, Euclidean/Cosine distances, RF AUROC, MMD, miLISI for simulated.h5ad vs test set, plus two-halves control
```

## Evaluation signals

- Random Forest AUROC close to 0.5 — the classifier cannot distinguish simulated from experimental cells (PBMC-All reference: 0.53)
- miLISI close to 2 on the test set (PBMC-All reference: 1.90), indicating simulated cells mix across the eleven cell types rather than collapsing
- MMD value comparable to the two-halves-of-test-set control (PBMC-All reference: 0.017)
- Euclidean and Cosine distances between mean expression profiles comparable to the control comparison
- t-SNE of jointly embedded experimental and simulated cells shows overlap across all distinct clusters, confirming no mode collapse

## Limitations

- Benchmarking results on PBMC-All are not conclusive; follow-up studies are required to fully characterize the effect of distinct cell types on the performance of GRN inference methods
- In the BoneMarrow dataset (which had only ~2.7k cells), the performance of all simulators dropped on the test set, so metric thresholds may not transfer to small datasets
- Resemblance to the reference dataset deteriorates if the imposed TF-gene relationships are significantly different from those consistent with the patterns in the reference dataset

## Evidence

- [methods] RF AUROC close to 0.5 (classifier cannot distinguish simulated from experimental cells), miLISI close to 2, and Euclidean/Cosine/MMD values comparable to the control comparison: [evidence span withheld — anchor status: not_found]
- [results] Without receiving cell type or cluster information, GRouNdGAN did not exhibit mode collapse on PBMC-All and generated cells from distinct clusters, achieving a test set miLISI of 1.90.: [evidence span withheld — anchor status: not_found]
- [readme] --evaluate       Evaluate the data quality of the simulated dataset: "--evaluate       Evaluate the data quality of the simulated dataset"
- [readme] You can evaluate the quality of GRouNdGAN simulations using the following quantitative and qualitative metrics:: "You can evaluate the quality of GRouNdGAN simulations using the following quantitative and qualitative metrics:"
