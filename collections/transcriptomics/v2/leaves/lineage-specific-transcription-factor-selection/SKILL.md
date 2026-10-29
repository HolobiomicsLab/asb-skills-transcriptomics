---
name: lineage-specific-transcription-factor-selection
description: Use when a scRNA-Seq timecourse with competing downstream lineages (e.g.
  hepatoblast -> hepatocyte vs cholangiocyte) requires choosing which lineage-restricted
  TF regulatory activity (TFA) features to feed into DICE or another priming/bifurcation
  quantifier.
when_to_use_negative:
- Candidate regulatory factors are already known or curated (e.g. via FindCandidateFactors
  overexpression analysis) — the differential branch t-test selection step is redundant
- No terminal lineage labels or pseudotime/diffusion components are available to define
  branch cells for the between-lineage t-tests
- The population is a static multipotent snapshot rather than a timecourse with diverging
  terminal branches — use the DICE priming-factor ranking (FindPrimingFactors LOO)
  workflow instead
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DICE
  role: Downstream consumer of the selected TF panel; CompDICE is computed over the
    chosen lineage-specific TFs binned by DPT
  repo: https://github.com/aet21/DICE
- name: destiny
  role: DiffusionMap (k=30) and DPT used to define terminal branch cells (DC2 > 0.05
    vs < -0.05) for the differential TFA t-tests
- name: Biobase
  role: Loads the yangLiver ExpressionSet and provides the TFA matrix of 22 liver-specific
    TFs used for selection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:ebacb230a44c79f529e4ed3ad85a5340af45eddc1f360dbfd72650ac2b7c0a40
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aet21/DICE
  repo_url: https://github.com/aet21/DICE
  tools:
  - Biobase
  - DICE
  - Hmisc
  - R
  - destiny
  - gridExtra
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# lineage-specific-transcription-factor-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects a small panel of lineage-specific transcription factors (e.g. top-4 hepatocyte and top-4 cholangiocyte TFs from the Yang liver timecourse) whose regulatory activity profiles will serve as the input features for downstream DICE bifurcation analysis.

## When to use

Use when a scRNA-Seq timecourse with competing downstream lineages (e.g. hepatoblast -> hepatocyte vs cholangiocyte) requires choosing which lineage-restricted TF regulatory activity (TFA) features to feed into DICE or another priming/bifurcation quantifier.

## When NOT to use

- Candidate regulatory factors are already known or curated (e.g. via FindCandidateFactors overexpression analysis) — the differential branch t-test selection step is redundant
- No terminal lineage labels or pseudotime/diffusion components are available to define branch cells for the between-lineage t-tests
- The population is a static multipotent snapshot rather than a timecourse with diverging terminal branches — use the DICE priming-factor ranking (FindPrimingFactors LOO) workflow instead

## Inputs

- TFA (transcription factor activity) matrix for liver-specific TFs from the yangLiver ExpressionSet (447 cells across 7 timepoints E10-E17)
- Diffusion map pseudotime (DPT) values per cell with diffusion component DC2 used to label terminal hepatocyte and cholangiocyte branch cells
- Developmental timepoint metadata per cell for the FDR-filtered linear regression of TF differentiation activity

## Outputs

- Ranked list of differentially active TFs per lineage with t-test statistics between terminal branches
- Selected panel of top-4 cholangiocyte TFs (LSR, ELF3, IRF6, BGN) and top-4 hepatocyte TFs (NR1I3, NR1I2, FOXA2, TRIM15) for DICE computation
- Set of TFs passing the FDR threshold 0.05/nrow(yangTFA.m) for increased differentiation activity

## How to apply

First confirm candidate TFs show increased differentiation activity with developmental timepoint by linear regression over the TFA matrix, filtering at an FDR threshold of 0.05/nrow(yangTFA.m). Then define terminal branch cells on a diffusion map pseudotime (destiny DiffusionMap, k=30, DPT rooted at the E10 cell with the largest absolute DC1 value) using a DC2 threshold (e.g. DC2 > 0.05 vs < -0.05 at the last timepoint), and run t-tests of differential TFA between terminal hepatocyte and cholangiocyte branches. Rank TFs by significance and keep the top-4 per lineage — in the Yang liver data this yields cholangiocyte factors LSR, ELF3, IRF6, BGN and hepatocyte factors NR1I3, NR1I2, FOXA2, TRIM15. The rationale is that this compact, lineage-discriminating 8-TF panel makes the DICE covariation signal interpretable and focused on the bifurcation rather than diluting it across all 22 liver-specific TFs.

## Related tools

- **DICE** (Downstream consumer of the selected TF panel; CompDICE is computed over the chosen lineage-specific TFs binned by DPT) — https://github.com/aet21/DICE
- **destiny** (DiffusionMap (k=30) and DPT used to define terminal branch cells (DC2 > 0.05 vs < -0.05) for the differential TFA t-tests)
- **Biobase** (Loads the yangLiver ExpressionSet and provides the TFA matrix of 22 liver-specific TFs used for selection)

## Examples

```
print(summary(factor(sign(stat.m[which(stat.m[,2] < 0.05/nrow(yangTFA.m)),1]))));
```

## Evaluation signals

- Selected factors match the expected panel: cholangiocyte LSR, ELF3, IRF6, BGN and hepatocyte NR1I3, NR1I2, FOXA2, TRIM15 for the Yang liver dataset
- All retained TFs pass the FDR-corrected significance threshold 0.05/nrow(yangTFA.m) for increased differentiation activity with developmental timepoint
- Terminal branch assignment is consistent: cells labeled at the last timepoint with DC2 > 0.05 (hepatocyte) or DC2 < -0.05 (cholangiocyte) show significant differential TFA by t-test
- The final feature set is exactly 8 TFs (top-4 per lineage), compact enough for stable DICE covariation estimation per DPT bin

## Limitations

- TF expression is hard to measure reliably in scRNA-Seq because of the assay's low sensitivity to detect dynamic changes in TF expression, hence selection uses inferred TFA profiles rather than raw expression
- The top-4 factor panel is dataset-specific; branch definitions depend on diffusion map parameters (k=30) and the DC2 +/- 0.05 cutoff, so results may not transfer directly to other datasets
- In the real datasets analyzed, covariation patterns between RFs did not deviate strongly from monotonic linear dependencies, limiting how much the non-linear capability of downstream DICE is exercised

## Evidence
