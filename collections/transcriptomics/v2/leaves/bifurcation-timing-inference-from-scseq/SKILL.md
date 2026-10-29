---
name: bifurcation-timing-inference-from-scseq
description: Use when a scRNA-Seq timecourse or pseudotyped dataset captures a binary
  lineage branching (e.g. hepatoblast to hepatocyte vs cholangiocyte) and the goal
  is to locate the transition/tipping point along pseudotime.
when_to_use_negative:
- The dataset is a static snapshot without pseudotime or lineage structure, where
  CompDICE/FindPrimingCells priming analysis (not bifurcation timing along a trajectory)
  is the appropriate use
- Only individual TF expression variances are available and no multi-TF regulatory
  activity matrix can be constructed — variance-based tipping detection is weaker
  and the article shows asynchronous variance increases miss the transition
- The lineage topology has more than two competing branches, since top-4-per-lineage
  TF selection and the two-branch DC2 thresholding (DC2 > 0.05 vs < -0.05) assume
  a binary bifurcation
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: DICE
  role: R package providing CreateDPTbin and CompDICE to bin pseudotime and compute
    distance covariance entropy per bin over selected TFs
  repo: github.com/aet21/DICE
- name: destiny
  role: DiffusionMap (k=30) and DPT to derive pseudotime rooted at the E10 cell with
    largest absolute DC1
- name: Biobase
  role: Loading the yangLiver ExpressionSet and extracting the TFA matrix
- name: Hmisc
  role: errbar to plot DICE mean +/- 1.96*SD bootstrap intervals with a symmetric
    sigmoid fit
- name: gridExtra
  role: Arranging the DICE trajectory figure panels
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:aa7fb3c5f462747f5d6a3723ef313893a24ae751d2fb86b2246754a668ad8be2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aet21/DICE
  tools:
  - Biobase
  - DICE
  - Hmisc
  - R
  - destiny
  - gridExtra
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# bifurcation-timing-inference-from-scseq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A method that computes Distance Covariance Entropy (DICE) over lineage-specific transcription-factor regulatory activity profiles binned along diffusion pseudotime to pinpoint the timing of a cell-fate bifurcation from scRNA-Seq data, more precisely than trajectory-inference or variance-based approaches.

## When to use

Use when a scRNA-Seq timecourse or pseudotyped dataset captures a binary lineage branching (e.g. hepatoblast to hepatocyte vs cholangiocyte) and the goal is to locate the transition/tipping point along pseudotime.

## When NOT to use

- The dataset is a static snapshot without pseudotime or lineage structure, where CompDICE/FindPrimingCells priming analysis (not bifurcation timing along a trajectory) is the appropriate use
- Only individual TF expression variances are available and no multi-TF regulatory activity matrix can be constructed — variance-based tipping detection is weaker and the article shows asynchronous variance increases miss the transition
- The lineage topology has more than two competing branches, since top-4-per-lineage TF selection and the two-branch DC2 thresholding (DC2 > 0.05 vs < -0.05) assume a binary bifurcation

## Inputs

- scRNA-Seq ExpressionSet covering a differentiation timecourse (e.g. yangLiver, 447 cells across E10-E17)
- TFA matrix of lineage-relevant transcription factors (e.g. 22 liver-specific TFs)
- Diffusion pseudotime vector computed over the TF profiles (destiny DiffusionMap, k=30, DPT)
- Precomputed bootstrap null (data(diceBoot)) or ability to run 1000 resamples per bin

## Outputs

- DICE value per pseudotime bin over the 8 selected lineage TFs
- Bootstrap mean +/- 1.96*SD error bars per bin and a symmetric sigmoid fit
- Bifurcation timing readout (DPT-bin interval mapped to developmental timepoint, e.g. between bins 5 and 6, after E14/E14.5)
- Figure of DICE versus DPT-bin (dice_vs_dptbin.png)

## How to apply

