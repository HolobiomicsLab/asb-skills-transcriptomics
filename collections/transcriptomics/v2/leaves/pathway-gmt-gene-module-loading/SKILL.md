---
name: pathway-gmt-gene-module-loading
description: Use when a single-cell dataset will be analyzed with VEGA or another
  annotation-informed latent variable model and a GMT file of gene module membership
  (Reactome, MSigDB, GO, PANTHER) must be loaded to define the decoder wiring.
when_to_use_negative:
- No GMT/gene annotation file is available and modules must be learned de novo rather
  than hard-wired from prior knowledge
- The gene module annotations are suspected to be incomplete or not context-specific
  for the dataset, since VEGA's hard-coded decoder connections leave no room for correcting
  prior knowledge
- The input is raw count data not yet filtered/normalized — preprocessing must precede
  VEGA training, and membership loading alone does not address this
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VEGA
  role: Consumes the GMT-derived membership matrix to wire its sparse linear decoder
    to user-provided gene modules
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Wraps the single-cell dataset (AnnData) whose gene identifiers are matched
    against the GMT membership when building the matrix
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scvi-tools
  role: Ecosystem used by VEGA for model implementation and data handling alongside
    the GMT-derived modules
  repo: https://scvi-tools.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:86572a2fcd059adc12c0b97555f0adc189f8f0535e204973dbc95bce67c5f38c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - f-scLVM
  - scanpy
  - scvi-tools
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# pathway-gmt-gene-module-loading

## Summary

Parse a GMT-format gene set file (e.g., Reactome pathways or MSigDB gene sets) and convert it into the gene module membership matrix that hard-wires VEGA's sparse linear decoder, connecting genes to gene module variables (GMVs) in the latent space.

## When to use

Use when a single-cell dataset will be analyzed with VEGA or another annotation-informed latent variable model and a GMT file of gene module membership (Reactome, MSigDB, GO, PANTHER) must be loaded to define the decoder wiring.

## When NOT to use

- No GMT/gene annotation file is available and modules must be learned de novo rather than hard-wired from prior knowledge
- The gene module annotations are suspected to be incomplete or not context-specific for the dataset, since VEGA's hard-coded decoder connections leave no room for correcting prior knowledge
- The input is raw count data not yet filtered/normalized — preprocessing must precede VEGA training, and membership loading alone does not address this

## Inputs

- GMT file specifying gene module variables (GMVs) and gene-gene module membership, e.g., a Reactome pathway GMT from MSigDB
- Single-cell dataset wrapped as a Scanpy AnnData object (e.g., Kang et al. PBMC count matrix, preprocessed)

## Outputs

- Gene module membership matrix linking genes to pathway latent dimensions (decoder mask for VEGA)
- List of gene module names / GMV identifiers (e.g., interferon-alpha/beta signaling, tryptophan catabolism pathways)

## How to apply

Obtain a GMT file specifying the gene module variables (GMVs) and their gene membership from a curated database such as MSigDB, Reactome, Gene Ontology, PANTHER, or MolSigDB. Parse the GMT file to build a binary/weighted gene module membership matrix in which rows correspond to genes in the single-cell dataset (wrapped as a Scanpy AnnData object) and columns correspond to pathway latent dimensions. Only genes present in the GMT-defined modules are connected in the decoder; this sparse wiring is what makes each latent dimension directly interpretable as a biological module. The resulting membership matrix is passed to the VEGA model constructor so its sparse linear decoder mirrors the user-provided gene modules before training on the preprocessed PBMC (or other) count data.

## Related tools

- **VEGA** (Consumes the GMT-derived membership matrix to wire its sparse linear decoder to user-provided gene modules) — https://github.com/LucasESBS/vega
- **scanpy** (Wraps the single-cell dataset (AnnData) whose gene identifiers are matched against the GMT membership when building the matrix) — https://scanpy.readthedocs.io/en/stable/
- **scvi-tools** (Ecosystem used by VEGA for model implementation and data handling alongside the GMT-derived modules) — https://scvi-tools.org/

## Evaluation signals

- The membership matrix dimensions match the AnnData gene set (rows) and the number of GMT-defined pathways (columns)
- Known modules are present as latent dimensions, e.g., interferon-alpha/beta signaling and tryptophan catabolism from a Reactome GMT
- Decoder weights are zero outside module membership, confirming the sparse hard-wired wiring mirrors the GMT
- Trained VEGA latent space retains biological signal (interferon module segregates stimulated from control cells), indicating the membership file was loaded correctly

## Limitations

- Prior biological knowledge from databases like MSigDB can be incomplete or not context-specific; unannotated genes playing important roles in a module are excluded by the hard-coded decoder
- The hard-coded connections of the linear decoder leave no room for correcting prior knowledge about gene modules when context requires it
- GMT parsing depends on gene identifier consistency between the GMT file and the AnnData object; mismatches silently drop genes from modules

## Evidence

- [methods] GMT file(s) with the gene module membership, such as provided by databases like `MSigDB: "GMT file(s) with the gene module membership, such as provided by databases like `MSigDB"
- [intro] guided by gene module membership as recorded in gene annotation databases (e.g., Gene Ontology, PANTHER, MolSigDB, or Reactome): "guided by gene module membership as recorded in gene annotation databases (e.g., Gene Ontology, PANTHER, MolSigDB, or Reactome)"
- [readme] A GMT file specifying the gene module variables (GMVs) and gene membership, eg. from MSigDB: [evidence span withheld — anchor status: external_doc]
- [abstract] whose decoder wiring mirrors user-provided gene modules, providing direct interpretability to the latent variables: "whose decoder wiring mirrors user-provided gene modules, providing direct interpretability to the latent variables"
