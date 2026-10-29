---
name: kernel-based-expression-projection
description: Use when a known nuisance signal (cell cycle, sex, spatial zonation,
  batch) encoded in marker genes or cell attributes must be removed from single-cell
  RNA-seq expression while preserving the biological structure of interest.
when_to_use_negative:
- No reliable prior knowledge exists for the signal to filter — SiFT strongly relies
  on the prior knowledge used to define the cell–cell similarity kernel, and an inaccurate
  or coarse-grained mapping may lead to loss of signal
- Downstream tasks require properly log-transformed data — filtering modifies gene
  expression so the output is no longer properly log-transformed
- The goal is batch-effect integration via a joint latent representation rather than
  gene-expression-level correction (methods like scVI or Harmony operate in latent
  space instead)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SiFT (sift-sc)
  role: Implements the kernel computation, projection of expression onto the signal,
    and deduction step (sift.SiFT with sift.kernels.KnnKernel/MappingKernel)
  repo: https://github.com/nitzanlab/sift-sc
- name: scVI
  role: Latent-space integration baseline used to benchmark the kernel-based filtering
    against joint-latent-representation correction
  repo: https://github.com/scverse/scvi-tools
- name: pip
  role: Installation channel for the sift-sc package (pip install sift-sc)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:278134a66d1939dd5170a8347ae6ed4cd9be821db7634e72b40e63a05ad0d7ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - SiFT
  - SiFT (sift-sc)
  - pip
  - scVI
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# kernel-based-expression-projection

## Summary

SiFT's core operation: define a probabilistic cell–cell similarity kernel capturing the biological signal to remove, project the gene expression count matrix onto that signal via matrix multiplication, and deduct the projection to obtain a filtered expression matrix that preserves other biological attributes.

## When to use

Use when a known nuisance signal (cell cycle, sex, spatial zonation, batch) encoded in marker genes or cell attributes must be removed from single-cell RNA-seq expression while preserving the biological structure of interest.

## When NOT to use

- No reliable prior knowledge exists for the signal to filter — SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel, and an inaccurate or coarse-grained mapping may lead to loss of signal
- Downstream tasks require properly log-transformed data — filtering modifies gene expression so the output is no longer properly log-transformed
- The goal is batch-effect integration via a joint latent representation rather than gene-expression-level correction (methods like scVI or Harmony operate in latent space instead)

## Inputs

- Log-normalized single-cell gene expression matrix (AnnData, e.g. Drosophila wing disc dataset)
- Prior knowledge defining the signal to filter: marker gene sets (e.g. a 55-gene cell cycle/sex marker set) or cell attributes/labels (sex, cell cycle phase, spatial coordinates)
- Choice of kernel specification (sift.kernels.KnnKernel, MappingKernel, RBFKernel, or LaplacianKernel)

## Outputs

- Filtered gene expression matrix with the target signal removed (still at gene expression level)
- Cell–cell similarity kernel capturing similarity with respect to the filtered signal
- Downstream evaluation metrics such as iLISI degradation scores per filtered attribute (e.g. degradation_scores.csv)

## How to apply

Start from a log-normalized single-cell expression matrix (AnnData) plus prior knowledge defining the signal to filter, e.g. marker gene sets for cell cycle or sex in Drosophila wing disc data. Choose a kernel via sift.kernels — KnnKernel or MappingKernel when cell attributes/labels define the signal, or RBFKernel/LaplacianKernel for continuous signals — and build the probabilistic cell–cell similarity kernel with sift.SiFT. Compute the projection of the expression count matrix onto the signal through matrix multiplication with the kernel, then deduct this projection from the original expression to obtain the filtered matrix. Rationale: because the kernel captures similarity only with respect to the target signal, the deducted component removes that signal while leaving unrelated biological variation intact, performing correction at the gene expression level rather than in a joint latent representation. Verify by computing integration metrics such as iLISI on held-out attributes (e.g. iLISI degradation of −0.6% for cell cycle, −4.2% for cell cycle+sex in the partial-marker robustness experiment) and comparing against regress_out and scVI baselines.

## Related tools

- **SiFT (sift-sc)** (Implements the kernel computation, projection of expression onto the signal, and deduction step (sift.SiFT with sift.kernels.KnnKernel/MappingKernel)) — https://github.com/nitzanlab/sift-sc
- **scVI** (Latent-space integration baseline used to benchmark the kernel-based filtering against joint-latent-representation correction) — https://github.com/scverse/scvi-tools
- **pip** (Installation channel for the sift-sc package (pip install sift-sc))

## Examples

```
pip install sift-sc
```

## Evaluation signals

- iLISI percentage degradation of the filtered attribute relative to the original marker set is small (target: SiFT −0.6% for cell cycle, −4.2% for cell cycle+sex in the Drosophila partial-marker experiment)
- Filtered data preserves biological attributes while removing the target signal — non-target attributes show minimal iLISI degradation compared to regress_out (−1.0%/−5.9%) and scVI (−13.5%/−16.3%)
- Projection step is reproducible across repetitions (e.g. mean degradation over 10 repetitions is stable)
- Output matrix retains the same genes × cells dimensions as the input, with the signal-specific component subtracted rather than a latent embedding returned

## Limitations

- Lacks a ground truth reference for evaluating filtering results; integration metrics used for evaluation may be application-specific
- Filtering modifies gene expression so data is no longer properly log-transformed, which certain downstream analysis tasks expect
- Statistical implications of applying commonly used analysis tools to corrected gene expression data are unexplored
- An inaccurate or coarse-grained kernel/mapping may lead to a loss of signal

## Evidence

- [intro] we compute a probabilistic cell–cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter: "we compute a probabilistic cell–cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter"
- [intro] we obtain a projection of the expression count matrix onto the signal we seek to filter using matrix multiplication: "we obtain a projection of the expression count matrix onto the signal we seek to filter using matrix multiplication"
- [discussion] This kernel is then used to obtain a projection of gene expression onto the signal, which is then removed from the original expression.: "This kernel is then used to obtain a projection of gene expression onto the signal, which is then removed from the original expression."
- [methods] Install `SiFT` in one of the following ways: Through **pip**:: pip install sift-sc: "Install `SiFT` in one of the following ways: Through **pip**:: pip install sift-sc"
- [methods] sift.kernels.RBFKernel sift.kernels.LaplacianKernel sift.kernels.KnnKernel sift.kernels.MappingKernel: "sift.kernels.RBFKernel
    sift.kernels.LaplacianKernel
    sift.kernels.KnnKernel
    sift.kernels.MappingKernel"
- [readme] Signal FilTering is a tool for uncovering hidden biological processes in single-cell data.: "Signal FilTering is a tool for uncovering hidden biological processes in single-cell data."
