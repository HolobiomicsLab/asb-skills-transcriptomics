---
name: differential-transcription-factor-expression-analysis
description: 'Use when you need candidate regulatory factors for bifurcation/priming
  analysis from scRNA-Seq: e.g. picking top TFs per lineage (retina: 7 neuronal +
  7 epithelial; liver: top-4 hepatocyte + top-4 cholangiocyte) before CompDICE.'
when_to_use_negative:
- Your input is already a curated, balanced feature (TF) set for each lineage — re-running
  DE selection would be redundant.
- You need priming significance P-values or primed-cell identification rather than
  candidate RF nomination — use CompPrimingPval or FindPrimedCells instead.
- The dataset lacks identifiable terminal cells of both competing lineages, leaving
  no basis for a two-branch differential comparison.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DICE
  role: Supplies the humanTF regulatory-factor list and the FindCandidateFactors differential
    overexpression workflow used to nominate candidate TFs per lineage
  repo: github.com/aet21/DICE
- name: R
  role: Implementation environment for the DICE package and the differential expression/filtering
    steps
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:2d18a0ca9dd7307785b30e19efbdadee863e70e3b7f65556233e7562ebf58e47
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aet21/DICE
  tools:
  - CompDICE
  - DICE
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# differential-transcription-factor-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects lineage-antagonistic candidate regulatory factors (TFs) by restricting an scRNA-Seq matrix to a curated TF list (e.g. the DICE package's humanTF) and performing differential overexpression analysis between terminal cells of competing lineages, producing an equal-sized top-N TF set per lineage for downstream DICE computation.

## When to use

Use when you need candidate regulatory factors for bifurcation/priming analysis from scRNA-Seq: e.g. picking top TFs per lineage (retina: 7 neuronal + 7 epithelial; liver: top-4 hepatocyte + top-4 cholangiocyte) before CompDICE.

## When NOT to use

- Your input is already a curated, balanced feature (TF) set for each lineage — re-running DE selection would be redundant.
- You need priming significance P-values or primed-cell identification rather than candidate RF nomination — use CompPrimingPval or FindPrimedCells instead.
- The dataset lacks identifiable terminal cells of both competing lineages, leaving no basis for a two-branch differential comparison.

## Inputs

- scRNA-Seq expression matrix (count or log-normalized), restricted to transcription factors via the DICE humanTF list
- Cell-type/lineage annotations identifying terminal cells of each competing lineage and multipotent progenitors
- Candidate TF list per lineage from differential expression (e.g. 10 neuronal-specific, 7 epithelial-specific TFs in the retina case)

## Outputs

- Equal-sized top-N TF sets per lineage (e.g. top-7 neuronal and top-7 epithelial TFs)
- Filtered candidate RF expression sub-matrix used as CompDICE input
- Stage-wise or pseudotime-binned TF expression covariation matrix for DICE entropy computation

## How to apply

Load the scRNA-Seq count matrix and restrict it to regulatory factors using the humanTF list shipped with the DICE R-package, since TF expression changes are subtle and genome-wide DE is too noisy. Perform differential expression/overexpression analysis comparing terminal cells of one lineage against the other (or against multipotent progenitors, as in FindCandidateFactors), yielding antagonistic TFs upregulated in each branch — e.g. 17 candidate TFs (10 neuronal-specific, 7 epithelial-specific) in the fetal retina (GSE107618) case. Optionally filter candidates by frequency of expression in the progenitor population (keeping RFs with expression frequency below the mean over all overexpressed RFs). Choose an EQUAL number of top TFs per branch (top-7 per retinal lineage; top-4 hepatocyte vs top-4 cholangiocyte in yangLiver) to avoid biasing downstream DICE inference toward one lineage. The selected TF expression matrix, binned by stage or pseudotime, is then the input for CompDICE distance covariance entropy estimation.

## Related tools

- **DICE** (Supplies the humanTF regulatory-factor list and the FindCandidateFactors differential overexpression workflow used to nominate candidate TFs per lineage) — github.com/aet21/DICE
- **R** (Implementation environment for the DICE package and the differential expression/filtering steps)

## Examples

```
library(DICE); data(humanTF);
```

## Evaluation signals

- The expression matrix was first intersected with the humanTF list before differential testing (TF-restricted, not genome-wide).
- The final candidate set contains an equal number of TFs for each competing branch (e.g. 7 vs 7), avoiding lineage bias.
- Selected TFs show antagonistic expression: each is significantly upregulated in its own terminal lineage versus the other branch.
- Frequency-of-expression filtering in the progenitor population applied (RFs kept only if expression frequency is below the mean over all overexpressed candidates).
- Downstream CompDICE on the selected TFs yields a stage-wise entropy profile with a clear minimum/transition point (e.g. week 13 in fetal retina) that individual TF variances do not show.

## Limitations

- scRNA-Seq has low sensitivity to detect dynamic changes in TF expression, so candidate nomination is noisy and must be restricted to a TF list.
- Sampling of cells in the transition region is often very sparse, weakening differential comparisons near the bifurcation.
- Unequal TF counts per lineage would bias downstream DICE inference toward one branch; candidate counts must be balanced.

## Evidence
