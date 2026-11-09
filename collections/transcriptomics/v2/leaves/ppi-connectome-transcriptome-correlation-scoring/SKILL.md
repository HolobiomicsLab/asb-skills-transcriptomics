---
name: ppi-connectome-transcriptome-correlation-scoring
description: Use when you have single-cell expression data (e.g. Chu et al. scRNA-Seq
  matrix) and a PPI connectome, and need to rank cells by differentiation potency
  without running the expensive full signalling entropy computation.
when_to_use_negative:
- When you need full signalling entropy itself or downstream SCENT outputs (e.g. potency
  states, Signalling Index over a cell population) rather than a per-cell correlation
  proxy
- When no PPI connectome/network is available — CCAT is a transcriptome-in-network-context
  measure and cannot be computed from the expression matrix alone
- When ordering cells in pseudo-time where network-independent approaches such as
  Monocle or MPath are explicitly preferred by the analysis design
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SCENT
  role: R package providing the CCAT function that computes the transcriptome-connectome
    Pearson correlation per single cell, and SR values used to validate the proxy
  repo: https://github.com/aet21/SCENT
- name: CCAT
  role: Function within SCENT that computes, for every cell, the Pearson correlation
    between its transcriptome profile and the PPI connectome
  repo: https://github.com/aet21/SCENT
- name: R
  role: Implementation language for running SCENT/CCAT, fitting the CCAT-vs-SR linear
    regression, and rendering the scatter plot
- name: BiocStyle
  role: Bioconductor documentation style used in the SCENT tutorial vignette (require(BiocStyle))
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:62e07dda657b4664f54c865ef212084734dbc55bbc39afca010a0270ed5dc4bc
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# ppi-connectome-transcriptome-correlation-scoring

## Summary

Computes CCAT — the Pearson correlation between a single cell's transcriptome and a protein-protein interaction (PPI) connectome — using the CCAT function of the R SCENT package, as a fast proxy for full signalling entropy (SR) and hence differentiation potency. In the Chu et al. scRNA-Seq dataset, CCAT agrees with SR with R2=0.96.

## When to use

Use when you have single-cell expression data (e.g. Chu et al. scRNA-Seq matrix) and a PPI connectome, and need to rank cells by differentiation potency without running the expensive full signalling entropy computation.

## When NOT to use

- When you need full signalling entropy itself or downstream SCENT outputs (e.g. potency states, Signalling Index over a cell population) rather than a per-cell correlation proxy
- When no PPI connectome/network is available — CCAT is a transcriptome-in-network-context measure and cannot be computed from the expression matrix alone
- When ordering cells in pseudo-time where network-independent approaches such as Monocle or MPath are explicitly preferred by the analysis design

## Inputs

- Single-cell RNA-Seq expression matrix (e.g. Chu et al. dataset shipped as an example with the SCENT package)
- Protein-protein interaction (PPI) connectome / interaction network
- Pre-computed signalling entropy (SR) values per cell, for proxy-validation regression (optional)

## Outputs

- Per-cell CCAT scores (Pearson correlation between each cell's transcriptome and the PPI connectome)
- Linear regression of CCAT vs SR with coefficient of determination (target R2 range 0.90-1.00; reported 0.96)
- Scatter plot of CCAT vs SR coloured by cell type with fitted regression line and R2 annotation (e.g. ccat_sr_correlation.png)

## How to apply

Install the SCENT package (devtools::install_github("aet21/SCENT")) and load the single-cell expression matrix (e.g. the Chu et al. dataset shipped with SCENT) plus a PPI connectome in R. Run the CCAT function from SCENT to compute, for every cell, the Pearson correlation between its transcriptome profile and the PPI connectome; the measure is driven by the subtle positive correlation between transcriptome and connectome that underlies signalling entropy. To validate the proxy, fit a linear regression of CCAT scores against previously computed SR values and extract the coefficient of determination — agreement should fall in R2=0.90-1.00 (reported 0.96). Then confirm CCAT separates known cell types (pluripotent vs differentiated populations) consistently with the SR-based ordering, e.g. by a scatter plot of CCAT vs SR coloured by cell type with the fitted line and R2 annotation. Rationale: CCAT avoids feature selection and prior biological knowledge, matching SCENT's design goal of estimating differentiation potency without marker expression or timepoint assumptions.

## Related tools

- **SCENT** (R package providing the CCAT function that computes the transcriptome-connectome Pearson correlation per single cell, and SR values used to validate the proxy) — https://github.com/aet21/SCENT
- **CCAT** (Function within SCENT that computes, for every cell, the Pearson correlation between its transcriptome profile and the PPI connectome) — https://github.com/aet21/SCENT
- **R** (Implementation language for running SCENT/CCAT, fitting the CCAT-vs-SR linear regression, and rendering the scatter plot)
- **BiocStyle** (Bioconductor documentation style used in the SCENT tutorial vignette (require(BiocStyle)))

## Examples

```
library(devtools); devtools::install_github("aet21/SCENT"); ccat <- sapply(1:ncol(expn.m), function(i) CCAT(expn.m[,i], ppi)) ; summary(lm(ccat ~ sr))$r.squared
```

## Evaluation signals

- Fitted CCAT-SR coefficient of determination falls in the range R2=0.90-1.00 (reported value 0.96)
- CCAT scores separate known cell types (pluripotent vs differentiated populations) consistently with the SR-based ordering
- Scatter plot of CCAT against SR coloured by cell type shows clear cell-type separation and is saved (e.g. ccat_sr_correlation.png)
- CCAT computed for all cells in the dataset (e.g. all 1,018 Chu et al. single cells) with no missing scores
- SCENT installed successfully via devtools::install_github("aet21/SCENT") and the Chu et al. example expression matrix loads without error

## Limitations

- CCAT is a proxy: its agreement with full signalling entropy (R2=0.96) was demonstrated on the Chu et al. dataset and should be re-validated by regression against SR when applied to new datasets
- CCAT depends on the quality and coverage of the PPI connectome; like signalling entropy it is a network-context measure, distinct from transcriptomic entropy measures used in StemID and SLICE which do not use network information
- As with SCENT generally, the method is intended for cases where dropouts or unestablished differentiation hierarchies preclude marker-gene-based potency annotation; it does not replace pseudo-time algorithms such as Monocle or MPath for trajectory inference

## Evidence

- [methods] CCAT proxy agreement: "A brief tutorial for estimating differentiation potency of single cells using SCENT and CCAT"
- [abstract] transcriptome-connectome correlation rationale: [evidence span withheld — anchor status: not_found]
- [readme] no prior knowledge needed: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge such as marker expression"
- [discussion] SCENT vs other tools: "We note that this particular insight is not readily obtainable using other algorithms such as Monocle or MPath1415."
- [readme] install command: "devtools::install_github("aet21/SCENT")"
