---
name: pathway-activity-inference-from-gene-expression
description: Use when you need to quantify signaling pathway activity from normalized
  gene expression (DESeq2 size-factor/VST RNA-seq, microarray, single-cell) for perturbation
  recall, driver mutation, drug-sensitivity, or patient-survival association analyses.
when_to_use_negative:
- When you need pathway activity for organisms/pathways without perturbation experiments
  — directed-path methods (e.g., Gene Ontology/Reactome abundance via GSVA) cover
  more pathways, but their expression level should not be interpreted as activity
- When a tissue-specific signature is available and the target is one specific tissue
  context — the consensus PROGENy signatures may be outperformed in narrow contexts
- When input is raw, un-normalized counts without size-factor normalization — DESeq2
  normalization precedes scoring in the reference workflows
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: progeny
  role: R/Bioconductor package providing the pathway weight matrix and scoring of
    pathway-responsive gene signatures from normalized expression
  repo: https://github.com/saezlab/progeny
- name: decoupler
  role: Recommended statistical framework (R or Python) for inferring pathway activities
    from the PROGENy signatures
  repo: https://github.com/scverse/decoupler
- name: saezlab/footprints
  role: 'Analysis code repository: perturbation experiment indices, z-score scripts,
    and the published ''speed_matrix'' model used to build PROGENy'
  repo: https://github.com/saezlab/footprints
- name: DESeq2
  role: Size-factor normalization and variance-stabilized transformation of RNA-seq
    counts prior to PROGENy scoring (GSE97979 workflow)
- name: survival
  role: coxph fits of PROGENy pathway scores against overall survival with study-of-origin
    and age covariates
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:ee49e576145ada603e66090427fb6484cc5fdb432f90cee37ed7aee700b8d511
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/progeny
  tools:
  - DESeq2
  - decoupler
  - progeny
  - saezlab/footprints
  - survival
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# pathway-activity-inference-from-gene-expression

## Summary

Infer signaling pathway activity scores from gene expression using PROGENy's Pathway RespOnsive GENes (PRGs): 11 pathway signatures (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF) derived from 208 ArrayExpress perturbation submissions (580 experiments), computed as weighted linear combinations of normalized expression via the progeny R/Bioconductor package.

## When to use

Use when you need to quantify signaling pathway activity from normalized gene expression (DESeq2 size-factor/VST RNA-seq, microarray, single-cell) for perturbation recall, driver mutation, drug-sensitivity, or patient-survival association analyses.

## When NOT to use

- When you need pathway activity for organisms/pathways without perturbation experiments — directed-path methods (e.g., Gene Ontology/Reactome abundance via GSVA) cover more pathways, but their expression level should not be interpreted as activity
- When a tissue-specific signature is available and the target is one specific tissue context — the consensus PROGENy signatures may be outperformed in narrow contexts
- When input is raw, un-normalized counts without size-factor normalization — DESeq2 normalization precedes scoring in the reference workflows

## Inputs

- Normalized gene expression matrix (DESeq2 size-factor-normalized / variance-stabilized RNA-seq counts, or GDSC microarray profiles from ArrayExpress E-MTAB-3610)
- PROGENy pathway weight/coefficient matrix (e.g., 'speed_matrix' model from the saezlab/footprints model directory, or the progeny Bioconductor package signatures)
- Perturbation metadata (GSE97979 conditions: 4OHT/MAPK, Ly294002/PI3K, TNF-alpha, TGF-beta, IFN-gamma, BSA vehicle control)
- Perturbation z-score matrix and binary pathway-perturbation indicator matrix (for leave-one-out re-fitting)
- Optional: MAF mutation calls, GISTIC CNA calls, GDSC IC50 drug table, TCGA clinical survival metadata for downstream validation

## Outputs

- Per-sample pathway activity score matrix (11 pathway columns: EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53, PI3K, TGFb, Trail, VEGF, androgen)
- pathway_score_deltas.csv (pathway, condition, delta_score vs. BSA control)
- pathway_perturbation_associations.csv (per-pathway self-association p-values from leave-one-out validation)
- pancancer_driver_pathway_fdr.csv (alteration, pathway, FDR) and drug_pathway_associations.csv (drug, pathway, effect size, FDR)
- survival_associations.csv (pathway, cohort, hazard ratio, FDR)

