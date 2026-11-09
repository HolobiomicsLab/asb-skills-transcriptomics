---
name: scatac-seq-chromvar-tf-activity-scoring
description: Use when validating DICE-predicted primed cell states and their candidate
  regulatory factors (e.g. Runx2, Zfhx3, Foxo3, Lmo7) in matched scATAC-Seq profiles,
  by testing whether motif-derived TF activity is higher in primed versus unprimed
  cells.
when_to_use_negative:
- No matched scATAC-Seq data exist for the cell population — the article notes SHARE-Seq
  profiles were unavailable for one study, forcing approximate assignment of primed
  states in separate scATAC-Seq populations
- The goal is to cluster cells or call peaks from raw scATAC-Seq rather than score
  predefined candidate TF motifs; chromVAR is used here for motif-activity annotation,
  not cluster identification
- Candidate regulatory factors have no known binding motifs, since chromVAR deviation
  scores require motif annotations
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3676
tools:
- name: chromVAR
  role: Computes per-cell TF motif activity (deviation) scores from the peak-level
    scATAC-Seq data for the candidate priming factors
  repo: https://github.com/GreenleafLab/chromVAR
- name: DICE
  role: Provides the primed-state cell assignments and candidate regulatory factors
    (from scRNA-Seq entropy of expression covariation) that the scATAC-Seq scoring
    validates
  repo: https://github.com/aet21/DICE
- name: Palantir
  role: Estimates cell-fate probabilities toward the HS and IRS lineages for every
    TAC, compared alongside chromVAR TF activity
  repo: https://github.com/dpeerlab/Palantir
- name: Seurat
  role: Log-normalizes the gene-level accessibility count matrix aggregated from peaks
    within 3kb upstream of marker genes and gene bodies
- name: ggpubr
  role: Draws boxplots of fate probability and summed TF activity per group with geom_signif
    Wilcoxon significance annotations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:6e0918782e51c99d90a7031ff897bd1c8a852abf14bbf560ffae3dfb9fe00231
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aet21/DICE
  repo_url: https://github.com/GreenleafLab/chromVAR
  tools:
  - DICE
  - Palantir
  - R
  - Seurat
  - chromVAR
  - ggplot2
  - ggpubr
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3676
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# scatac-seq-chromvar-tf-activity-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes transcription factor activity scores from peak-level scATAC-Seq data with chromVAR, using TF binding motifs, to validate primed states (e.g. HS- and IRS-primed TACs) predicted from scRNA-Seq data alone by DICE in matched single-cell multiome data (SHARE-Seq).

## When to use

Use when validating DICE-predicted primed cell states and their candidate regulatory factors (e.g. Runx2, Zfhx3, Foxo3, Lmo7) in matched scATAC-Seq profiles, by testing whether motif-derived TF activity is higher in primed versus unprimed cells.

## When NOT to use

- No matched scATAC-Seq data exist for the cell population — the article notes SHARE-Seq profiles were unavailable for one study, forcing approximate assignment of primed states in separate scATAC-Seq populations
- The goal is to cluster cells or call peaks from raw scATAC-Seq rather than score predefined candidate TF motifs; chromVAR is used here for motif-activity annotation, not cluster identification
- Candidate regulatory factors have no known binding motifs, since chromVAR deviation scores require motif annotations

## Inputs

- Peak-level scATAC-Seq count matrix for cells matched to the scRNA-Seq primed-state assignments (SHARE-Seq)
- DICE-derived primed vs unprimed cell assignments (e.g. HS- and IRS-primed TACs)
- TF binding motif set for candidate regulatory factors (Runx2, Zfhx3, Foxo3, Lmo7)
- Palantir cell-fate probabilities toward HS and IRS lineages
- cisTopic topic scores and marker-gene-associated peak annotations

## Outputs

- Per-cell chromVAR TF activity scores for candidate priming factors
- One-sided Wilcoxon rank-sum P-values comparing primed vs unprimed TACs (alternative='greater')
- ggplot2/ggpubr boxplots of fate probability and summed TF activity per group with geom_signif annotations (tac_priming_atac_validation.png)

