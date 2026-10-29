---
name: mofa-factor-weight-feature-extraction
description: Use when a trained MOFA model over scPAFA-derived pathway activity scores
  has latent factors linking pathways across cell types to a disease phenotype, and
  you need to select the top-weighted pathways as features for classifier training
  or validation.
when_to_use_negative:
- The study has fewer than 15 samples (not cells)—MOFA-based multicellular pathway
  module identification requires at least 15 samples, so no factor weights are available
  to extract
- You need causal pathway-cell type relationships—the modules are data-driven associations
  only, not biological causal effects
- The input is already a feature table from bulk data with no single-cell PAS/MOFA
  factor structure—use standard feature selection instead
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scPAFA
  role: Provides the Step 4 downstream-analysis workflow that identifies disease-related
    multicellular pathway modules (latent factors and corresponding pathway weights
    across cell types) from the trained MOFA model
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: MOFA (mofapy2)
  role: Trains the factor model whose latent factors and weights are the source of
    the high-weight pathway features extracted here
  repo: https://github.com/bioFAM/mofapy2
- name: scikit-learn
  role: Consumes the extracted high-weight pathway features (via GSVA scores) to train
    an L1-regularized LinearSVC (C=0.2) classifier for validation
- name: GSVA
  role: Computes pathway-level scores for the extracted high-weight pathways on bulk
    expression matrices, converting single-cell features into classifier inputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:f23174aaa7995bac286a8963745b0d93a5a62b6e3f928d2470169b786e4febb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuoliHuang/scPAFA
  tools:
  - GSVA
  - MOFA
  - MOFA (mofapy2)
  - Python
  - scPAFA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# mofa-factor-weight-feature-extraction

## Summary

Extracts high-weight pathways (features) from trained MOFA factors—i.e., disease-related multicellular pathway modules identified by scPAFA—so they can be dissected and used to train downstream machine-learning classifiers on independent data.

## When to use

Use when a trained MOFA model over scPAFA-derived pathway activity scores has latent factors linking pathways across cell types to a disease phenotype, and you need to select the top-weighted pathways as features for classifier training or validation.

## When NOT to use

- The study has fewer than 15 samples (not cells)—MOFA-based multicellular pathway module identification requires at least 15 samples, so no factor weights are available to extract
- You need causal pathway-cell type relationships—the modules are data-driven associations only, not biological causal effects
- The input is already a feature table from bulk data with no single-cell PAS/MOFA factor structure—use standard feature selection instead

## Inputs

- Trained MOFA model object over the pseudobulk scPAFA PAS matrix (pathway x cell type weights per latent factor)
- Sample-level clinical metadata (e.g., disease/MSI status) aligned to MOFA samples
- Pathway dictionary used to build the PAS matrix (pathway name -> gene list)
- Bulk RNA-seq expression matrices for validation cohorts (e.g., GSE39582, TCGA COAD-READ) when features are used for classifier testing

## Outputs

- Ranked list of high-weight pathways per disease-related multicellular pathway module (e.g., top 30 pathways by MOFA factor weight)
- Characterized/annotated multicellular pathway modules with pathway-by-cell-type weight structures
- Feature set used to derive GSVA scores and train/evaluate a classifier (per-cohort AUROC values)

## How to apply

After scPAFA's Step 3 (pseudobulk processing and MOFA model training) has captured variance in PAS among samples, perform Step 4 downstream analysis: join the MOFA latent factors and their pathway-by-cell-type weights to sample-level clinical metadata, and statistically identify disease-related multicellular pathway modules. Rank pathways within the disease-associated module by their MOFA factor weights and extract the top candidates—for example, in the CRC MMR-status task, the 30 pathways carrying the highest factor weights were selected. Feed these high-weight pathways forward as a gene-set feature basis: compute GSVA scores on bulk expression matrices and train an L1-regularized LinearSVC (C=0.2 chosen by 4-fold CV) to validate the module. Rationale: modules 'can further be dissected to extract features for training classifiers', so factor weights define a compact, biologically interpretable feature set rather than using all pathways.

## Related tools

- **scPAFA** (Provides the Step 4 downstream-analysis workflow that identifies disease-related multicellular pathway modules (latent factors and corresponding pathway weights across cell types) from the trained MOFA model) — https://github.com/ZhuoliHuang/scPAFA
- **MOFA (mofapy2)** (Trains the factor model whose latent factors and weights are the source of the high-weight pathway features extracted here) — https://github.com/bioFAM/mofapy2
- **scikit-learn** (Consumes the extracted high-weight pathway features (via GSVA scores) to train an L1-regularized LinearSVC (C=0.2) classifier for validation)
- **GSVA** (Computes pathway-level scores for the extracted high-weight pathways on bulk expression matrices, converting single-cell features into classifier inputs)

## Examples

```
from scPAFA import downstream; modules = downstream.identify_modules(mofa_model, clinical_metadata); top30 = modules['MMR'].sort_values('weight', ascending=False).head(30)
```

## Evaluation signals

- The extracted pathway count matches the intended feature size (e.g., 30 highest-weight pathways from the MMR-related module)
- Each selected pathway's absolute MOFA factor weight exceeds that of non-selected pathways in the same module (correct ranking)
- Downstream classifier trained on GSVA scores of the extracted pathways reaches held-out test AUROC ≈ 0.91 on GSE39582 and independent AUROC ≈ 0.91 on TCGA COAD-READ, matching Fig. 3d-g
- Extracted pathway names resolve against the original pathway dictionary (MsigDB or NCATS BioPlanet) used for PAS computation
- Per-cohort AUROC values are written to mmr_signature_auroc_results.csv with one row per validation cohort

## Limitations

- MOFA is a linear model; interpretability is achieved by sacrificing information content in each factor
- Pseudobulk integration of the PAS matrix leads to information loss since PAS of multiple cells is aggregated
- Pathways and cell types within the same module suggest data-driven associations only, not biological causal effects; interpretability may decrease with multiple disease states

## Evidence

- [readme] disease-related multicellular pathway modules (latent factor and its corresponding weights of pathways across cell types) can be identified by statistical analysis: "disease-related multicellular pathway modules (latent factor and its corresponding weights of pathways across cell types) can be identified by statistical analysis"
- [discussion] These modules are consistent with previous reports in biological interpretation and can further be dissected to extract features for training classifiers.: "These modules are consistent with previous reports in biological interpretation and can further be dissected to extract features for training classifiers."
- [methods] MOFA model is trained to capture variance in PAS among different samples: "MOFA model is trained to capture variance in PAS among different samples"
- [discussion] MOFA is a linear model, interpretability is achieved by sacrificing information content in each factor: "MOFA is a linear model, interpretability is achieved by sacrificing information content in each factor"
- [discussion] the selection of pathways is a supervised process, where using a redundant pathway set may increase the complexity of the results: "the selection of pathways is a supervised process, where using a redundant pathway set may increase the complexity of the results"
