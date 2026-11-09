---
name: palantir-cell-fate-probability-comparison
description: Use when DICE has already assigned primed vs unprimed cells toward competing
  lineages and you must validate those assignments against Palantir fate probabilities
  (and, for SHARE-Seq data, matched scATAC-Seq chromVAR TF activity).
when_to_use_negative:
- Only one terminal lineage exists (no bifurcation), so there are no competing Palantir
  fate probabilities and no DICE priming to validate.
- 'No matched scATAC-Seq in the same cells is available but cross-modality validation
  claims are required: the paper notes that without SHARE-Seq profiles it had to invent
  an approximate method to assign corresponding primed states in separate scATAC-Seq
  populations.'
- The goal is to discover primed states and regulatory factors de novo from scRNA-Seq
  alone — use DICE (CompDICE, CompPrimingPval, FindPrimingFactors, FindPrimedCells)
  rather than this validation-only comparison.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3173
tools:
- name: Palantir
  role: Estimates per-cell fate probabilities toward each competing terminal lineage
    for the multipotent population; these probabilities are the primary comparison
    variable between primed and unprimed cells
  repo: dpeerlab/Palantir
- name: DICE
  role: Defines the primed vs unprimed groups and lineage-specific priming factors
    (FindPrimedCells, FindPrimingFactors) whose validation the fate-probability comparison
    tests
  repo: github.com/aet21/DICE
- name: chromVAR
  role: Computes TF motif-activity scores (Runx2, Zfhx3, Foxo3, Lmo7) from peak-level
    SHARE-Seq scATAC-Seq data as the co-tested accessibility readout
  repo: GreenleafLab/chromVAR
- name: Seurat
  role: Log-normalizes the gene-level accessibility count matrix aggregated from scATAC-Seq
    peaks (3kb upstream of each marker gene and gene body)
- name: ggpubr
  role: Draws the per-group boxplots of fate probability and summed TF activity with
    geom_signif Wilcoxon P-value annotations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:42ecd7ff94a61b4179808b338f45a77455660591dde3037cd8fef4ad85ae5638
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aet21/DICE
  repo_url: https://github.com/dpeerlab/Palantir
  tools:
  - DICE
  - Palantir
  - R
  - Seurat
  - chromVAR
  - ggplot2
  - ggpubr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# palantir-cell-fate-probability-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates DICE-predicted primed cell states by comparing Palantir-derived cell-fate probabilities toward each competing lineage between primed and unprimed cells (e.g., HS- vs IRS-primed TACs in SHARE-Seq hair-follicle data, GP vs MEMP in the Ranzoni hematopoiesis data), supported by matched scATAC-Seq chromVAR TF-activity and gene-accessibility tests.

## When to use

Use when DICE has already assigned primed vs unprimed cells toward competing lineages and you must validate those assignments against Palantir fate probabilities (and, for SHARE-Seq data, matched scATAC-Seq chromVAR TF activity).

## When NOT to use

- Only one terminal lineage exists (no bifurcation), so there are no competing Palantir fate probabilities and no DICE priming to validate.
- No matched scATAC-Seq in the same cells is available but cross-modality validation claims are required: the paper notes that without SHARE-Seq profiles it had to invent an approximate method to assign corresponding primed states in separate scATAC-Seq populations.
- The goal is to discover primed states and regulatory factors de novo from scRNA-Seq alone — use DICE (CompDICE, CompPrimingPval, FindPrimingFactors, FindPrimedCells) rather than this validation-only comparison.

## Inputs

- scRNA-Seq expression matrix for the multipotent progenitor population (input to Palantir fate-probability estimation)
- DICE-derived primed/unprimed cell assignments (FindPrimedCells output; e.g., HS- and IRS-primed TACs)
- Palantir cell-fate probability matrix toward each terminal lineage (GP/MEMP, HS/IRS) per cell
- Matched peak-level scATAC-Seq data (SHARE-Seq) with TF binding motifs for chromVAR and marker-gene annotations for the 3kb-upstream + gene-body accessibility aggregation

## Outputs

- Per-cell Palantir fate probabilities toward each competing lineage
- One-sided Wilcoxon rank-sum P-values (alternative='greater') for primed vs unprimed fate probability, chromVAR TF activity, and gene accessibility
- ggplot2/ggpubr boxplot figure (tac_priming_atac_validation.png) with geom_signif significance annotations

