---
name: subgroup-specific-network-comparison
description: Use when you have normalised count data plus metadata for two or more
  phenotype subgroups (e.g., BRCA_Her2 vs BRCA_LumA) and want the subgroup-specific
  gene-gene interactions as a feature list for ML models.
when_to_use_negative:
- You need multi-omic differential networks — the DEGGs repository is no longer maintained
  and has been moved into multiDEGGs (get_diffNetworks on assayData lists)
- You want a single global correlation network rather than interactions that are differential
  between specified subgroups
- You want individual gene expression differences rather than gene-gene interaction
  (correlation) differences
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DEGGs
  role: Core R package that generates subgroup-specific gene-gene interaction networks
    from count and design matrices and extracts significant pairs via extract_sig_deggs
  repo: https://github.com/elisabettasciacca/DEGGs
- name: devtools
  role: Used to install DEGGs from GitHub with build_vignettes = TRUE
- name: Shiny
  role: Interactive visualisation of the subnetworks via View_interactive_subnetwork
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad192
    title: 'DEGGs: an R package with shiny app for the identification of differentially
      expressed gene–gene interactions in high-throughput sequencing data'
  source_content_hash: sha256:eac8548e229eaa271129e910333888a884f9500d143ec93ec4aa1582060259b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elisabettasciacca/DEGGs
  tools:
  - DEGGs
  - R
  - Shiny
  - devtools
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# subgroup-specific-network-comparison

## Summary

Generate group-specific gene-gene interaction networks for sample subgroups with DEGGs and extract the significant gene-gene pairs across subgroups for downstream use (e.g., as machine-learning features). Differential significance per gene-gene link is computed via robust linear regression with an interaction term.

## When to use

Use when you have normalised count data plus metadata for two or more phenotype subgroups (e.g., BRCA_Her2 vs BRCA_LumA) and want the subgroup-specific gene-gene interactions as a feature list for ML models.

## When NOT to use

- You need multi-omic differential networks — the DEGGs repository is no longer maintained and has been moved into multiDEGGs (get_diffNetworks on assayData lists)
- You want a single global correlation network rather than interactions that are differential between specified subgroups
- You want individual gene expression differences rather than gene-gene interaction (correlation) differences

## Inputs

- Normalised gene count matrix (e.g., BRCA_normCounts from TCGA-BRCA RNA-seq data)
- Sample metadata table (e.g., BRCA_metadata) with a subgroup variable such as "SUBTYPE"
- Subgroup labels to compare (e.g., c("BRCA_Her2", "BRCA_LumA"))
- Optionally an existing subnetworks object of class Deggs (input to extract_sig_deggs)

## Outputs

- A subnetworks object of class Deggs with subgroup-specific gene networks
- A table of significant gene-gene pairs found in all subgroups (from extract_sig_deggs), exportable as significant_gene_pairs.csv for use as ML features

## How to apply

Install DEGGs from GitHub (devtools::install_github("elisabettasciacca/DEGGs", build_vignettes = TRUE)) and load BRCA_normCounts and BRCA_metadata, first verifying that count matrix columns align with metadata rows using identical(). Call generate_subnetworks with normalised_counts, metadata, subgroup_variable = "SUBTYPE", subgroups = c("BRCA_Her2", "BRCA_LumA"), entrezIDs = TRUE and convert_to_gene_symbols = TRUE to obtain a subnetworks object of class Deggs. For each gene-gene link, DEGGs provides differential statistical significance through robust linear regression with an interaction term, so each subnetwork contains only links that differ between the compared subgroups. Apply extract_sig_deggs to the subnetworks object to get a table listing all significant gene-gene pairs found across all subgroups, then write it out (e.g., significant_gene_pairs.csv) as the ML feature set. Optionally inspect networks interactively with View_interactive_subnetwork and single gene-pair regression fits with print_regressions.

## Related tools

- **DEGGs** (Core R package that generates subgroup-specific gene-gene interaction networks from count and design matrices and extracts significant pairs via extract_sig_deggs) — https://github.com/elisabettasciacca/DEGGs
- **devtools** (Used to install DEGGs from GitHub with build_vignettes = TRUE)
- **Shiny** (Interactive visualisation of the subnetworks via View_interactive_subnetwork)

## Examples

```
subnetworks_object <- generate_subnetworks(normalised_counts = BRCA_normCounts, metadata = BRCA_metadata, subgroup_variable = "SUBTYPE", subgroups = c("BRCA_Her2", "BRCA_LumA"), entrezIDs = TRUE, convert_to_gene_symbols = TRUE, cores = 2); extract_sig_deggs(subnetworks_object)
```

## Evaluation signals

- identical() check passes: count matrix columns match metadata rows before calling generate_subnetworks
- extract_sig_deggs returns a non-empty, well-formed table of gene-gene pairs for the compared subgroups (BRCA_Her2, BRCA_LumA)
- Exported significant_gene_pairs.csv exists and row-for-row matches the table returned by extract_sig_deggs(subnetworks_object)
- Gene identifiers in the extracted pairs are gene symbols (entrezIDs = TRUE with convert_to_gene_symbols = TRUE was applied)

## Limitations

- The DEGGs repository is no longer maintained; the package has been moved and significantly expanded into multiDEGGs
- Shiny-based View_interactive_subnetwork locks the R console until escape is pressed, unless run in the background (e.g., callr::r_bg with pingr to detect startup)
- Statistical significance depends on the package's differential regression testing (robust linear regression with an interaction term); no user-facing threshold like padj_method is exposed in the DEGGs README (multiDEGGs exposes padj_method, e.g., bonferroni)

## Evidence

- [abstract] For each gene–gene link, differential statistical significance is provided through robust linear regression with an interaction term: "For each gene–gene link, differential statistical significance is provided through robust linear regression with an interaction term"
- [methods] extract_sig_deggs lists all significant pairs: "To get a table listing all the significant gene-gene pairs found in all subgroups, you will need to use the extract_sig_deggs"
- [readme] README example with BRCA datasets and generate_subnetworks: "subnetworks_object <- generate_subnetworks(normalised_counts = BRCA_normCounts,
 metadata = BRCA_metadata,
 subgroup_variable = "SUBTYPE","
- [readme] DEGGs deprecated in favour of multiDEGGs: "This repository is no longer maintained. The package has been moved and significantly expanded."
