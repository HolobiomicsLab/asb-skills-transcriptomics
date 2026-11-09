---
name: pathway-activity-inference-at-single-cell-level
description: Use when single-cell RNA-seq data (e.g., AnnData-wrapped MIX-Seq drug
  treatment data) needs to be embedded in a biologically interpretable latent space
  defined by GMT gene sets, and per-cell pathway activity or differential pathway
  activity between conditions is the goal.
when_to_use_negative:
- When predictive/generative performance is the priority rather than interpretability
  — VEGA's sparse, single-layer decoder sacrifices predictive performance and has
  limited generative capacity
- When prior gene module annotations (e.g., MSigDB) are incomplete or not context-specific
  and unannotated genes are expected to matter — the hard-coded decoder connections
  leave no room for correcting prior knowledge
- When the goal is de novo discovery of pathways or latent factors without a priori
  gene module definitions
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: VEGA
  role: Sparse VAE (VAE Enhanced by Gene Annotations) whose decoder wiring mirrors
    user-provided gene modules; used to embed cells into the interpretable latent
    space, infer per-cell GMVs, and run Bayesian differential pathway activity testing
  repo: https://github.com/LucasESBS/vega
- name: scVI
  role: Reference framework whose differential expression procedure inspired VEGA's
    Bayesian differential pathway activity testing; VEGA is built within the scvi-tools
    ecosystem
  repo: https://github.com/LucasESBS/vega
- name: MSigDB
  role: Source of GMT gene sets (e.g., hallmark gene sets) that define the latent
    space of the VEGA model
- name: vega-reproducibility
  role: Repository with preprocessed data archive, train_all_models.sh script, and
    Jupyter notebooks to reproduce the paper's figures (clone the 'revision1' branch
    for up-to-date code)
  repo: https://github.com/LucasESBS/vega-reproducibility
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:bc0088d1673ee54de35b68d20b0147692c14add7046a8972fa93ae54ea4ba5db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - MSigDB
  - VEGA
  - scVI
  - vega-reproducibility
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# pathway-activity-inference-at-single-cell-level

## Summary

Infers gene module/pathway activities (GMVs) per single cell using VEGA, a sparse VAE whose decoder wiring mirrors user-provided gene modules (e.g., MSigDB hallmark GMT files), enabling interpretable latent-space analysis of perturbation responses such as drug treatments in MIX-Seq data.

## When to use

Use when single-cell RNA-seq data (e.g., AnnData-wrapped MIX-Seq drug treatment data) needs to be embedded in a biologically interpretable latent space defined by GMT gene sets, and per-cell pathway activity or differential pathway activity between conditions is the goal.

## When NOT to use

- When predictive/generative performance is the priority rather than interpretability — VEGA's sparse, single-layer decoder sacrifices predictive performance and has limited generative capacity
- When prior gene module annotations (e.g., MSigDB) are incomplete or not context-specific and unannotated genes are expected to matter — the hard-coded decoder connections leave no room for correcting prior knowledge
- When the goal is de novo discovery of pathways or latent factors without a priori gene module definitions

## Inputs

- Preprocessed single-cell dataset wrapped as a Scanpy AnnData object (e.g., MIX-Seq drug treatment data from Srivatsan et al. figshare deposit)
- GMT file(s) specifying gene module variables (GMVs) and gene membership, e.g., MSigDB hallmark gene sets
- Condition/labels metadata distinguishing treatments (e.g., Trametinib, Dabrafenib) from control (DMSO)

## Outputs

- Per-cell gene module activity (GMV) scores in the interpretable latent space
- Trained VEGA models per condition (saved to ./trained_models/ in the reproducibility workflow)
- Differentially activated gene modules between treatment and control with Bayesian evidence statistics (log_e Bayes factors)

## How to apply

Preprocess the single-cell dataset (as recommended before passing it to VEGA) and wrap it with Scanpy as an AnnData object. Initialize VEGA — a sparse VAE with a linear decoder whose weights are masked by gene module membership — using GMT files (e.g., MSigDB hallmark gene sets) to define the latent space; VEGA is implemented with PyTorch within the scanpy and scvi-tools ecosystem. Train one model per condition of interest (e.g., per drug treatment plus a DMSO control in the MIX-Seq experiment from Srivatsan et al.), then infer gene module activities (GMVs) at the single-cell level for every trained model. For condition contrasts, run VEGA's Bayesian differential pathway activity testing procedure (inspired by scvi-tools differential expression) to find significantly activated gene modules between treatment and control. Validate against expected biology: e.g., G2M checkpoint GMV should decrease under the MEK inhibitor Trametinib, and drug-specific responses (interferon-alpha/gamma for Trametinib) plus condition-specific effect sizes (larger average |log_e(BF)| for Dabrafenib in BRAF-mutant melanoma lines, 0.763 vs 0.668) should be recovered.

## Related tools

- **VEGA** (Sparse VAE (VAE Enhanced by Gene Annotations) whose decoder wiring mirrors user-provided gene modules; used to embed cells into the interpretable latent space, infer per-cell GMVs, and run Bayesian differential pathway activity testing) — https://github.com/LucasESBS/vega
- **scVI** (Reference framework whose differential expression procedure inspired VEGA's Bayesian differential pathway activity testing; VEGA is built within the scvi-tools ecosystem) — https://github.com/LucasESBS/vega
- **MSigDB** (Source of GMT gene sets (e.g., hallmark gene sets) that define the latent space of the VEGA model)
- **vega-reproducibility** (Repository with preprocessed data archive, train_all_models.sh script, and Jupyter notebooks to reproduce the paper's figures (clone the 'revision1' branch for up-to-date code)) — https://github.com/LucasESBS/vega-reproducibility

## Examples

```
bash train_all_models.sh
```

## Evaluation signals

- G2M checkpoint GMV activity shows a decrease under Trametinib treatment relative to DMSO control, consistent with the expected mechanism of action of a MEK inhibitor
- Trametinib-specific interferon-alpha/gamma response modules are detected by the differential pathway activity test
- Average |log_e(BF)| for Dabrafenib-treated BRAF-mutant melanoma lines is larger (0.763) than for other lines (0.668)
- Trained models are written to ./trained_models/ after running train_all_models.sh and downstream notebooks reproduce the paper figures
- Latent space dimensions correspond one-to-one to user-provided GMT gene modules, giving directly interpretable GMVs

## Limitations

- The sparse, single-layer decoder architecture prevents further improvement of generalizability and robustness
- VEGA's generative capacity is limited and sacrifices predictive performance for biological interpretability of the latent space
- Hard-coded decoder connections leave no room for correcting prior gene module knowledge when context requires it; MSigDB-derived priors can be incomplete or not context-specific
- Datasets must be preprocessed before being passed to VEGA, as recommended by the authors

## Evidence

- [abstract] decoder wiring mirrors gene modules: "we introduce a novel sparse Variational Autoencoder architecture, VEGA (VAE Enhanced by Gene Annotations), whose decoder wiring mirrors user-provided gene modules"
- [methods] preprocess before VEGA: "Note: We recommend to preprocess the single-cell dataset before passing it to VEGA."
- [methods] GMT input requirement: "GMT file(s) with the gene module membership, such as provided by databases like `MSigDB <https://www.gsea-msigdb.org/gsea/index.jsp>`_"
- [methods] Bayesian differential testing: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [readme] reproducibility workflow: "3) Run the bash script to train all models:
``nbash train_all_models.sh
``nOnce done training, models are in ./trained_models/"
- [discussion] interpretability-performance tradeoff: "VEGA’s limited generative capacity sacrifices predictive performance for biological interpretability of the latent space"