## How to apply

Assign primed versus unprimed cells first with DICE (FindPrimedCells) from the scRNA-Seq data, and drop cells assigned as primed to both competing lineages (e.g., intersect of the HS- and IRS-primed TAC vectors) before any statistical comparison. Run Palantir on the multipotent-progenitor population (MPPs in the Ranzoni data, TACs in SHARE-Seq) to estimate per-cell fate probabilities toward each terminal lineage (GP vs MEMP; HS vs IRS). For the epigenomic arm, aggregate scATAC-Seq peaks within 3kb upstream of each marker gene plus the gene body into a gene-level accessibility matrix log-normalized with Seurat, and compute chromVAR TF-activity scores for the lineage priming factors (Runx2/Zfhx3 for HS, Foxo3/Lmo7 for IRS) from peak-level data and their binding motifs. Compare primed vs unprimed groups with one-sided Wilcoxon rank-sum tests (alternative='greater') on fate probability toward the assigned lineage, TF activity, and gene accessibility. Render ggplot2/ggpubr boxplots per group with geom_signif Wilcoxon P-value annotations (saved as tac_priming_atac_validation.png); the rationale is that Palantir's stochastic fate model yields a continuous lineage-commitment readout that must be elevated in DICE-primed cells if the predicted priming is real.

## Related tools

- **Palantir** (Estimates per-cell fate probabilities toward each competing terminal lineage for the multipotent population; these probabilities are the primary comparison variable between primed and unprimed cells) — dpeerlab/Palantir
- **DICE** (Defines the primed vs unprimed groups and lineage-specific priming factors (FindPrimedCells, FindPrimingFactors) whose validation the fate-probability comparison tests) — github.com/aet21/DICE
- **chromVAR** (Computes TF motif-activity scores (Runx2, Zfhx3, Foxo3, Lmo7) from peak-level SHARE-Seq scATAC-Seq data as the co-tested accessibility readout) — GreenleafLab/chromVAR
- **Seurat** (Log-normalizes the gene-level accessibility count matrix aggregated from scATAC-Seq peaks (3kb upstream of each marker gene and gene body))
- **ggpubr** (Draws the per-group boxplots of fate probability and summed TF activity with geom_signif Wilcoxon P-value annotations)

## Examples

```
overlap.v <- intersect(atac.MEMP.prime.cell.v,atac.GP.prime.cell.v)
```

## Evaluation signals

- One-sided Wilcoxon rank-sum tests (alternative='greater') comparing fate probability toward the assigned lineage in primed vs unprimed TACs must yield significant P-values, as reported via the paper's wilcox.test annotations.
- Directionality invariant: primed cells show higher fate probability toward their assigned lineage (HS-primed toward HS; IRS-primed toward IRS) than unprimed TACs.
- Overlapping primed-cell assignments between the two competing lineages are removed (intersect of the two primed-cell vectors) before testing.
- Cross-modality concordance: significant fate-probability differences co-occur with higher chromVAR TF activity for the lineage's factors (Runx2/Zfhx3 for HS; Foxo3/Lmo7 for IRS) and higher gene accessibility over marker-gene-associated peaks (3kb upstream + gene body, Seurat log-normalized).
- Final figure tac_priming_atac_validation.png renders ggplot2/ggpubr boxplots per group with geom_signif annotations showing the one-sided Wilcoxon P-values.

## Limitations

- Without SHARE-Seq-style matched scRNA-Seq & scATAC-Seq in the same cells, cross-modality validation relied on an invented, approximate method to assign corresponding primed states in separate scATAC-Seq populations.
- Validation is purely computational: no experimental work was performed to demonstrate the importance of the identified regulatory factors in priming.
- scRNA-Seq has low sensitivity to detect dynamic changes in TF expression, so TF-activity conclusions should lean on the scATAC-Seq/chromVAR arm rather than RNA counts.
- Palantir >=1.4.5 switched to numpy's PCG64 default PRNG (from the Mersenne Twister), so numerical outputs are expected to differ from previous versions; pin the prior version if exact reproducibility is required (README release notes).

## Evidence

- [readme] Palantir effectively captures the continuity in cell states and the stochasticity in cell fate determination.: [evidence span withheld — anchor status: external_doc]