First derive lineage-specific transcription-factor activity (TFA) profiles — in the Yang liver timecourse, t-tests of differential TFA between terminal hepatocyte (DC2 > 0.05) and cholangiocyte (DC2 < -0.05) branches select top-4 TFs per lineage (NR1I3, NR1I2, FOXA2, TRIM15 vs LSR, ELF3, IRF6, BGN). Infer a diffusion map over the TFs with destiny's DiffusionMap (k=30), compute DPT rooted at the earliest-timepoint cell with largest absolute DC1, and bin pseudotime with CreateDPTbin(minBins=7). Then run CompDICE per bin on the 8 selected TFs, assess sampling variability by bootstrapping DICE over 1000 resamples per bin (or use the precomputed diceBoot data), and plot mean +/- 1.96*SD with Hmisc errbar plus a symmetric sigmoid fit. The bifurcation timing is read off where DICE transitions — in the Yang dataset between DPT-bin 5 and 6, immediately after E14/E14.5 — and cross-checking bin membership against developmental timepoints (E10-E17). Rationale: DICE on combined lineage TF panels shows clear transition-like behavior whereas individual TF variances increase only asynchronously, giving sharper bifurcation timing.

## Related tools

- **DICE** (R package providing CreateDPTbin and CompDICE to bin pseudotime and compute distance covariance entropy per bin over selected TFs) — github.com/aet21/DICE
- **destiny** (DiffusionMap (k=30) and DPT to derive pseudotime rooted at the E10 cell with largest absolute DC1)
- **Biobase** (Loading the yangLiver ExpressionSet and extracting the TFA matrix)
- **Hmisc** (errbar to plot DICE mean +/- 1.96*SD bootstrap intervals with a symmetric sigmoid fit)
- **gridExtra** (Arranging the DICE trajectory figure panels)

## Examples

```
library(DICE); data(diceBoot); bin.o <- CreateDPTbin(dpt.v,minBins=7); dice.v <- CompDICE(tfa.sub.m, bin.o)
```

## Evaluation signals

- TF differentiation activity increases significantly with developmental timepoint (linear regression, FDR threshold 0.05/nrow(yangTFA.m))
- Top-4 TFs per lineage match the expected panels: LSR, ELF3, IRF6, BGN (cholangiocyte) and NR1I3, NR1I2, FOXA2, TRIM15 (hepatocyte)
- CreateDPTbin produces at least minBins=7 bins and bin membership cross-tabulates consistently with developmental timepoints E10-E17
- Bootstrapped DICE per bin shows a clear transition-like rise (fit by a symmetric sigmoid) rather than noisy asynchronous increases; bootstrap variability from 1000 resamples stays within error bars
- The read-off bifurcation falls between DPT-bin 5 and 6, immediately after E14/E14.5, matching the published result

## Limitations

- Individual TF variances increase only asynchronously and do not pinpoint the transition — DICE over the combined 8-TF panel is required for a sharp signal
- Sampling of cells in the transition region is often sparse, hence the need for bootstrapping (1000 resamples per bin) or the precomputed diceBoot data
- DICE displayed smoother patterns than DNB-based methods such as BioTIP, but a comprehensive comparison with other tipping-point algorithms such as scGET has not been performed

## Evidence

- [abstract] DICE can pinpoint the timing of bifurcations more precisely than lineage-trajectory inference algorithms or competing variance-based methods.: "DICE can pinpoint the timing of bifurcations more precisely than lineage-trajectory inference algorithms or competing variance-based methods."
- [methods] bin.o <- CreateDPTbin(dpt.v,minBins=7);: "bin.o <- CreateDPTbin(dpt.v,minBins=7);"
- [methods] we thus select the top-4 cholangiocyte (LSR, ELF3, IRF6 and BGN) and hepatocyte factors (NR1I3, NR1I2, FOXA2, TRIM15), respectively.: "we thus select the top-4 cholangiocyte (LSR, ELF3, IRF6 and BGN) and hepatocyte factors (NR1I3, NR1I2, FOXA2, TRIM15), respectively."
- [readme] DICE is an R-package which uses the concept of Distance Covariance Entropy to help quantify bifurcation dynamics from scRNA-Seq data: "DICE is an R-package which uses the concept of Distance Covariance Entropy to help quantify bifurcation dynamics from scRNA-Seq data"
- [intro] Accurately identifying cell-fate transition points from scRNA-Seq data is difficult because sampling of cells from the transition region is often very sparse: "Accurately identifying cell-fate transition points from scRNA-Seq data is difficult because sampling of cells from the transition region is often very sparse"
