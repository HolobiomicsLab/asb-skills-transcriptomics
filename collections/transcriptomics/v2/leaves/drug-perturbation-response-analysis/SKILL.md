---
name: drug-perturbation-response-analysis
description: Use when analyzing single-cell transcriptomic data from drug-treated
  cancer cell lines (e.g., MIX-Seq with multiple drugs and a DMSO control) and you
  need interpretable pathway-level response readouts rather than generic latent embeddings.
when_to_use_negative:
- When the goal is a predictive or generative model of drug response — VEGA's sparse
  single-layer decoder sacrifices predictive/generative capacity for interpretability
- When gene modules of interest are not represented in available annotation databases
  (e.g., MSigDB), since hard-coded decoder connections cannot correct incomplete or
  non-context-specific prior knowledge
- When input is raw unnormalized count data — VEGA works on preprocessed single-cell
  datasets
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: VEGA
  role: Sparse VAE with decoder wiring mirroring MSigDB hallmark GMT gene modules;
    trains per-condition models and infers single-cell GMV activity plus Bayesian
    differential pathway testing
  repo: https://github.com/LucasESBS/vega
- name: scVI
  role: Reference model in the scvi-tools ecosystem VEGA builds on; its differential
    expression framework inspired VEGA's Bayesian testing procedure
- name: MSigDB
  role: Source of hallmark gene set GMT files defining the interpretable latent space
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:b19ba00c8b5c461eaec0be3d1abc0dc3febbd422d51ee9ea0cc205787cb00d04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - MSigDB
  - VEGA
  - scVI
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# drug-perturbation-response-analysis

## Summary

Applies VEGA, a sparse variational autoencoder whose decoder wiring mirrors user-provided gene modules, to large-scale single-cell drug-perturbation datasets (e.g., MIX-Seq protocols from Srivatsan et al.) to detect and interpret drug-response patterns as gene module variable (GMV) activities in an interpretable latent space.

## When to use

Use when analyzing single-cell transcriptomic data from drug-treated cancer cell lines (e.g., MIX-Seq with multiple drugs and a DMSO control) and you need interpretable pathway-level response readouts rather than generic latent embeddings.

## When NOT to use

- When the goal is a predictive or generative model of drug response — VEGA's sparse single-layer decoder sacrifices predictive/generative capacity for interpretability
- When gene modules of interest are not represented in available annotation databases (e.g., MSigDB), since hard-coded decoder connections cannot correct incomplete or non-context-specific prior knowledge
- When input is raw unnormalized count data — VEGA works on preprocessed single-cell datasets

## Inputs

- Single-cell drug treatment dataset from a MIX-Seq protocol (Srivatsan et al. figshare deposit), as a scanpy AnnData object, preprocessed per VEGA recommendations
- GMT file(s) of gene module membership (e.g., MSigDB hallmark gene sets)
- Condition labels distinguishing each drug treatment from the DMSO control

## Outputs

- Trained VEGA model per condition (drug treatments and DMSO control)
- Single-cell gene module activity (GMV) values per drug condition
- Bayesian differential pathway activity results per treatment-vs-DMSO comparison, with evidence measures such as log_e(BF) per cell line group

## How to apply

Load the MIX-Seq drug treatment AnnData object (figshare deposit from Srivatsan et al.) and apply the recommended preprocessing before passing it to VEGA. Initialize the VEGA architecture with MSigDB hallmark gene sets supplied as GMT files so the sparse decoder wiring mirrors the gene modules, then train one VEGA model per drug treatment plus one for the DMSO control using the scvi-tools/scanpy ecosystem. Infer single-cell gene module activities (GMVs) for each trained model, and run VEGA's Bayesian differential pathway activity testing (inspired by scvi-tools differential expression) against the DMSO control to identify significantly activated or deactivated modules. Correct application is confirmed by reproducing mechanism-of-action signatures: e.g., decreased G2M checkpoint GMV activity under the MEK inhibitor Trametinib, and drug-specific evidence such as larger average |log_e(BF)| for Dabrafenib in BRAF-mutant melanoma lines (0.763) versus other lines (0.668).

## Related tools

- **VEGA** (Sparse VAE with decoder wiring mirroring MSigDB hallmark GMT gene modules; trains per-condition models and infers single-cell GMV activity plus Bayesian differential pathway testing) — https://github.com/LucasESBS/vega
- **scVI** (Reference model in the scvi-tools ecosystem VEGA builds on; its differential expression framework inspired VEGA's Bayesian testing procedure)
- **MSigDB** (Source of hallmark gene set GMT files defining the interpretable latent space)

## Examples

```
git clone https://github.com/LucasESBS/vega-reproducibility && cd vega-reproducibility && pip install -r requirements.txt && bash train_all_models.sh
```

## Evaluation signals

- G2M checkpoint GMV activity significantly decreases under Trametinib, matching the expected mechanism of action of a MEK inhibitor
- Trametinib-specific interferon-alpha/gamma response is recovered in the differential pathway testing results
- Average |log_e(BF)| for Dabrafenib-treated BRAF-mutant melanoma lines (0.763) is larger than for other lines (0.668)
- Reproduction of the paper's figures using the vega-reproducibility repository's preprocessed data archive, train_all_models.sh script, and jupyter notebooks
- Each condition (drug treatments and DMSO control) yields a successfully trained VEGA model embedded in the hallmark-defined latent space

## Limitations

- The sparse, single-layer decoder architecture limits generalizability and robustness, and VEGA's generative capacity is limited — predictive performance is sacrificed for biological interpretability
- Hard-coded decoder connections leave no room for correcting prior gene module annotations when context requires it
- Prior biological knowledge from databases like MSigDB can be incomplete or not context-specific, as unannotated genes can play important roles in certain gene modules

## Evidence

- [abstract] Sparse decoder mirrors gene modules: "we introduce a novel sparse Variational Autoencoder architecture, VEGA (VAE Enhanced by Gene Annotations), whose decoder wiring mirrors user-provided gene modules"
- [methods] GMV inference and Bayesian testing: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [readme] Reproducibility workflow: [evidence span withheld — anchor status: external_doc]
- [methods] Preprocessing recommendation: "Note: We recommend to preprocess the single-cell dataset before passing it to VEGA."
- [methods] GMT input requirement: "GMT file(s) with the gene module membership, such as provided by databases like `MSigDB <https://www.gsea-msigdb.org/gsea/index.jsp>`_"
