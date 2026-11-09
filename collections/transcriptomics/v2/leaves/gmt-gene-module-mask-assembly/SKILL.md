---
name: gmt-gene-module-mask-assembly
description: 'Use when building a VEGA model or similar annotation-guided VAE: you
  have GMT-format gene set files (e.g., MSigDB) and a Scanpy AnnData matrix, and need
  the decoder mask linking GMVs to member genes.'
when_to_use_negative:
- The analysis goal is a generic, fully expressive generative model with maximal predictive
  performance rather than an interpretable latent space — VEGA's hard-coded sparse
  single-layer decoder sacrifices generative capacity for interpretability.
- The gene modules are unknown or you expect prior annotations (e.g., MSigDB) to be
  incomplete or not context-specific and need the model to correct them — the hard-coded
  connections leave no room for correcting prior knowledge.
- Input is raw unprocessed counts without the recommended preprocessing or a GMT membership
  file matching the AnnData gene universe.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: VEGA
  role: The VAE whose decoder is wired by the assembled GMT-derived gene membership
    mask; consumes the mask to link each GMV to its member genes
  repo: https://github.com/LucasESBS/vega
- name: PyTorch
  role: Framework in which the masked sparse linear decoder implementing the mask
    wiring is implemented
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:a8a27fa985cb804a8685aa77ab7ba0cc2fe9814aec92562066b0b9198d726f5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - PyTorch
  - VEGA
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gmt-gene-module-mask-assembly

## Summary

Assembles the binary gene membership mask from GMT gene-module files that wires VEGA's sparse linear decoder, so each latent gene-module variable (GMV) connects only to its annotated member genes in the order of the AnnData expression matrix.

## When to use

Use when building a VEGA model or similar annotation-guided VAE: you have GMT-format gene set files (e.g., MSigDB) and a Scanpy AnnData matrix, and need the decoder mask linking GMVs to member genes.

## When NOT to use

- The analysis goal is a generic, fully expressive generative model with maximal predictive performance rather than an interpretable latent space — VEGA's hard-coded sparse single-layer decoder sacrifices generative capacity for interpretability.
- The gene modules are unknown or you expect prior annotations (e.g., MSigDB) to be incomplete or not context-specific and need the model to correct them — the hard-coded connections leave no room for correcting prior knowledge.
- Input is raw unprocessed counts without the recommended preprocessing or a GMT membership file matching the AnnData gene universe.

## Inputs

- GMT file(s) with the gene module membership (e.g., from MSigDB, Gene Ontology, PANTHER, MolSigDB, or Reactome)
- Single-cell dataset wrapped in a Scanpy AnnData object (preprocessed), whose gene order defines mask columns

## Outputs

- Binary gene-membership mask M (GMVs × genes) aligned to the AnnData gene order
- Mask-sparsity unit-test report (e.g., vega_unit_test_report.md) verifying each output gene receives input only from its annotated module or an FC node
- Masked sparse decoder wiring used by the vega.py module

## How to apply

Parse the GMT file(s) specifying the gene module variables (GMVs) and gene membership — e.g., from MSigDB — and build a binary mask M with rows as GMVs and columns as genes, matching exactly the gene order of the AnnData expression matrix so decoder output aligns with the count matrix. Wire the mask so a link exists from a GMV to an output gene only when that gene is annotated as a member of that module; genes absent from all modules are handled by appending fully connected hidden nodes rather than leaving them disconnected. Validate the mask by unit tests confirming sparsity (each output gene receives input only from its annotated module or an FC node) before feeding it to the single-layer masked sparse linear decoder, whose weights must remain nonnegative. This mask-defined sparse wiring is the mechanism that makes VEGA's latent variables directly interpretable as pathway, GRN, or cell-type module activities.

## Related tools

- **VEGA** (The VAE whose decoder is wired by the assembled GMT-derived gene membership mask; consumes the mask to link each GMV to its member genes) — https://github.com/LucasESBS/vega
- **PyTorch** (Framework in which the masked sparse linear decoder implementing the mask wiring is implemented)

## Examples

```
import vega; vega.vega.LoadFromFile / model = vega.VEGA(my_adata, gmt_file='msigdb_hallmark.gmt')  # builds the GMV×gene mask from GMT and the AnnData gene order
```

## Evaluation signals

- Mask sparsity unit test passes: each output gene column has nonzeros only for its annotated module row (plus any appended fully connected node), never across arbitrary GMVs.
- Mask dimensions and column order exactly match the gene order of the AnnData expression matrix; row count equals the number of GMVs parsed from the GMT file.
- All decoder weights stay nonnegative after initialization and parameter clamping during training.
- Genes not annotated to any module are covered by appended fully connected hidden nodes, not dropped silently.
- Trained VEGA latent GMV activities recover known module activation (e.g., pathway activity differences such as interferon response in the Kang et al. PBMCs dataset).

## Limitations

- The hard-coded connections of the linear decoder do not leave room for correcting prior knowledge about gene modules when the context requires it.
- Prior biological knowledge from databases like MSigDB can be incomplete or not context-specific, since additional unannotated genes can play an important role in certain gene modules.
- The sparse, single-layer decoder architecture prevents further improvement of generalizability and robustness, and VEGA's generative capacity is limited, sacrificing predictive performance for biological interpretability.

## Evidence

- [methods] GMT-derived mask wiring: "GMT file(s) with the gene module membership, such as provided by databases like `MSigDB <https://www.gsea-msigdb.org/gsea/index.jsp>`_"
- [abstract] decoder mirrors modules: "a novel sparse Variational Autoencoder architecture, VEGA (VAE Enhanced by Gene Annotations), whose decoder wiring mirrors user-provided gene modules"
- [readme] required inputs: "A single-cell dataset wrapped using the Scanpy package
* A GMT file specifying the gene module variables (GMVs) and gene membership, eg. from MSigDB"
- [intro] annotation databases: "guided by gene module membership as recorded in gene annotation databases (e.g., Gene Ontology, PANTHER, MolSigDB, or Reactome)"
- [discussion] hard-coded connections limitation: "hard-coded connections of the linear decoder do not leave any room for correcting prior knowledge about gene modules when the context requires it"