## How to apply

Load the peak-level scATAC-Seq profiles for the same cell population whose primed assignments were derived from scRNA-Seq (e.g. DICE HS-primed via Runx2/Zfhx3 and IRS-primed via Foxo3/Lmo7 in SHARE-Seq hair-follicle TACs). Run chromVAR on the peak-count matrix with the binding motifs of the candidate regulatory factors to obtain per-cell TF activity (deviation) scores. Optionally complement these with Palantir cell-fate probabilities toward the HS and IRS lineages, cisTopic topic scores, and gene-level accessibility aggregated over peaks within 3kb upstream of marker genes plus the gene body, log-normalized with Seurat. Compare primed versus unprimed TACs using one-sided Wilcoxon rank-sum tests (alternative='greater') on TF activity and fate probability, then visualize with ggplot2/ggpubr boxplots annotated with geom_signif and save the figure (e.g. tac_priming_atac_validation.png). The rationale is that DICE priming calls from transcriptome covariation entropy should be reflected epigenomically as elevated motif accessibility for the implicated TFs.

## Related tools

- **chromVAR** (Computes per-cell TF motif activity (deviation) scores from the peak-level scATAC-Seq data for the candidate priming factors) — https://github.com/GreenleafLab/chromVAR
- **DICE** (Provides the primed-state cell assignments and candidate regulatory factors (from scRNA-Seq entropy of expression covariation) that the scATAC-Seq scoring validates) — https://github.com/aet21/DICE
- **Palantir** (Estimates cell-fate probabilities toward the HS and IRS lineages for every TAC, compared alongside chromVAR TF activity) — https://github.com/dpeerlab/Palantir
- **Seurat** (Log-normalizes the gene-level accessibility count matrix aggregated from peaks within 3kb upstream of marker genes and gene bodies)
- **ggpubr** (Draws boxplots of fate probability and summed TF activity per group with geom_signif Wilcoxon significance annotations)

## Examples

```
BiocManager::install("chromVAR"); devtools::install_github("aet21/DICE")  # then chromVAR::compute_deviations(atac_counts, motifs) on peak data for Runx2/Zfhx3/Foxo3/Lmo7
```

## Evaluation signals

- Primed TACs show significantly higher chromVAR TF activity toward their assigned lineage (one-sided Wilcoxon alternative='greater', P-values reported via wilcox.test annotations)
- Palantir fate probability toward the assigned lineage is significantly greater in primed than unprimed TACs
- Summed TF activity (Runx2+Zfhx3 for HS; Foxo3+Lmo7 for IRS) separates primed from unprimed groups in the boxplot with geom_signif significance bars
- Gene-level accessibility over marker-gene-associated peaks (3kb upstream + gene body, Seurat log-normalized) is elevated in the primed group
- Overlapping primed cells between competing lineage assignments are excluded before testing

## Limitations

- When SHARE-Seq (same-cell joint profiles) are unavailable, primed states must be assigned in separate scATAC-Seq populations using an approximate method, as the authors note for the liver hematopoiesis study
- No experimental perturbation validates that the identified regulatory factors causally drive priming; validation is purely epigenomic
- chromVAR motif-activity scores may complement rather than outperform other methods for single-cell clustering tasks
- chromVAR installation requires the gsl system library and may need manual installation of dependencies such as GO.db

## Evidence

- [methods] Using chromVAR, compute TF activity scores for Runx2, Zfhx3, Foxo3 and Lmo7 from the peak-level scATAC-Seq data and their binding motifs.: [evidence span withheld — anchor status: not_found]
- [methods] Compare primed versus unprimed TACs with one-sided Wilcoxon rank-sum tests (alternative='greater') on fate probability and TF activity.: [evidence span withheld — anchor status: not_found]
- [readme] The package aims to identify motifs or other genomic annotations associated with variability in chromatin accessibility between individual cells or samples.: [evidence span withheld — anchor status: external_doc]
