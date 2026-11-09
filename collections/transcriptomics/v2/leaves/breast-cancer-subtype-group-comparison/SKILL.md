---
name: breast-cancer-subtype-group-comparison
description: Use when you have a normalised count matrix plus metadata for TCGA-BRCA
  (or similar) samples with a SUBTYPE column, and the goal is to find gene–gene interactions
  whose correlation differs between two phenotypic subgroups.
when_to_use_negative:
- 'You need to detect non-linear gene–gene associations: DEGGs relies on linear regressions
  with an interaction term and the authors explicitly state it cannot detect non-linear
  gene–gene relations'
- The count matrix column names and metadata row names do not align (identical() check
  fails), since subgroup assignment requires matched samples
- You are working with multi-omic data (RNAseq + Proteomics/Olink) — the maintained
  successor multiDEGGs with get_diffNetworks is the appropriate choice
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: DEGGs
  role: R package performing subgroup-specific differential gene-gene interaction
    detection via robust linear regression with an interaction term; provides generate_subnetworks,
    extract_sig_deggs, View_interactive_subnetwork and print_regressions
  repo: https://github.com/elisabettasciacca/DEGGs
- name: devtools
  role: Installs the DEGGs package from GitHub with build_vignettes = TRUE
- name: callr
  role: Runs the shiny View_interactive_subnetwork app in the background so the R
    console is not locked
- name: pingr
  role: Used to know when the background-launched shiny app is up and running
- name: multiDEGGs
  role: Successor package (on CRAN/GitHub) for the expanded, maintained version of
    this workflow including multi-omic data
  repo: https://github.com/elisabettasciacca/multiDEGGs
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad192
    title: 'DEGGs: an R package with shiny app for the identification of differentially
      expressed gene–gene interactions in high-throughput sequencing data'
  source_content_hash: sha256:6872cefd08eca4575e5d8ebecd441399d2d0c96d2ebc1e635ddbfdcb4447cf90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elisabettasciacca/DEGGs
  tools:
  - DEGGs
  - R
  - callr
  - devtools
  - multiDEGGs
  - pingr
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# breast-cancer-subtype-group-comparison

## Summary

Identifies subgroup-specific gene–gene interaction networks from TCGA-BRCA RNA-seq counts by running DEGGs' robust linear regression with an interaction term across two breast cancer subgroups (e.g. BRCA_Her2 vs BRCA_LumA) and extracting the significantly differential gene–gene pairs.

## When to use

Use when you have a normalised count matrix plus metadata for TCGA-BRCA (or similar) samples with a SUBTYPE column, and the goal is to find gene–gene interactions whose correlation differs between two phenotypic subgroups.

## When NOT to use

- You need to detect non-linear gene–gene associations: DEGGs relies on linear regressions with an interaction term and the authors explicitly state it cannot detect non-linear gene–gene relations
- The count matrix column names and metadata row names do not align (identical() check fails), since subgroup assignment requires matched samples
- You are working with multi-omic data (RNAseq + Proteomics/Olink) — the maintained successor multiDEGGs with get_diffNetworks is the appropriate choice

## Inputs

- BRCA_normCounts: normalised RNA-seq count matrix (samples as columns) from TCGA breast cancer data
- BRCA_metadata: clinical metadata data frame with a SUBTYPE column whose rownames match the count matrix column names
- Entrez IDs for genes (entrezIDs = TRUE), with optional conversion to gene symbols

## Outputs

- A DEGGs subnetworks object containing subgroup-specific gene-gene interaction networks for each specified subgroup
- A table of significant gene-gene pairs per subgroup from extract_sig_deggs (exportable as e.g. sig_deggs_pairs.csv)
- Interactive shiny network views and per-pair differential regression plots

## How to apply

Load the packaged BRCA_normCounts and BRCA_metadata with data() and confirm sample alignment via identical(colnames(BRCA_normCounts), rownames(BRCA_metadata)) returning TRUE. Call DEGGs::generate_subnetworks on the normalised counts with subgroup_variable = 'SUBTYPE', subgroups = c('BRCA_Her2', 'BRCA_LumA'), entrezIDs = TRUE, convert_to_gene_symbols = TRUE and cores = 2, which fits a robust linear regression with an interaction term for every gene–gene link per subgroup. Optionally inspect results interactively with View_interactive_subnetwork (launchable non-blockingly via callr::r_bg) or inspect a single pair with print_regressions (e.g. NOTCH2–DTX4). Finally, extract the significant differentially expressed gene–gene (DEGGs) pairs with DEGGs::extract_sig_deggs and export the returned table (e.g. to sig_deggs_pairs.csv) for downstream comparison of subgroup-specific links.

## Related tools

- **DEGGs** (R package performing subgroup-specific differential gene-gene interaction detection via robust linear regression with an interaction term; provides generate_subnetworks, extract_sig_deggs, View_interactive_subnetwork and print_regressions) — https://github.com/elisabettasciacca/DEGGs
- **devtools** (Installs the DEGGs package from GitHub with build_vignettes = TRUE)
- **callr** (Runs the shiny View_interactive_subnetwork app in the background so the R console is not locked)
- **pingr** (Used to know when the background-launched shiny app is up and running)
- **multiDEGGs** (Successor package (on CRAN/GitHub) for the expanded, maintained version of this workflow including multi-omic data) — https://github.com/elisabettasciacca/multiDEGGs

## Examples

```
subnetworks_object <- generate_subnetworks(normalised_counts = BRCA_normCounts, metadata = BRCA_metadata, subgroup_variable = "SUBTYPE", subgroups = c("BRCA_Her2", "BRCA_LumA"), entrezIDs = TRUE, convert_to_gene_symbols = TRUE, cores = 2); extract_sig_deggs(subnetworks_object)
```

## Evaluation signals

- identical(colnames(BRCA_normCounts), rownames(BRCA_metadata)) returns TRUE before running generate_subnetworks
- generate_subnetworks completes without error and returns an object of class DEGGs for both subgroups (BRCA_Her2 and BRCA_LumA)
- extract_sig_deggs returns a non-empty table listing significant gene-gene pairs for both subgroups
- print_regressions for a known significant pair (e.g. gene_A = 'NOTCH2', gene_B = 'DTX4') renders two subgroup-specific regression fits
- The exported CSV contains one row per significant pair with subgroup-level significance for both subgroups

## Limitations

- DEGGs cannot detect non-linear gene–gene relations; the authors explicitly state this as a limitation since it relies on linear regressions with an interaction term
- The DEGGs GitHub repository is no longer maintained; the package has been moved and significantly expanded into multiDEGGs
- The shiny View_interactive_subnetwork app typically locks the R console until escape is pressed unless launched in the background via callr::r_bg

## Evidence

- [abstract] Robust linear regression with interaction term for differential link significance: "For each gene–gene link, differential statistical significance is provided through robust linear regression with an interaction term"
- [methods] Installation from GitHub via devtools: "devtools::install_github("elisabettasciacca/DEGGs", build_vignettes = TRUE)"
- [methods] Shiny app locks the console; workaround exists: "Shiny typically locks the R console until the user presses escape to close the shiny app. However this behaviour can be bypassed by calling the app in background"
- [readme] README example calling generate_subnetworks with BRCA subgroups: "subnetworks_object <- generate_subnetworks(normalised_counts = BRCA_normCounts, metadata = BRCA_metadata, subgroup_variable = "SUBTYPE", subgroups = c("BRCA_Her2", "BRCA_LumA"), entrezIDs = TRUE,"
- [readme] Repository deprecation notice pointing to multiDEGGs: "This repository is no longer maintained. The package has been moved and significantly expanded."
