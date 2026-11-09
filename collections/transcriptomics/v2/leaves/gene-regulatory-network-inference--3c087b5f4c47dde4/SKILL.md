---
name: gene-regulatory-network-inference
description: Use when a raw or preprocessed scRNA-seq reference dataset needs a TF-gene
  regulatory graph to guide causal GAN training (GRouNdGAN) or to serve as ground
  truth for benchmarking GRN inference methods.
when_to_use_negative:
- You need a multi-layer GRN or a TF-gene-phenotype causal graph — the current GRouNdGAN
  version only supports imposing a bipartite GRN
- The dataset contains significantly more genes/TFs than the reference configurations
  — hyperparameters (including GRN choices) may need re-tuning on a validation set
  rather than reusing defaults
- You want GRN benchmark conclusions on a heterogeneous dataset like PBMC-All without
  follow-up analysis — the article states those results are not conclusive regarding
  effects of distinct cell types
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: GRouNdGAN
  role: Provides the --create_grn, --benchmark_grn, and ground-truth export commands
    that wrap GRNBoost2 inference, causal-graph formatting, and GRN benchmarking
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers the GRN from the preprocessed train set; top k = 15 regulating TFs
    per gene are retained for the causal graph
- name: Beeline
  role: Reference benchmarking framework whose experimental GRN-inference results
    GRouNdGAN's benchmark results are compared against
  repo: https://github.com/Murali-group/Beeline
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:cbf077f4bdf0722c02c5853a39bb0db2c5a9dac3267446ad7cce602593c82e02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - Beeline
  - GRNBoost2
  - GRouNdGAN
  - TensorBoard
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gene-regulatory-network-inference

## Summary

Infer a gene regulatory network (GRN) from preprocessed single-cell RNA-seq data using GRNBoost2, keeping the top k=15 regulating transcription factors per gene, and format the result as the causal graph consumed by GRouNdGAN for GRN-guided simulation and GRN-inference benchmarking.

## When to use

Use when a raw or preprocessed scRNA-seq reference dataset needs a TF-gene regulatory graph to guide causal GAN training (GRouNdGAN) or to serve as ground truth for benchmarking GRN inference methods.

## When NOT to use

- You need a multi-layer GRN or a TF-gene-phenotype causal graph — the current GRouNdGAN version only supports imposing a bipartite GRN
- The dataset contains significantly more genes/TFs than the reference configurations — hyperparameters (including GRN choices) may need re-tuning on a validation set rather than reusing defaults
- You want GRN benchmark conclusions on a heterogeneous dataset like PBMC-All without follow-up analysis — the article states those results are not conclusive regarding effects of distinct cell types

## Inputs

- Preprocessed scRNA-seq train split (from GRouNdGAN --preprocess; library-size normalized, 1000 highly variable genes)
- Transcription factor list (Homo_sapiens_TF.csv)
- Optionally a user-supplied bipartite TF-gene GRN to impose instead of GRNBoost2 output
- GRouNdGAN config file with [GRN] section parameters (top k = 15 TFs per gene)

## Outputs

- Formatted causal graph (TF-gene adjacency) consumed by GRouNdGAN --train
- Exported ground-truth GRN as CSV when [GRN Benchmarking]/ground truth save path is set
- GRN inference benchmark results from --benchmark_grn comparing a reconstructed GRN to the ground truth

## How to apply

After preprocessing the reference data (e.g., 10x Genomics PBMC or Dahlin GSE107727; min cells = 3, min genes = 10, library size = 20000, 1000 highly variable genes), run GRouNdGAN's --create_grn command, which uses GRNBoost2 (Moerman et al., 2018) to infer a GRN on the preprocessed train set and formats it as the causal graph. Restrict regulators to transcription factors from the Homo_sapiens_TF.csv list and keep the top k=15 most important TFs per gene. Alternatively, skip --create_grn and impose your own GRN, noting that only bipartite TF-gene graphs are supported (no multi-layer or TF-gene-phenotype graphs). For benchmarking, evaluate reconstructed GRNs against the ground truth with --benchmark_grn and optionally export the ground truth GRN as CSV; the article's recommended workflow is to first find regulatory edges from the reference (e.g., with GRNBoost2) and then impose a selected subset in the simulated data.

## Related tools

- **GRouNdGAN** (Provides the --create_grn, --benchmark_grn, and ground-truth export commands that wrap GRNBoost2 inference, causal-graph formatting, and GRN benchmarking) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers the GRN from the preprocessed train set; top k = 15 regulating TFs per gene are retained for the causal graph)
- **Beeline** (Reference benchmarking framework whose experimental GRN-inference results GRouNdGAN's benchmark results are compared against) — https://github.com/Murali-group/Beeline

## Examples

```
groundgan --create_grn  # infer GRN with GRNBoost2 (top k=15 TFs/gene) and format as causal graph for --train
```

## Evaluation signals

- The inferred GRN is restricted to TFs from Homo_sapiens_TF.csv and contains at most k = 15 regulating TFs per gene
- The formatted causal graph is a bipartite TF-gene graph that GRouNdGAN --train accepts without errors
- Training on the imposed GRN preserves resemblance: RF AUROC near 0.5, miLISI near 2, and Euclidean/Cosine/MMD comparable to the two-half test-set control
- --benchmark_grn ranks eight GRN inference methods consistently with BEELINE's experimental results on the same datasets
- Ground-truth GRN export produces a well-formed CSV when [GRN Benchmarking]/ground truth save path is specified

## Limitations

- The causal graph underlying an observational dataset is not necessarily unique; verifying the imposed GRN is the most probable one was beyond the study's scope
- Only bipartite GRNs are supported; multi-layer GRNs and TF-gene-phenotype causal graphs are planned for future versions
- Resemblance of simulated data to the reference deteriorates if imposed TF-gene relationships differ significantly from patterns in the reference data
- Hyperparameters may need re-tuning via a validation set for new datasets or datasets with very different gene/TF counts

## Evidence

- [methods] GRN creation via GRNBoost2: "This command uses GRNBoost2 (Moerman et al., 2018) to infer a GRN on the preprocessed train set."
- [methods] create_grn command description: "--create_grn     Infer a GRN from preprocessed data using GRNBoost2 and appropriately format as causal graph"
- [methods] Custom GRN option: "It is possible to instead impose your own GRN onto GRouNdGAN. If you're opting for this option, skip the ``--create_grn`` command."
- [discussion] Recommended benchmarking workflow: "we recommend first finding a set of regulatory edges from the reference dataset (e.g., using GRNBoost2 or even other data modalities)"
- [discussion] Bipartite limitation: "The current version of GRouNdGAN only allows imposing a bipartite GRN and does not support a multi-layer GRN"
