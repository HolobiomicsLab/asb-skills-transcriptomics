---
name: single-cell-rna-seq-simulation
description: Use when benchmarking GRN inference algorithms on scRNA-seq data and
  reliable ground truth is needed, or when realistic in-silico scRNA-seq data with
  imposed TF-gene regulatory dynamics is required.
when_to_use_negative:
- You need unnormalized count output or multi-layer GRNs / TF-gene-phenotype causal
  graphs — the current model generates only library-size normalized expression and
  supports only bipartite GRNs
- You need a predictive model that adapts to new data without retraining — GRouNdGAN
  is generative and must be retrained for each new dataset
- Reference dataset is very small (e.g., ~2.7k cells BoneMarrow) — simulator performance
  drops on the test set relative to training
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: GRouNdGAN
  role: Causal implicit GAN that imposes a user/GRNBoost2-derived GRN as the data-generating
    graph on simulated scRNA-seq data
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers the regulatory edges (top 15 TFs per gene) from the preprocessed reference
    data that are imposed as the causal graph
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: BEELINE
  role: Reference benchmark whose curated-benchmark method ordering the GRouNdGAN-simulated
    benchmarking is compared against
  repo: https://github.com/Murali-group/Beeline
- name: pyenv
  role: Manages Python version/environment for running the GRouNdGAN pipeline
  repo: https://github.com/pyenv/pyenv
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:fdf5c3b339412da35b5c42f7a93a73f98f8dbd8f9452d2821dbceee2e63bf3d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - BEELINE
  - GRNBoost2
  - GRouNdGAN
  - pyenv
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-rna-seq-simulation

## Summary

Simulate realistic single-cell RNA-seq data whose inter-gene relationships follow a user-supplied (or GRNBoost2-inferred) causal GRN using GRouNdGAN, a GRN-guided causal implicit generative model, so that downstream GRN inference methods can be benchmarked against gold-standard ground truth.

## When to use

Use when benchmarking GRN inference algorithms on scRNA-seq data and reliable ground truth is needed, or when realistic in-silico scRNA-seq data with imposed TF-gene regulatory dynamics is required.

## When NOT to use

- You need unnormalized count output or multi-layer GRNs / TF-gene-phenotype causal graphs — the current model generates only library-size normalized expression and supports only bipartite GRNs
- You need a predictive model that adapts to new data without retraining — GRouNdGAN is generative and must be retrained for each new dataset
- Reference dataset is very small (e.g., ~2.7k cells BoneMarrow) — simulator performance drops on the test set relative to training

## Inputs

- Experimental reference scRNA-seq dataset (e.g., BoneMarrow GSE72857, PBMC-ALL 10x Genomics fresh 68k PBMC Donor A, Dahlin GSE107727, or a cell-type subset like PBMC-CTL)
- Raw count matrix plus config file with preprocessing arguments
- Optionally, a user-defined bipartite GRN to impose instead of the GRNBoost2-inferred one

## Outputs

- GRouNdGAN-simulated scRNA-seq expression matrix (e.g., 100k cells x 1000 genes) with library-size normalized values
- Causal ground truth GRN (exportable CSV via [GRN Benchmarking]/ground truth save path)
- Trained model .pth checkpoint
- Simulated-data quality evaluation report

## How to apply

Run the GRouNdGAN pipeline (docker pull yazdanz/groundgan:4b98686) on a reference scRNA-seq dataset: first --preprocess the raw counts, then --create_grn to infer a GRN from the preprocessed training set with GRNBoost2 (each gene is regulated by its top 15 identified TFs) and format it as a causal graph (or supply a custom GRN instead, skipping --create_grn). Then --train the GAN on the BoneMarrow/PBMC-style reference (e.g., GSE72857 or 10x PBMC Donor A) and --generate 100k cells x 1000 genes of in-silico data using the populated .pth checkpoint. Validate realism with --evaluate (gene identities, cell trajectories, pseudo-time ordering preserved) and optionally --perturb for in-silico TF knockout sampling from interventional distributions. The causal graph imposed on the generator becomes the ground truth GRN for --benchmark_grn evaluation against inferred networks.

## Related tools

- **GRouNdGAN** (Causal implicit GAN that imposes a user/GRNBoost2-derived GRN as the data-generating graph on simulated scRNA-seq data) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers the regulatory edges (top 15 TFs per gene) from the preprocessed reference data that are imposed as the causal graph) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **BEELINE** (Reference benchmark whose curated-benchmark method ordering the GRouNdGAN-simulated benchmarking is compared against) — https://github.com/Murali-group/Beeline
- **pyenv** (Manages Python version/environment for running the GRouNdGAN pipeline) — https://github.com/pyenv/pyenv

## Examples

```
git clone https://github.com/Emad-COMBINE-lab/GRouNdGAN.git; docker pull yazdanz/groundgan:4b98686
```

## Evaluation signals

- Simulated data passes --evaluate quality checks: gene identities, cell trajectories, pseudo-time ordering, and technical/biological noise preserved relative to the reference
- Ground truth GRN exports as a CSV whose edges match the imposed causal graph (one edge per gene from each of its top 15 GRNBoost2 TFs)
- The trained model checkpoint (.pth) is populated in the [EXPERIMENT]/checkpoint field before --generate runs without error
- Benchmarking an inferred GRN with --benchmark_grn yields precision-at-k (k = 15) per gene and a global PR curve with AUPRC, including a random-predictor baseline AUPRC
- Simulated dataset dimensions match the configured output (e.g., 100k cells x 1000 genes) for BoneMarrow/PBMC references

## Limitations

- Only library-size normalized expression is generated; unnormalized counts are planned for future iterations
- Only bipartite GRNs are supported; multi-layer GRNs and TF-gene-phenotype causal graphs are not supported yet
- Resemblance to the reference deteriorates if imposed TF-gene relationships differ significantly from patterns in the reference data
- The underlying causal graph of an observational dataset is not necessarily unique; hyperparameters may need re-tuning on a validation set for datasets with very different gene/TF counts
- Benchmarking on mixed cell-type data (e.g., PBMC-All) is not conclusive; distinct cell types need characterization in follow-up studies

## Evidence

- [abstract] GRN-guided simulation pipeline: "We introduce GRouNdGAN, a gene regulatory network (GRN)-guided reference-based causal implicit generative model for simulating single-cell RNA-seq data"
- [discussion] Recommended edge-finding strategy: "we recommend first finding a set of regulatory edges from the reference dataset (e.g., using GRNBoost2 or even other data modalities) and imposing a selected subset of those in the simulated data"
- [methods] create_grn command: "--create_grn     Infer a GRN from preprocessed data using GRNBoost2 and appropriately format as causal graph"
- [methods] Benchmark command: "--benchmark_grn  Evaluate the performance of a GRN inference method in inferring the ground truth GRN"
- [discussion] Normalization caveat: "the model generates expression values library-size normalized across the considered genes and TFs"
