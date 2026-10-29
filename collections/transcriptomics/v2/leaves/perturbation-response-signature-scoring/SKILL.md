---
name: perturbation-response-signature-scoring
description: Use when bulk or single-cell normalized gene-expression data (e.g. DESeq2
  size-factor-normalized or variance-stabilized counts) should be converted into per-sample
  pathway activity scores for signaling pathways like MAPK, NFkB, PI3K, or TGFb.
when_to_use_negative:
- The input is raw unnormalized counts — z-scores/fold-changes require normalized
  and QC'd series (normalize_data.r → expr.r → zscores.r) before scoring
- A single-signature, condition-specific pathway score is desired instead of the consensus
  pathway-responsive core; the article notes tissue-specific signatures 'may still
  more closely reflect activation' in a specific context
- Pathway scoring must account for direct post-translational modification measurements
  — gene-expression-based scores do not directly observe PTMs, only signaling footprints
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: PROGENy
  role: Provides the consensus pathway-responsive gene signatures used to compute
    pathway activity scores from normalized expression
  repo: github.com/saezlab/progeny
- name: progeny
  role: R/Bioconductor package supplying the pathway weight matrix and scoring function
    for pathway activity inference
  repo: github.com/saezlab/progeny
- name: decoupler
  role: Recommended inference framework for applying the PROGENy weights to omics
    data (R and Python)
  repo: github.com/scverse/decoupler
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:86abc3bf5b6d27f41bcf2407b9b158509efe1c7e477f6e1bad57e54fe09901ef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PROGENy
  - decoupler
  - progeny
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# perturbation-response-signature-scoring

## Summary

Score pathway activity from gene-expression data using PROGENy's consensus Pathway RespOnsive GENes (PRGs) — a set of perturbation-response signatures derived from a compendium of ArrayExpress perturbation experiments for 11 pathways (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF).

## When to use

Use when bulk or single-cell normalized gene-expression data (e.g. DESeq2 size-factor-normalized or variance-stabilized counts) should be converted into per-sample pathway activity scores for signaling pathways like MAPK, NFkB, PI3K, or TGFb.

## When NOT to use

- The input is raw unnormalized counts — z-scores/fold-changes require normalized and QC'd series (normalize_data.r → expr.r → zscores.r) before scoring
- A single-signature, condition-specific pathway score is desired instead of the consensus pathway-responsive core; the article notes tissue-specific signatures 'may still more closely reflect activation' in a specific context
- Pathway scoring must account for direct post-translational modification measurements — gene-expression-based scores do not directly observe PTMs, only signaling footprints

## Inputs

- Normalized gene-expression matrix (e.g. DESeq2 size-factor-normalized or variance-stabilized RNA-seq counts, or microarray expression)
- Sample/condition metadata specifying perturbation conditions and matched controls (e.g. 4OHT, Ly294002, TNF-alpha, TGF-beta, IFN-gamma vs. BSA vehicle)
- PROGENy pathway weight matrix (speed_matrix; 100 most significant pathway-responsive genes per pathway) from the progeny R package

## Outputs

- Per-sample pathway activity score matrix (samples × pathways)
- Contrast table of pathway-score differences per condition vs. control (pathway_score_deltas.csv: pathway, condition, delta_score)
- Leave-one-out association table (pathway_perturbation_associations.csv) of per-pathway p-values

## How to apply

Build or load the PROGENy model coefficients: in the reference work, z-scores are computed for each perturbation experiment from ArrayExpress series, and a linear model is fit on the z-scores with a binary pathway-perturbation matrix as the independent variable; the 100 most significant genes per pathway and their z-scores form the signature matrix ('speed_matrix'). Load the weight matrix via the progeny Bioconductor package and compute per-sample pathway scores as the weighted linear combination of normalized expression. For perturbation contrasts (e.g. 4OHT/MAPK, Ly294002/PI3K, TNF-alpha, TGF-beta, IFN-gamma in GSE97979), subtract the BSA vehicle-control score to obtain a delta_score per pathway. For leave-one-out validation, rebuild the model with each perturbation experiment excluded before scoring, then compute pathway-score vs. perturbation-condition association p-values across the compendium.

## Related tools

- **PROGENy** (Provides the consensus pathway-responsive gene signatures used to compute pathway activity scores from normalized expression) — github.com/saezlab/progeny
- **progeny** (R/Bioconductor package supplying the pathway weight matrix and scoring function for pathway activity inference) — github.com/saezlab/progeny
- **decoupler** (Recommended inference framework for applying the PROGENy weights to omics data (R and Python)) — github.com/scverse/decoupler

## Examples

```
Rscript -e 'library(progeny); scores <- progeny(expr_matrix)'
```

## Evaluation signals

- Leave-one-out validation: each pathway's self-association p-value with its perturbation condition should be < 1e-10 (< 1e-5 for Trail), and all expected pathway labels present in the association table
- In the GSE97979 HEK293-ER experiments, the sign of each delta_score (perturbation minus BSA control) must match the direction of change reported in Fig. 2d of the article (RAF/MAPK up, PI3K inhibition, NFkB, TGFb, JAK-STAT activation)
- pathway_score_deltas.csv must exist with the fields pathway, condition, delta_score
- Signature dimensionality check: the model selects the 100 most significant genes per pathway and uses their z-scores as coefficients
- Benchmark check: PROGENy scores should recover perturbed pathways better than GO/Reactome GSVA, SPIA, Pathifier, and PARADIGM on the perturbation-recall analysis (Fig. 2)

## Limitations

- PROGENy provides a good estimate of pathway activity in large and heterogeneous data sets, but signatures derived from a specific tissue may more closely reflect activation status in that context
- More studies are required to fully elucidate the differences between the common perturbation response and additional transcriptional modules that may not always be activated
- Pathway scores from gene expression do not account for post-translational modifications, so it is unclear under some circumstances whether they reflect true signaling activity
- Some perturbation experiments are excluded from signature building due to QC failure or ambiguous perturbation-phenotype correspondence, as flagged in the ArrayExpress index files

## Evidence

- [discussion] PROGENy leverages a large compendium of pathway-responsive gene signatures: "We developed PROGENy in order to achieve this. PROGENy leverages a large compendium of pathway-responsive gene signatures"
- [abstract] these results show that PROGENy accurately infers pathway activity from gene expression in a wide range of conditions: "these results show that PROGENy accurately infers pathway activity from gene expression in a wide range of conditions"
- [readme] We select the 100 most significant genes and use their z-scores as coefficients in the model.: [evidence span withheld — anchor status: external_doc]
- [readme] we fit a linear model on the z-scores with a binary matrix indicating pathway perturbations (incl. perturbations of multiple pathways) as the independent variable.: [evidence span withheld — anchor status: external_doc]
