---
name: sparse-masked-decoder-construction
description: Use when building an interpretable VAE decoder whose latent variables
  must correspond one-to-one to user-supplied gene modules (GMT membership) rather
  than to free-form latent dimensions.
when_to_use_negative:
- The goal is a maximally accurate generative model rather than an interpretable latent
  space — VEGA's sparse single-layer decoder sacrifices predictive performance and
  has limited generative capacity
- The prior gene modules are incomplete or context-specific corrections are needed,
  since the hard-coded decoder connections leave no room for correcting prior knowledge
- The data are raw counts that require a count likelihood (VEGA was designed for normalized
  expression profiles; adapting it to counts follows the scVI rationale)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: VEGA
  role: The VAE implementation whose decoder wiring mirrors user-provided gene modules;
    constructed and trained in this skill
  repo: https://github.com/LucasESBS/vega
- name: PyTorch
  role: Framework used to implement the encoder (mu, Sigma, latent dropout) and the
    masked sparse linear decoder with nonnegative weight clamping
- name: scanpy
  role: Used to wrap the preprocessed single-cell dataset (AnnData) that defines the
    gene order matched by the mask
- name: scvi-tools
  role: Ecosystem used alongside scanpy in the VEGA implementation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:1ac0100ac3b9348c100e2cfa545ddf5d94d4a83c6a32becf5f1677c22c4b2ce8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - PyTorch
  - VEGA
  - scanpy
  - scvi-tools
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sparse-masked-decoder-construction

## Summary

Constructs VEGA's single-layer sparse linear decoder by wiring a binary gene-membership mask (built from GMT gene-module files) so each latent gene-module variable (GMV) connects only to its annotated genes, with nonnegative weights and appended fully connected hidden nodes for unannotated genes.

## When to use

Use when building an interpretable VAE decoder whose latent variables must correspond one-to-one to user-supplied gene modules (GMT membership) rather than to free-form latent dimensions.

## When NOT to use

- The goal is a maximally accurate generative model rather than an interpretable latent space — VEGA's sparse single-layer decoder sacrifices predictive performance and has limited generative capacity
- The prior gene modules are incomplete or context-specific corrections are needed, since the hard-coded decoder connections leave no room for correcting prior knowledge
- The data are raw counts that require a count likelihood (VEGA was designed for normalized expression profiles; adapting it to counts follows the scVI rationale)

## Inputs

- GMT file(s) specifying gene module variables (GMVs) and gene membership, e.g., from MSigDB
- Preprocessed single-cell expression matrix wrapped in a Scanpy AnnData object
- Optional batch-key covariate annotations for the covariate input path

## Outputs

- vega.py module containing the masked sparse linear decoder and encoder (mu, Sigma)
- vega_unit_test_report.md with pass/fail results of mask-sparsity and weight-constraint checks
- Trained VEGA model with per-GMV interpretable latent activities for single-cell module-activity inference

## How to apply

Parse the GMT gene-module membership files (e.g., from MSigDB, Reactome, Gene Ontology, PANTHER) to assemble a binary mask M whose columns match the gene order of the Scanpy AnnData expression matrix. Implement a PyTorch encoder producing mu and Sigma with latent dropout in the probabilistic latent layer, then implement the decoder as a single-layer masked sparse linear layer with a nonnegativity constraint on its weights, appended fully connected (FC) hidden nodes to cover unannotated genes, and an optional batch-key covariate input path. The mask ensures each output gene receives input only from its annotated module variable or an FC node, which is what makes the latent space directly interpretable. Unit-test that the mask is sparse per-gene and that all decoder weights remain nonnegative after initialization and parameter clamping, writing pass/fail results to a report alongside vega.py. Optionally add a Bayesian testing procedure on module activities to find significantly activated gene modules in the dataset.

## Related tools

- **VEGA** (The VAE implementation whose decoder wiring mirrors user-provided gene modules; constructed and trained in this skill) — https://github.com/LucasESBS/vega
- **PyTorch** (Framework used to implement the encoder (mu, Sigma, latent dropout) and the masked sparse linear decoder with nonnegative weight clamping)
- **scanpy** (Used to wrap the preprocessed single-cell dataset (AnnData) that defines the gene order matched by the mask)
- **scvi-tools** (Ecosystem used alongside scanpy in the VEGA implementation)

## Examples

```
pip install scvega; import vega; model = vega.VEGA(mydata, gmt_file='modules.gmt')
```

## Evaluation signals

- Mask-sparsity unit test passes: each output gene receives decoder input only from its annotated module variable or an appended FC node
- All decoder weights are nonnegative both after initialization and after every parameter-clamping step during training
- Mask column/row ordering matches the gene order of the AnnData expression matrix assembled from the GMT files
- Trained model yields per-GMV activities that identify significantly activated gene modules in the Bayesian differential activity test
- Reproduction of paper results matches figures generated with the LucasESBS/vega-reproducibility pipeline

## Limitations

- The sparse, single-layer decoder prevents further improvement of generalizability and robustness, and the generative capacity of VEGA is limited
- VEGA's limited generative capacity sacrifices predictive performance for biological interpretability of the latent space
- Hard-coded connections of the linear decoder leave no room for correcting prior knowledge about gene modules when context requires it; MSigDB-like priors can be incomplete or not context-specific

## Evidence

- [abstract] decoder wiring mirrors gene modules: "we introduce a novel sparse Variational Autoencoder architecture, VEGA (VAE Enhanced by Gene Annotations), whose decoder wiring mirrors user-provided gene modules"
- [methods] GMT membership mask construction: "GMT file(s) with the gene module membership, such as provided by databases like `MSigDB <https://www.gsea-msigdb.org/gsea/index.jsp>`_"
- [intro] sparse linear decoder informed by networks: "Here we propose VEGA (VAE enhanced by gene annotations), a VAE with a sparse linear decoder informed by biological networks"
- [discussion] single-layer decoder limitation: "The clear limitations of the current architecture resides in the sparse, single-layer decoder of the model."
- [readme] README: VEGA inputs: "A single-cell dataset wrapped using the Scanpy package
* A GMT file specifying the gene module variables (GMVs) and gene membership, eg. from MSigDB"
- [readme] README: package install: "With pip, you can run
``npip install scvega
"
