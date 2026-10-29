---
name: tcga-multi-omics-data-integration
description: Use when you have tissue-matched TCGA expression, MAF mutation calls,
  and GISTIC copy-number calls and need to test whether driver alterations (e.g. EGFR
  amplification, BRAF/TP53/VHL mutations) activate their corresponding signaling pathways.
when_to_use_negative:
- When the question requires phosphoprotein-level pathway activity rather than expression-inferred
  footprints, since PROGENy does not account for post-translational modifications
  directly
- When studying context-specific or tissue-specific pathway activation where tissue-derived
  signatures may better reflect activation than the consensus PROGENy signatures
- When the input is a single small sample set without driver alteration status — comparing
  against perturbation experiments or tissue-matched normals was required for some
  competing methods (SPIA, Pathifier), and pathway score interpretation needs sufficient
  sample size
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: PROGENy
  role: Computes per-sample pathway activity (footprint) scores from the TCGA expression
    matrix for the 11 signaling pathways used in the ANOVA association tests
  repo: https://github.com/saezlab/progeny
- name: PARADIGM
  role: One of the alternative pathway methods used to compute pathway scores on TCGA
    primary tumors for comparison against PROGENy footprints
  repo: https://github.com/sbenz/Paradigm
- name: decoupler
  role: Recommended downstream method for inferring pathway activities from PROGENy
    signatures coupled with statistical methods in R or Python
  repo: https://github.com/scverse/decoupler
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:17b3ed337e153c9b6c957b522e59017ec9793159dad9b702e2a6204592729ac9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/progeny
  tools:
  - PARADIGM
  - PROGENy
  - decoupler
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# tcga-multi-omics-data-integration

## Summary

Integrates TCGA multi-omics data (RNA-seq expression, MAF somatic mutation calls, GISTIC copy-number values) by computing PROGENy pathway activity scores per sample and testing their association with driver alterations within each tumor type, demonstrating that expression-inferred signaling footprints recover functional effects of known driver mutations.

## When to use

Use when you have tissue-matched TCGA expression, MAF mutation calls, and GISTIC copy-number calls and need to test whether driver alterations (e.g. EGFR amplification, BRAF/TP53/VHL mutations) activate their corresponding signaling pathways.

## When NOT to use

- When the question requires phosphoprotein-level pathway activity rather than expression-inferred footprints, since PROGENy does not account for post-translational modifications directly
- When studying context-specific or tissue-specific pathway activation where tissue-derived signatures may better reflect activation than the consensus PROGENy signatures
- When the input is a single small sample set without driver alteration status — comparing against perturbation experiments or tissue-matched normals was required for some competing methods (SPIA, Pathifier), and pathway score interpretation needs sufficient sample size

## Inputs

- TCGA RNA-seq expression matrix (Firehose release 28 Jan 2016)
- MAF file of TCGA somatic mutation calls (e.g. BRAF, TP53, VHL driver mutations)
- Thresholded GISTIC copy-number values (e.g. EGFR amplification calls)
- TCGA cohort/tumor-type labels for tissue-matched grouping

## Outputs

- Per-sample PROGENy pathway activity score matrix (11 pathways: EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF)
- Pan-cancer driver-pathway association table written to pancancer_driver_pathway_fdr.csv with alteration, pathway, and FDR columns

## How to apply

Load the TCGA expression matrix (Firehose release 28 Jan 2016), MAF somatic mutation calls, and thresholded GISTIC copy-number values for tissue-matched cohorts, using primary tumors ('01A' in barcode) where a tissue-matched normal ('11A') is available. Compute PROGENy pathway activity scores per sample from the expression matrix using the progeny R package, whose 11 pathways (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF) were derived as a common core of perturbation-responsive genes from ArrayExpress perturbation experiments. For each driver alteration (GISTIC amplifications such as EGFR; MAF mutations such as BRAF, TP53, VHL), fit an ANOVA of alteration status against each pathway score within each tumor type, then correct all ANOVA p-values to FDR across the pan-cancer table (alteration, pathway, FDR). The rationale is that PROGENy footprints capture post-transcriptional pathway activity better than gene-set expression methods, so they can recover the effect of known driver mutations.

## Related tools

- **PROGENy** (Computes per-sample pathway activity (footprint) scores from the TCGA expression matrix for the 11 signaling pathways used in the ANOVA association tests) — https://github.com/saezlab/progeny
- **PARADIGM** (One of the alternative pathway methods used to compute pathway scores on TCGA primary tumors for comparison against PROGENy footprints) — https://github.com/sbenz/Paradigm
- **decoupler** (Recommended downstream method for inferring pathway activities from PROGENy signatures coupled with statistical methods in R or Python) — https://github.com/scverse/decoupler

## Examples

```
library(progeny); progeny_scores <- progeny(expr_matrix, scale = TRUE); summary(aov(progeny_scores$EGFR ~ egfr_gistic_amplification))
```

## Evaluation signals

- EGFR amplification associates with EGFR and MAPK pathway scores at FDR < 10^-9
- BRAF mutation shows FDR < 10^-10, TP53 mutation FDR < 10^-64, and VHL mutation–hypoxia FDR < 10^-200 in the association table
- Association table pancancer_driver_pathway_fdr.csv contains exactly the alteration, pathway, FDR columns and FDR-corrected ANOVA p-values
- Pathway scores are computed only for primary tumors with '01A' in the barcode where tissue-matched data is available
- Other pathway expression methods (Gene Ontology/Reactome GSVA, SPIA, Pathifier, PARADIGM) should fail to detect the strong EGFR amplification activation that PROGENy recovers

## Limitations

- Expression-based pathway scores do not take into account post-translational modifications of signaling
- Tissue-specific signatures may more closely reflect pathway activation status than the consensus PROGENy signatures in a given context
- More studies are required to fully elucidate the differences between the common perturbation response and additional transcriptional modules that may not always be activated

## Evidence

- [abstract] PROGENy can recover the effect of known driver mutations: "PROGENy can (i) recover the effect of known driver mutations"
- [intro] Placing alterations in pathway context motivates the analysis: "putting the genomic alterations investigated in the functional context of the pathways they alter may provide additional information"
- [readme] TCGA primary tumor data source and sample selection: [evidence span withheld — anchor status: external_doc]
- [readme] The 11 PROGENy pathways covered: " * EGFR
 * Hypoxia
 * JAK-STAT
 * MAPK
 * NFkB
 * p53/DDR (DNA damage response)"
- [intro] Expression-based pathway methods neglect PTMs: "do not take into account the effect of post-translational modifications (Fig. 1a)"
