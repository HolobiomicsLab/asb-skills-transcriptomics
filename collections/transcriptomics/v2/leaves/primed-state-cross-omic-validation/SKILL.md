---
name: primed-state-cross-omic-validation
description: Use when a scRNA-Seq analysis has assigned primed states (e.g. via DICE
  FindPrimedCells) and matched scATAC-Seq profiles exist for the same or equivalent
  cells, and you must confirm that priming leaves an epigenomic signature.
when_to_use_negative:
- No matched scATAC-Seq data exist for the cells or cell-types of interest — the article
  notes that SHARE-Seq-style joint profiles were unavailable for some studies, forcing
  an approximate assignment method
- The goal is to infer primed states de novo from scRNA-Seq rather than to validate
  already-assigned primed cells epigenomically (use DICE FindPrimedCells first)
- Experimental perturbation evidence is required for the regulatory factors — the
  study itself performed no experimental validation, only scATAC-Seq validation
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2229
tools:
- name: DICE
  role: Identifies primed states and candidate regulatory factors from scRNA-Seq whose
    epigenomic signature is then validated
  repo: github.com/aet21/DICE
- name: chromVAR
  role: Computes TF motif activity scores from peak-level scATAC-Seq accessibility
    to test whether priming factors (Runx2, Zfhx3, Foxo3, Lmo7) are more active in
    primed cells
  repo: github.com/GreenleafLab/chromVAR
- name: Palantir
  role: Estimates cell-fate probabilities toward competing lineages used as the outcome
    for one-sided Wilcoxon tests
  repo: github.com/dpeerlab/Palantir
- name: Seurat
  role: Log-normalizes the gene-level accessibility count matrix aggregated from scATAC-Seq
    peaks
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:cd87bea7e08b70caca93a145a233fd92aadf59bc3a6ba745f97582246c598580
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aet21/DICE
  tools:
  - DICE
  - Palantir
  - R
  - Seurat
  - chromVAR
  - ggplot2
  - ggpubr
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# primed-state-cross-omic-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validate scRNA-Seq-inferred primed cell states and their regulatory factors in matched scATAC-Seq data by testing whether primed cells show higher chromatin accessibility/TF motif activity and lineage fate probability toward their assigned branch (e.g. SHARE-Seq hair-follicle TACs).

## When to use

Use when a scRNA-Seq analysis has assigned primed states (e.g. via DICE FindPrimedCells) and matched scATAC-Seq profiles exist for the same or equivalent cells, and you must confirm that priming leaves an epigenomic signature.

## When NOT to use

- No matched scATAC-Seq data exist for the cells or cell-types of interest — the article notes that SHARE-Seq-style joint profiles were unavailable for some studies, forcing an approximate assignment method
- The goal is to infer primed states de novo from scRNA-Seq rather than to validate already-assigned primed cells epigenomically (use DICE FindPrimedCells first)
- Experimental perturbation evidence is required for the regulatory factors — the study itself performed no experimental validation, only scATAC-Seq validation

## Inputs

- DICE-derived primed-cell assignments per lineage (e.g. primed vs unprimed TACs from FindPrimedCells)
- Matched scATAC-Seq peak-level count matrix for the same or equivalent cell population (e.g. SHARE-Seq)
- TF binding motif set and regulatory factor list (e.g. humanTF) for chromVAR
- Palantir cell-fate probability object toward each lineage branch

## Outputs

- Per-cell chromVAR TF activity scores for candidate priming factors
- Palantir fate-probability scores per cell per lineage
- Gene-level accessibility matrix aggregated over peaks 3kb upstream of marker genes and gene body (Seurat log-normalized)
- One-sided Wilcoxon rank-sum P-values comparing primed vs unprimed cells
- Boxplot figure with geom_signif annotations (e.g. tac_priming_atac_validation.png)

## How to apply

Load the DICE-derived primed-cell assignments (e.g. HS- vs IRS-primed TACs, excluding overlapping assignments) together with the matched scATAC-Seq peak matrix. Using chromVAR, compute TF activity scores for the candidate regulatory factors (e.g. Runx2, Zfhx3, Foxo3, Lmo7) from peak-level accessibility and their binding motifs; in parallel, run Palantir to estimate cell-fate probabilities toward each competing lineage, aggregate peaks within 3kb upstream of marker genes plus gene body into a gene-level accessibility matrix, and log-normalize it with Seurat. Compare primed versus unprimed TACs with one-sided Wilcoxon rank-sum tests (alternative='greater') on fate probability and TF activity. Visualize group differences with ggplot2/ggpubr boxplots annotated by geom_signif Wilcoxon P-values (e.g. tac_priming_atac_validation.png).

## Related tools

- **DICE** (Identifies primed states and candidate regulatory factors from scRNA-Seq whose epigenomic signature is then validated) — github.com/aet21/DICE
- **chromVAR** (Computes TF motif activity scores from peak-level scATAC-Seq accessibility to test whether priming factors (Runx2, Zfhx3, Foxo3, Lmo7) are more active in primed cells) — github.com/GreenleafLab/chromVAR
- **Palantir** (Estimates cell-fate probabilities toward competing lineages used as the outcome for one-sided Wilcoxon tests) — github.com/dpeerlab/Palantir
- **Seurat** (Log-normalizes the gene-level accessibility count matrix aggregated from scATAC-Seq peaks)

## Examples

```
library(DICE); data(humanTF); primed <- FindPrimedCells(dice.res); wilcox.test(fate.prob[primed.HS], fate.prob[unprimed], alternative='greater')
```

## Evaluation signals

- Primed TACs show significantly higher fate probability toward their assigned lineage under a one-sided Wilcoxon rank-sum test (alternative='greater')
- chromVAR TF activity for the assigned lineage's priming factors (e.g. Runx2/Zfhx3 for HS, Foxo3/Lmo7 for IRS) is significantly higher in primed than unprimed cells
- Overlapping primed-cell assignments between lineages are excluded via intersect before testing
- Figure renders with ggpubr boxplots and geom_signif Wilcoxon P-value annotations and saves successfully (tac_priming_atac_validation.png)
- Gene-level accessibility matrix is derived from peaks within 3kb upstream of marker genes plus gene body and is log-normalized

## Limitations

- When matched same-cell scATAC-Seq (SHARE-Seq) is unavailable, primed states must be assigned in the scATAC-Seq data by an approximate method, which the authors flag as a limitation
- No experimental validation of the regulatory factors was performed; support is purely computational (scATAC-Seq accessibility/motif activity)
- Software versions for chromVAR, Seurat and Palantir dependencies were not fully documented in the article

## Evidence
