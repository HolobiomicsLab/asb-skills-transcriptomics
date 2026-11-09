---
name: signalling-entropy-vs-connectome-agreement-analysis
description: Use when you have single-cell expression data with SCENT signalling entropy
  (SR) values and want to test or use the CCAT connectome-correlation score as a fast
  proxy for SR/differentiation potency.
when_to_use_negative:
- You need the full signalling entropy (SR) estimate itself with its network information,
  not a fast proxy — CCAT is a correlation-based surrogate for SR
- Other entropy-based measures such as StemID or SLICE would be used instead when
  transcriptomic entropy without network information is desired
- Your goal is pseudo-time ordering with algorithms like Monocle or MPath rather than
  potency estimation via SCENT/CCAT
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_0203
tools:
- name: SCENT
  role: R package providing the CCAT function and the example Chu et al. dataset;
    installed from GitHub to compute per-cell connectome-transcriptome correlations
    and SR
  repo: aet21/SCENT
- name: CCAT
  role: Function within SCENT that computes, for every cell, the Pearson correlation
    between its transcriptome profile and the PPI connectome
- name: R
  role: Language in which the CCAT computation, linear regression of CCAT vs SR, and
    plotting are performed
- name: BiocStyle
  role: Bioconductor style used for the SCENT tutorial documentation workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:dc72653d8733ea1500c13d6fcb0f8e488d70b06b794590bae672c5515c46a0cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - BiocStyle
  - CCAT
  - R
  - SCENT
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# signalling-entropy-vs-connectome-agreement-analysis

## Summary

Computes the CCAT (connectome-transcriptome Pearson correlation) score for every single cell using the SCENT package and validates it as a proxy for full signalling entropy (SR) by fitting a linear regression and reporting the coefficient of determination, which reaches R2=0.96 on the Chu et al. dataset.

## When to use

Use when you have single-cell expression data with SCENT signalling entropy (SR) values and want to test or use the CCAT connectome-correlation score as a fast proxy for SR/differentiation potency.

## When NOT to use

- You need the full signalling entropy (SR) estimate itself with its network information, not a fast proxy — CCAT is a correlation-based surrogate for SR
- Other entropy-based measures such as StemID or SLICE would be used instead when transcriptomic entropy without network information is desired
- Your goal is pseudo-time ordering with algorithms like Monocle or MPath rather than potency estimation via SCENT/CCAT

## Inputs

- Chu et al. single-cell RNA-Seq expression matrix (example dataset shipped with the SCENT package)
- Signalling entropy (SR) values for each cell from an upstream SR computation
- PPI signalling connectome used by SCENT/CCAT

## Outputs

- Per-cell CCAT scores (Pearson correlation between transcriptome and connectome)
- Linear regression of CCAT vs SR with coefficient of determination (target R2 ~0.96, range 0.90-1.00)
- Scatter plot of CCAT vs SR coloured by cell type with fitted regression line and R2 annotation, saved as ccat_sr_correlation.png

## How to apply

Load the single-cell expression matrix (the Chu et al. dataset ships as an example with the SCENT package) together with the SR values from the upstream SR reproduction into R. Run the CCAT function from the SCENT package (installed via devtools::install_github("aet21/SCENT")) to compute, for each cell, the Pearson correlation between its transcriptome profile and the PPI connectome. Fit a linear regression of CCAT scores against SR values and extract the coefficient of determination, targeting the reported agreement of R2=0.96 (acceptable range 0.90-1.00). Verify that CCAT scores order pluripotent vs differentiated populations consistently with SR, then produce a scatter plot of CCAT vs SR coloured by cell type with the regression line and R2 annotation.

## Related tools

- **SCENT** (R package providing the CCAT function and the example Chu et al. dataset; installed from GitHub to compute per-cell connectome-transcriptome correlations and SR) — aet21/SCENT
- **CCAT** (Function within SCENT that computes, for every cell, the Pearson correlation between its transcriptome profile and the PPI connectome)
- **R** (Language in which the CCAT computation, linear regression of CCAT vs SR, and plotting are performed)
- **BiocStyle** (Bioconductor style used for the SCENT tutorial documentation workflow)

## Examples

```
devtools::install_github("aet21/SCENT"); ccat <- CCAT(data = exprMatrix, ppi = connectome)
```

## Evaluation signals

- Fitted linear regression of CCAT against SR yields R2 in the range 0.90-1.00, with the reported value being 0.96
- Scatter plot of CCAT vs SR shows clear separation of pluripotent vs differentiated cell populations consistent with the SR-based ordering
- CCAT scores are returned for all 1,018 Chu et al. single cells (one Pearson correlation value per cell)
- CCAT is computed without feature selection, matching the SCENT design principle that potency estimation needs no prior marker knowledge
- Output file ccat_sr_correlation.png exists and contains the regression line and R2 annotation

## Limitations

- CCAT is a proxy for SR; its agreement is empirical (R2~0.96) and the relation between single-cell signalling entropy and transcriptional entropy (as defined by Richard et al.) is non-trivial and not established
- Signalling entropy/CCAT's advantage is accuracy and robustness relative to other entropy measures, driven in part by a subtle positive correlation between transcriptome and connectome; on datasets lacking this coupling the proxy may degrade
- Validation of SI peaks before cell-fate transitions (criticality hallmarks) remains untested pending high-resolution time-course and genome-wide scRNA-Seq data

## Evidence

- [abstract] Signalling entropy provides a more accurate and robust potency estimate than other entropy-based measures, driven in part by a subtle positive correlation between the transcriptome and connectome: [evidence span withheld — anchor status: not_found]
- [methods] A brief tutorial for estimating differentiation potency of single cells using SCENT and CCAT: "A brief tutorial for estimating differentiation potency of single cells using SCENT and CCAT"
- [discussion] signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training: "signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training"
- [readme] devtools::install_github("aet21/SCENT"): "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge"
