---
name: false-discovery-rate-thresholding
description: Use when a background-subtracted microarray expression matrix has two
  known response classes (e.g. wild-type vs. mutant samples) and you need a statistically
  ranked list of differentially expressed genes under a permissive FDR cutoff such
  as 0.4.
when_to_use_negative:
- The input is raw, unnormalized array data — normalization must be done first via
  PIMENTo's Preprocess Data command before SAM significance testing
- The samples lack a defined response class structure (no phenotype labels such as
  WT vs. villin-gp130Act) for the SAM response vector
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: PIMENTo
  role: Provides the Sig_GenesSAM command that runs SAM testing and exports significant
    genes with distance-matrix and PCA plots
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: samr
  role: R package whose Significance Analysis of Microarrays code performs the underlying
    significance testing inside PIMENTo
- name: R
  role: Programming language in which PIMENTo and the samr-based SAM testing are executed
- name: RStudio
  role: IDE used to run PIMENTo commands with autocomplete capability
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:0adfabc1067bdc54a79953459eb74a38b453ae44ee42824a16e0a2ac5d55ca8f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PIMENTo
  - R
  - RStudio
  - samr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# false-discovery-rate-thresholding

## Summary

Apply false-discovery-rate (FDR) thresholding via Significance Analysis of Microarrays (SAM) to identify genes whose expression differs significantly between response classes, using PIMENTo's Sig_GenesSAM command which wraps the R 'samr' package.

## When to use

Use when a background-subtracted microarray expression matrix has two known response classes (e.g. wild-type vs. mutant samples) and you need a statistically ranked list of differentially expressed genes under a permissive FDR cutoff such as 0.4.

## When NOT to use

- The input is raw, unnormalized array data — normalization must be done first via PIMENTo's Preprocess Data command before SAM significance testing
- The samples lack a defined response class structure (no phenotype labels such as WT vs. villin-gp130Act) for the SAM response vector

## Inputs

- Background-subtracted expression matrix from upstream PIMENTo preprocessing of the E-MTAB-2400 mouse microarray dataset
- Response class vector (e.g. c(1,1,1,2,2,2)) encoding WT vs. villin-gp130Act sample groups
- FDR cutoff parameter (fdr.cutoff, e.g. 0.4)

## Outputs

- Tabular file of significant genes passing the FDR cutoff
- Euclidean distance similarity plot of samples based on significant genes (Fig. 5)
- PCA plot of samples based on significant genes (Fig. 6)

## How to apply

Load the background-subtracted expression matrix produced by PIMENTo's upstream Preprocess Data/normalization step from the E-MTAB-2400 mouse microarray dataset (WT vs. villin-gp130Act small intestinal crypts). Define the response class vector, e.g. c(1,1,1,2,2,2) encoding three wild-type (class 1) and three villin-gp130Act (class 2) samples. Run PIMENTo's SigGenesSAM command, which performs Significance Analysis of Microarrays testing via the R package 'samr', setting fdr.cutoff to 0.4 to select genes whose expression differs significantly between classes. Export the significant genes to a tabular output file, then generate the Euclidean distance similarity plot and PCA plot of samples based on the significant genes to confirm class separation.

## Related tools

- **PIMENTo** (Provides the Sig_GenesSAM command that runs SAM testing and exports significant genes with distance-matrix and PCA plots) — https://www.github.com/MUSC-CGM/PIMENTo
- **samr** (R package whose Significance Analysis of Microarrays code performs the underlying significance testing inside PIMENTo)
- **R** (Programming language in which PIMENTo and the samr-based SAM testing are executed)
- **RStudio** (IDE used to run PIMENTo commands with autocomplete capability)

## Examples

```
library(PIMENTo); SigGenesSAM(data=bg_subtracted_matrix, response=c(1,1,1,2,2,2), fdr.cutoff=0.4)
```

## Evaluation signals

- The significant-genes tabular output file is populated at FDR 0.4
- The Euclidean distance matrix (Fig. 5) shows clear differences between samples of different response classes and similarity within classes
- The PCA plot (Fig. 6) visually separates wild-type and villin-gp130Act response classes
- Class separation in the plots is consistent with the c(1,1,1,2,2,2) response vector structure

## Limitations

- No version numbers are stated for samr, limma, affy, R, or Bioconductor dependencies
- The citation for the samr package is an incomplete placeholder '[ref]' in the article
- A permissive fdr.cutoff of 0.4 is used, which admits a relatively large expected proportion of false discoveries
- The R Shiny user interface was not yet available; only command-line R usage via RStudio was supported

## Evidence

- [other] Run the PIMENTo SigGenesSAM command, which performs Significance Analysis of Microarrays testing via the R package 'samr', with fdr.cutoff set to 0.4: [evidence span withheld — anchor status: not_found]
- [discussion] Significance testing is carried out using the code for Significance Analysis of Microarrays from the R package: [evidence span withheld — anchor status: not_found]
- [other] the distance matrix generated by this SAM run (Fig. 5) shows a clear difference between samples of different response classes: [evidence span withheld — anchor status: not_found]
- [readme] PIMENTo is an R package to normalize, analyze, and visualize microarray data: "PIMENTo is an R package to normalize, analyze, and visualize microarray data"
- [readme] devtools::install_github("MUSC-CGM/PIMENTo"): "devtools::install_github("MUSC-CGM/PIMENTo")"
