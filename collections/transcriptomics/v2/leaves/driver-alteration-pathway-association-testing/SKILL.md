---
name: driver-alteration-pathway-association-testing
description: Use when you have TCGA-style expression, MAF mutation calls, and GISTIC
  copy-number calls per sample and want to verify that a driver alteration activates
  its corresponding signaling pathway at the transcriptomic-footprint level.
when_to_use_negative:
- You lack matched alteration calls (MAF or GISTIC) per sample and only have expression
  — the ANOVA grouping variable is undefined
- You need pathway activity in cell lines or single-cell data without driver alteration
  annotation — use perturbation-recall or drug-association analyses instead
- The pathway of interest is not among the 11 PROGENy pathways (EGFR, Hypoxia, JAK-STAT,
  MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3518
tools:
- name: progeny
  role: R/Bioconductor package holding the pathway-responsive gene signatures used
    to compute pathway activity scores from the TCGA expression matrix
  repo: https://github.com/saezlab/progeny
- name: decoupler
  role: Recommended downstream method to infer pathway activities from PROGENy signatures
    coupled with a statistical method
  repo: https://github.com/scverse/decoupler
- name: knitr
  role: Used to auto-generate the supplementary report scripts for the analysis pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:e53cdf6743bac6ec3889c93bb12bf6c8d564e82da12a9f0b122b24ed47ef6ed6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/progeny
  tools:
  - decoupler
  - knitr
  - progeny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# driver-alteration-pathway-association-testing

## Summary

Tests whether PROGENy-inferred pathway activity scores are elevated in TCGA tumor samples harboring specific driver alterations (e.g., EGFR amplification from GISTIC, BRAF/TP53/VHL mutations from MAF), using ANOVA per tumor type with FDR correction. This links copy-number and somatic mutation events to functional signaling footprints inferred from RNA expression.

## When to use

Use when you have TCGA-style expression, MAF mutation calls, and GISTIC copy-number calls per sample and want to verify that a driver alteration activates its corresponding signaling pathway at the transcriptomic-footprint level.

## When NOT to use

- You lack matched alteration calls (MAF or GISTIC) per sample and only have expression — the ANOVA grouping variable is undefined
- You need pathway activity in cell lines or single-cell data without driver alteration annotation — use perturbation-recall or drug-association analyses instead
- The pathway of interest is not among the 11 PROGENy pathways (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF)

## Inputs

- TCGA gene expression matrix per tumor type (Firehose release 28 Jan 2016, primary tumors barcode 01A)
- MAF file of TCGA somatic mutation calls (BRAF, TP53, VHL, etc.)
- Thresholded GISTIC copy-number calls (e.g., EGFR amplifications)
- PROGENy pathway-responsive gene weight matrix (progeny R/Bioconductor package)

## Outputs

- pancancer_driver_pathway_fdr.csv with columns: alteration, pathway, FDR
- Per-alteration ANOVA p-values across the 11 PROGENy pathways per tumor type

## How to apply

Load tissue-matched TCGA cohorts from the Firehose 28 Jan 2016 release: the expression matrix, MAF somatic mutation calls, and thresholded GISTIC copy-number values. Compute PROGENy pathway scores per sample with the progeny R package, which infers the activity of 11 signaling pathways (EGFR, MAPK, p53, hypoxia, etc.) from a consensus of pathway-responsive genes. For each alteration–pathway pair within each tumor type, fit an ANOVA of alteration status (e.g., EGFR amplification; BRAF, TP53, VHL mutation) against the pathway score, then Benjamini-Hochberg correct all p-values to FDR. Rationale: PROGENy footprints recover driver effects better than pathway-expression methods because they capture post-transcriptional signaling via perturbation-derived responsive genes. The output is a pan-cancer table of alteration, pathway, and FDR; the paper's benchmark expects EGFR amplification–EGFR/MAPK FDR < 10^-9, BRAF mutation < 10^-10, TP53 mutation < 10^-64, and VHL mutation–hypoxia < 10^-200.

## Related tools

- **progeny** (R/Bioconductor package holding the pathway-responsive gene signatures used to compute pathway activity scores from the TCGA expression matrix) — https://github.com/saezlab/progeny
- **decoupler** (Recommended downstream method to infer pathway activities from PROGENy signatures coupled with a statistical method) — https://github.com/scverse/decoupler
- **knitr** (Used to auto-generate the supplementary report scripts for the analysis pipeline)

## Examples

```
BiocManager::install("progeny"); scores <- progeny::progeny(expr_matrix, scale=TRUE); aov(pathway_score ~ alteration_status, data = tcga_samples)
```

## Evaluation signals

- EGFR amplification shows FDR < 10^-9 against both EGFR and MAPK pathway scores, while pathway-expression methods (Gene Ontology/Reactome GSVA, SPIA, Pathifier, PARADIGM) do not detect it
- BRAF mutation associates with MAPK at FDR < 10^-10 and TP53 mutation with p53 pathway at FDR < 10^-64
- VHL mutation associates with hypoxia at FDR < 10^-200
- Output CSV contains exactly the columns alteration, pathway, FDR after Benjamini-Hochberg correction of all ANOVA p-values
- Pathway scores are computed only for primary tumor samples (01A in barcode) with tissue-matched normal where required

## Limitations

- PROGENy covers only 11 signaling pathways derived from 208 ArrayExpress submissions (580 experiments); alterations outside these pathways cannot be tested
- Consensus signatures may miss context-specific activation: tissue-specific signatures may more closely reflect pathway activation in a given context than the consensus PROGENy signatures
- Gene-expression-based scores do not measure post-translational signaling directly; scores reflect the transcriptional footprint of pathway activity

## Evidence

- [abstract] PROGENy can (i) recover the effect of known driver mutations: "PROGENy can (i) recover the effect of known driver mutations"
- [methods] driver mutation association workflow: [evidence span withheld — anchor status: not_found]
- [readme] README TCGA samples: [evidence span withheld — anchor status: external_doc]
- [readme] README driver mutation analysis: [evidence span withheld — anchor status: external_doc]
- [intro] post-translational caveat: "It is therefore unclear if and under what circumstances the pathway scores obtained by these methods reflect"
