---
name: melanoma-prognostic-t-cell-subpopulation-analysis
description: Use when bulk transcriptomic data with matched survival (e.g., TCGA SKCM06)
  must be dissected at T cell-subpopulation resolution using scRNA-seq-derived markers
  to compare prognostic associations of Tex subtypes.
when_to_use_negative:
- The bulk cohort is small or lacks matched survival information — TimiGP requires
  a large gene expression dataset with matched survival data to identify prognostic
  gene pairs.
- The goal is immunotherapy response prediction rather than prognosis — use the TimiGP-Response
  module instead of the prognosis workflow.
- High-resolution single-cell data are already available — direct scRNA-seq analysis
  or dedicated interaction tools (e.g., scSeqComm) may be more appropriate than marker-pair
  inference from bulk data.
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: TimiGP
  role: 'Core R package implementing the gene-pair framework: preprocessing, marker
    pairing, Cox selection, enrichment, and favorability scoring'
  repo: https://github.com/CSkylarL/TimiGP
- name: TimiCOX
  role: Performs univariate Cox regression on each marker pair with BH adjustment;
    pairs with HR>1 are reversed to orient toward favorable prognosis
  repo: https://github.com/CSkylarL/TimiGP
- name: TimiPermFDR
  role: Computes permutation-based FDR (niter=100) for cell-cell interaction enrichment
    results
  repo: https://github.com/CSkylarL/TimiGP
- name: TimiFS
  role: Computes per-cell-type favorability scores from network out-degrees (favorable)
    and in-degrees (unfavorable)
  repo: https://github.com/CSkylarL/TimiGP
- name: devtools
  role: Used to install the TimiGP package from GitHub
  repo: https://github.com/r-lib/devtools
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:e6c144e3555e13b463a4793373843f4fba895689af1e6ef5838613ed12382cdd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiCOX
  - TimiFS
  - TimiGP
  - TimiPermFDR
  - devtools
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# melanoma-prognostic-t-cell-subpopulation-analysis

## Summary

A TimiGP-based workflow that applies scRNA-seq-derived T cell-subpopulation markers to bulk metastatic melanoma data (TCGA SKCM06) to resolve prognostic heterogeneity among CD8+ exhausted T cell (Tex) subtypes, showing CD8+ GZMK+ Tex has a less favorable prognostic association than CD8+ TCF7+ Tex.

## When to use

Use when bulk transcriptomic data with matched survival (e.g., TCGA SKCM06) must be dissected at T cell-subpopulation resolution using scRNA-seq-derived markers to compare prognostic associations of Tex subtypes.

## When NOT to use

- The bulk cohort is small or lacks matched survival information — TimiGP requires a large gene expression dataset with matched survival data to identify prognostic gene pairs.
- The goal is immunotherapy response prediction rather than prognosis — use the TimiGP-Response module instead of the prognosis workflow.
- High-resolution single-cell data are already available — direct scRNA-seq analysis or dedicated interaction tools (e.g., scSeqComm) may be more appropriate than marker-pair inference from bulk data.

## Inputs

- SKCM06info: TCGA SKCM06 (metastatic melanoma) clinical info with event and time-to-event fields
- SKCM06rna: TCGA SKCM06 bulk transcriptome profile (RNA-seq)
- scRNA-seq-derived T cell subpopulation marker gene list (Tirosh2016/Zheng2021 marker sets)

## Outputs

- Marker Pair Score (MPS) matrix from TimiGenePair
- Directed marker pair Cox regression results with BH-adjusted p-values (favorable-oriented pairs)
- Cell-cell interaction enrichment results with permutation FDR (TimiPermFDR, niter=100)
- favorability_scores.csv: per-cell-type favorability scores from TimiFS

## How to apply

Load SKCM06info and SKCM06rna, filter clinical info with TimiCheckEvent, and preprocess the transcriptome with TimiPrePropress (extract the scRNA-seq marker subset, log transformation, gene-wise median normalization). Generate the Marker Pair Score matrix with TimiGenePair (logical pairwise comparison, cont=FALSE), then run TimiCOX univariate Cox regression on all marker pairs with BH adjustment, reversing pairs with HR>1 so all selected pairs orient toward favorable prognosis. Annotate cell interactions with TimiCellPair from the Tirosh2016/Zheng2021 marker sets, build the enrichment background with TimiBG, and select significant gene pairs (QV<0.05) for TimiEnrich (BH adjustment) with TimiPermFDR permutation FDR (niter=100). Finally compute the favorability score per cell type with TimiFS (out-degree/sum of out-degrees*100 for favorable, in-degree/sum of in-degrees*100 for unfavorable) and export favorability_scores.csv; the rationale is that gene-pair relations circumvent immune co-infiltration bias that confounds abundance-based cell-type prognostic estimates.

## Related tools

- **TimiGP** (Core R package implementing the gene-pair framework: preprocessing, marker pairing, Cox selection, enrichment, and favorability scoring) — https://github.com/CSkylarL/TimiGP
- **TimiCOX** (Performs univariate Cox regression on each marker pair with BH adjustment; pairs with HR>1 are reversed to orient toward favorable prognosis) — https://github.com/CSkylarL/TimiGP
- **TimiPermFDR** (Computes permutation-based FDR (niter=100) for cell-cell interaction enrichment results) — https://github.com/CSkylarL/TimiGP
- **TimiFS** (Computes per-cell-type favorability scores from network out-degrees (favorable) and in-degrees (unfavorable)) — https://github.com/CSkylarL/TimiGP
- **devtools** (Used to install the TimiGP package from GitHub) — https://github.com/r-lib/devtools

## Examples

```
devtools::install_github("CSkylarL/TimiGP"); rna_m <- TimiPrePropress(rna = SKCM06rna, marker = Tcell_markers); mps <- TimiGenePair(data = rna_m, cont = FALSE); res <- TimiCOX(mps, SKCM06info, p.adj = 'BH')
```

## Evaluation signals

- Stromal and tumor cells receive unfavorable favorability scores in the TimiFS output.
- GZMK+ CD8 T cells and CD4 Tems receive favorable favorability scores.
- CD8+ GZMK+ Tex shows a less favorable prognostic association than CD8+ TCF7+ Tex, matching the original scRNA-seq analysis (Tirosh2016).
- All selected marker pairs from TimiCOX are oriented toward favorable prognosis (HR flipped when HR>1).
- Significant gene pairs pass QV<0.05 after BH adjustment and TimiPermFDR permutation (niter=100).

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs.
- Sensitivity to identify a specific cell-cell interaction is affected by the number of corresponding marker genes; cell types with fewer marker genes are less likely to be identified as significant.
- No wet-lab experiments were performed to validate the important findings, raising uncertainty about the reliability of inferred cell-cell interactions.

## Evidence

- [other] TimiGP demonstrated prognostic heterogeneity among CD8+ exhausted T cell subtypes, supporting that CD8+ GZMK+ Tex has a less favorable prognostic association than CD8+ TCF7+ Tex: [evidence span withheld — anchor status: not_found]
