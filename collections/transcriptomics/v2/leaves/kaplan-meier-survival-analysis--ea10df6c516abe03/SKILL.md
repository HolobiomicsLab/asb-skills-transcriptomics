---
name: kaplan-meier-survival-analysis
description: Use when patient samples have been partitioned into subgroups (e.g.,
  from METAFlux flux-profile clustering of TCGA LUAD/LUSC) and you must test whether
  group membership predicts time-to-event survival outcomes.
when_to_use_negative:
- Input samples lack patient-level time-to-event clinical metadata, so no survival
  comparison is possible (e.g., cell-line or NCI-60 flux data)
- The analysis goal is flux prediction itself (MRAS calculation, compute_flux with
  human_blood or cell_medium) rather than testing clinical relevance of resulting
  subgroups
- Subgroup labels are not yet defined — use clustering (e.g., Seurat-based clustering
  and UMAP embedding) to define groups before survival testing
tools:
- name: R (survival package)
  role: Fits Kaplan-Meier survival curves and performs the log-rank test comparing
    LUAD1 vs 'LUSC-like' LUAD survival
- name: METAFlux
  role: Produces the per-sample flux profiles (compute_flux, human_blood medium) from
    which the clinically compared LUAD subgroups are derived
  repo: https://github.com/KChen-lab/METAFlux
- name: Seurat
  role: Supports clustering of flux profiles and UMAP embedding used to identify the
    LUSC-like LUAD group prior to survival analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:f009c95eec507a5fcc2e2fa3adc99ca103fa9ffe8710cf73d75bd5c664428b46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  - R (survival package)
  - Seurat
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# kaplan-meier-survival-analysis

## Summary

Kaplan-Meier survival analysis with log-rank testing to link metabolically defined tumor subgroups to patient outcomes, as done in METAFlux where TCGA LUAD flux-based clusters (LUAD1 vs 'LUSC-like' LUADs) were compared for survival (log-rank P = 0.0084).

## When to use

Use when patient samples have been partitioned into subgroups (e.g., from METAFlux flux-profile clustering of TCGA LUAD/LUSC) and you must test whether group membership predicts time-to-event survival outcomes.

## When NOT to use

- Input samples lack patient-level time-to-event clinical metadata, so no survival comparison is possible (e.g., cell-line or NCI-60 flux data)
- The analysis goal is flux prediction itself (MRAS calculation, compute_flux with human_blood or cell_medium) rather than testing clinical relevance of resulting subgroups
- Subgroup labels are not yet defined — use clustering (e.g., Seurat-based clustering and UMAP embedding) to define groups before survival testing

## Inputs

- Per-sample reaction flux matrix (13,082 Human-GEM reactions x TCGA samples, exported as CSV) or cluster assignments derived from it
- Patient clinical survival metadata for TCGA samples (time-to-event and censoring status)
- Group/cluster labels assigning each patient to e.g. LUAD1 vs 'LUSC-like' LUAD

## Outputs

- Kaplan-Meier survival curve per subgroup (PNG, e.g., LUAD1 vs 'LUSC-like' LUAD)
- Log-rank test P value comparing subgroup survival (target reproduction: P = 0.0084)
- Per-group survival summary (number of patients, events, median survival)

## How to apply

First generate per-sample clinical groups from the biology: compute METAFlux fluxes for TCGA LUAD/LUSC samples using compute_flux with the human_blood medium, cluster samples into two clusters based on their flux profiles, and identify the metabolically more active 'LUSC-like' LUAD subgroup. Then, with the survival R package, fit Kaplan-Meier curves of overall survival for each group (e.g., LUAD1 vs 'LUSC-like' LUADs) using patient time-to-event data, and assess separation with a log-rank test. In the METAFlux paper's TCGA LUAD/LUSC application this test yielded a significant P value of 0.0084, showing LUAD1 tumors had better survival — a result not obtainable from metabolic gene expression data alone. Export the survival curve as a PNG alongside the flux matrix and glucose uptake (HMR_9034) comparison to make the clinical association fully traceable.

## Related tools

- **R (survival package)** (Fits Kaplan-Meier survival curves and performs the log-rank test comparing LUAD1 vs 'LUSC-like' LUAD survival)
- **METAFlux** (Produces the per-sample flux profiles (compute_flux, human_blood medium) from which the clinically compared LUAD subgroups are derived) — https://github.com/KChen-lab/METAFlux
- **Seurat** (Supports clustering of flux profiles and UMAP embedding used to identify the LUSC-like LUAD group prior to survival analysis)

## Examples

```
survdiff(Surv(OS_time, OS_status) ~ cluster, data = luad_meta)  # log-rank LUAD1 vs LUSC-like LUAD, expect P = 0.0084
```

## Evaluation signals

- Reproduced log-rank P value for LUAD1 vs 'LUSC-like' LUAD comparison matches the reported P = 0.0084
- Survival curves show LUAD1 tumors with better survival outcomes than the 'LUSC-like' LUAD cluster (correct directionality)
- Flux matrix exported as CSV, glucose uptake table (HMR_9034), UMAP PNG, and survival curve PNG all present in the output directory
- Each TCGA LUAD patient appears in exactly one survival group; group sizes are plausible for the two-cluster solution
- Clustering of flux profiles (post optional cbrt normalization) cleanly separates the LUSC-like LUAD subgroup before survival testing

## Limitations

- Survival result depends on upstream choices: the two-cluster flux-profile clustering, the human_blood medium constraint, and optional cubic-root (cbrt) normalization of flux scores can all change subgroup composition
- No software version numbers are stated for METAFlux, R, or Seurat in the provided text, so exact reproducibility of the P = 0.0084 value may vary across versions
- Flux-based subgroup survival association is observational; the reported difference does not establish causation between glucose uptake phenotype and outcome

## Evidence

- [other] Task card finding: [evidence span withheld — anchor status: not_found]
- [methods] Methods flux computation: "flux<-compute_flux(mras=scores,medium = human_blood)"
- [readme] README capability statement: [evidence span withheld — anchor status: external_doc]
