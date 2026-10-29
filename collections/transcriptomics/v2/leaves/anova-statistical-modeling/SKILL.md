---
name: anova-statistical-modeling
description: Use when you have binary driver-alteration calls (amplification or mutation)
  per sample plus continuous pathway activity scores and need to test which alterations
  activate which pathways.
when_to_use_negative:
- You are scoring pathways by mean expression of pathway gene sets (Gene Ontology
  or Reactome via GSVA) rather than perturbation-based footprints — the article shows
  such pathway-expression methods do not detect the strong EGFR-amplification activation
  that PROGENy does.
- The outcome is continuous patient survival or drug response rather than binary alteration
  status — the article uses separate survival and drug-association analyses for those,
  not ANOVA of alteration status.
- Samples lack matched tumor/normal or the expression matrix and alteration calls
  are not from tissue-matched cohorts — the pipeline assumes tissue-matched TCGA cohorts.
- No replicates per alteration group (single sample per class) — ANOVA of a binary
  factor against the score needs within-group variance to be estimable.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: progeny
  role: Computes the 11 pathway activity scores (EGFR, MAPK, Hypoxia, p53/DDR, etc.)
    per sample that serve as the response variables of the ANOVA models
  repo: github.com/saezlab/progeny
- name: footprints
  role: Repository containing the analyses/tcga_mutation scripts for the driver-mutation
    functional-impact analysis (Fig. 3)
  repo: saezlab/footprints
- name: decoupler
  role: Alternative engine for inferring pathway activities from PROGENy signatures
    with a unified set of statistical methods
  repo: scverse/decoupler
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:a50906cf2c35b909c5d26706572922ec3955c01d44b3d76e58b1aec942c5b661
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - decoupler
  - footprints
  - progeny
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# anova-statistical-modeling

## Summary

Fits per-tumor-type ANOVA models of driver-alteration status (EGFR amplification from GISTIC; BRAF/TP53/VHL mutations from MAF) against PROGENy pathway activity scores, then FDR-corrects the p-values to assemble a pan-cancer association table. This is how the article demonstrates that PROGENy footprints recover the functional impact of known driver mutations on signaling.

## When to use

Use when you have binary driver-alteration calls (amplification or mutation) per sample plus continuous pathway activity scores and need to test which alterations activate which pathways.

## When NOT to use

- You are scoring pathways by mean expression of pathway gene sets (Gene Ontology or Reactome via GSVA) rather than perturbation-based footprints — the article shows such pathway-expression methods do not detect the strong EGFR-amplification activation that PROGENy does.
- The outcome is continuous patient survival or drug response rather than binary alteration status — the article uses separate survival and drug-association analyses for those, not ANOVA of alteration status.
- Samples lack matched tumor/normal or the expression matrix and alteration calls are not from tissue-matched cohorts — the pipeline assumes tissue-matched TCGA cohorts.
- No replicates per alteration group (single sample per class) — ANOVA of a binary factor against the score needs within-group variance to be estimable.

## Inputs

- TCGA tumor expression matrix (Firehose release 28 Jan 2016, primary tumors '01A')
- MAF somatic mutation calls (e.g. BRAF, TP53, VHL driver mutations)
- Thresholded GISTIC copy-number calls (e.g. EGFR amplifications)

## Outputs

- pancancer_driver_pathway_fdr.csv: pan-cancer association table with alteration, pathway, and FDR columns
- Per-alteration per-pathway ANOVA p-values within each tumor type

## How to apply

Load the TCGA expression matrix (Firehose release 28 Jan 2016), MAF somatic mutation calls, and thresholded GISTIC copy-number values for each tissue-matched cohort. Compute PROGENy pathway activity scores per sample from the expression matrix with the progeny R package. For each driver alteration (EGFR amplification from GISTIC calls; BRAF, TP53, VHL mutations from the MAF), fit an ANOVA of alteration status against each PROGENy pathway score within each tumor type. Collect ANOVA p-values, correct for multiple testing to FDR, and assemble a pan-cancer table with alteration, pathway, and FDR columns. The rationale is to place genomic alterations in the functional context of the pathways they alter, i.e. to recover the effect of known driver mutations on signaling activity.

## Related tools

- **progeny** (Computes the 11 pathway activity scores (EGFR, MAPK, Hypoxia, p53/DDR, etc.) per sample that serve as the response variables of the ANOVA models) — github.com/saezlab/progeny
- **footprints** (Repository containing the analyses/tcga_mutation scripts for the driver-mutation functional-impact analysis (Fig. 3)) — saezlab/footprints
- **decoupler** (Alternative engine for inferring pathway activities from PROGENy signatures with a unified set of statistical methods) — scverse/decoupler

## Examples

```
p_scores <- progeny::progeny(expr = tcga_expr); summary(aov(p_scores$EGFR ~ egfr_amp_status, data = tcga))
```

## Evaluation signals

- Reproduced FDRs hit the reported orders of magnitude: EGFR amplification < 10^-9 (activating both EGFR and MAPK), BRAF mutation < 10^-10, TP53 mutation < 10^-64, VHL mutation–hypoxia < 10^-200.
- Output table contains the expected columns: alteration, pathway, and FDR (p-values must be multiple-testing corrected before comparison).
- Pathway scores were computed with the progeny package (11 pathways: EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF), not by mean expression of GO/Reactome gene sets.
- Association tests are run within each tumor type on tissue-matched cohorts, then assembled pan-cancer, rather than pooled across all tumor types.
- EGFR amplification must activate both EGFR and MAPK scores — a single-pathway signal indicates the wrong scoring or test setup.

## Limitations

- Pathway-expression methods (e.g. Gene Ontology/Reactome gene-set scoring) do not detect the strong EGFR-amplification activation; results depend on using perturbation-derived PROGENy footprints.
- PROGENy's consensus signatures may be less accurate than tissue-specific signatures in a given context; a signature derived from a specific tissue may more closely reflect activation status there.
- Expression-based pathway scores cannot account for post-translational modifications, so inference of activity from transcriptomics is an approximation.
- Score computation on primary tumors ('01A') required tissue-matched normals ('11A') for some methods (SPIA, Pathifier) in the original analysis.

## Evidence

- [abstract] recover effect of driver mutations: "PROGENy can (i) recover the effect of known driver mutations"
- [intro] alterations in pathway context: "putting the genomic alterations investigated in the functional context of the pathways they alter may provide additional information on mechanisms of pathogenesis"
- [discussion] pathway expression methods insufficient: "The latter can be used for many more pathways, as information on the pathway components is more often available than perturbation experiments. However, our results indicate that one should be"
- [readme] TCGA scoring setup: [evidence span withheld — anchor status: not_found]
- [readme] driver mutation analysis location: "#### Functional impact of driver mutations (Fig. 3)

Analyses available in [analyses/tcga_mutation](analyses/tcga_mutation)."
- [readme] pathway list: " * EGFR
 * Hypoxia
 * JAK-STAT
 * MAPK
 * NFkB
 * p53/DDR (DNA damage response)
 * PI3K
 * TGFb
 * Trail
 * VEGF (and PDGF)"