## How to apply

Build or load the pathway weight matrix (the published model, 'speed_matrix', fits a linear model on perturbation z-scores against a binary pathway-perturbation indicator and keeps the 100 most significant genes as coefficients). Normalize the expression matrix first (e.g., DESeq2 size factors with variance-stabilized expression for scoring, as in the GSE97979 HEK293-ER workflow), then compute per-sample pathway scores as weighted linear combinations using the progeny package. For validation workflows, score relative to controls (subtract BSA vehicle scores to get delta_score per pathway/condition), or use leave-one-out re-fitting where the tested perturbation experiment is excluded from model building. Downstream analyses (ANOVA for driver mutations and GDSC IC50s conditioned on MSI and tissue; Cox proportional hazards via survival::coxph regressing out study of origin and age) then test whether scores recover the perturbed pathway, driver alterations, drug sensitivity, or survival effects.

## Related tools

- **progeny** (R/Bioconductor package providing the pathway weight matrix and scoring of pathway-responsive gene signatures from normalized expression) — https://github.com/saezlab/progeny
- **decoupler** (Recommended statistical framework (R or Python) for inferring pathway activities from the PROGENy signatures) — https://github.com/scverse/decoupler
- **saezlab/footprints** (Analysis code repository: perturbation experiment indices, z-score scripts, and the published 'speed_matrix' model used to build PROGENy) — https://github.com/saezlab/footprints
- **DESeq2** (Size-factor normalization and variance-stabilized transformation of RNA-seq counts prior to PROGENy scoring (GSE97979 workflow))
- **survival** (coxph fits of PROGENy pathway scores against overall survival with study-of-origin and age covariates)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("progeny"); scores <- progeny(vst_expr, scale = TRUE, organism = "Human")
```

## Evaluation signals

- Leave-one-out validation: each pathway's self-association p-value with its perturbation experiment is p < 1e-10 (p < 1e-5 for Trail), and all expected pathway labels are present in pathway_perturbation_associations.csv
- GSE97979 HEK293-ER deltas: the sign of each delta_score matches the direction of change reported for the stimulated pathways in Fig. 2d, and the CSV has pathway/condition/delta_score fields
- TCGA driver associations: EGFR amplification activates EGFR and MAPK at FDR < 1e-9; BRAF < 1e-10, TP53 < 1e-64, VHL–hypoxia < 1e-200
- GDSC drug associations: exactly 178 significant drug-pathway rows at 10% FDR, with 'Nutlin-3a'–p53 as the strongest hit
- TCGA survival: significantly increased hazard for EGFR, MAPK, PI3K, Hypoxia pan-cancer, decreased hazard (increased survival) for Trail, and significant p53 in adrenocortical carcinoma, all FDR < 1e-3

## Limitations

- Individual perturbation signatures are heterogeneous and lack robustness; PROGENy captures only the common core response, and more studies are needed to distinguish it from additional transcriptional modules that may not always be activated
- Tissue-specific signatures may more closely reflect pathway activation status than consensus PROGENy signatures in the same context
- Expression level of a pathway's gene set (directed-path methods like GO/Reactome) should not be interpreted as pathway activity
- SPIA and Pathifier require tissue-matched normal samples (TCGA '11A' barcodes), and PARADIGM depends on a specific TCGA signaling network

## Evidence

- [abstract] PROGENy leverages perturbation compendium: "Here we present PROGENy, a method that overcomes both limitations by leveraging a large compendium of publicly available perturbation experiments to yield a common core of Pathway RespOnsive GENes."
- [readme] Model construction from z-scores: [evidence span withheld — anchor status: external_doc]
- [readme] Published model location: [evidence span withheld — anchor status: external_doc]
- [readme] Scoring via decoupleR: [evidence span withheld — anchor status: external_doc]
- [discussion] Context specificity caveat: "While PROGENy provides a good estimate of pathway activity in large and heterogeneous data sets, signatures derived from, for instance, a specific tissue may still more closely reflect activation"
