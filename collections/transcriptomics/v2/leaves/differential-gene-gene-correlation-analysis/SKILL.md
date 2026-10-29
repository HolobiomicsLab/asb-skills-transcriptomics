---
name: differential-gene-gene-correlation-analysis
description: Use when you have normalized sequencing count data plus sample metadata
  with a categorical subgroup variable (e.g. TCGA-BRCA SUBTYPE = BRCA_Her2 vs BRCA_LumA)
  and need group-specific gene-gene interaction networks.
when_to_use_negative:
- Your goal is detecting non-linear gene-gene associations — DEGGs relies on linear
  regressions with an interaction term and the authors explicitly state it cannot
  detect non-linear relations
- You want multi-omic differential network analysis — the DEGGs repository is no longer
  maintained and has been moved and significantly expanded into multiDEGGs
- You only have a single group/condition with no categorical subgroup variable to
  compare — the regression interaction-term design requires at least two phenotypic
  groups
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: DEGGs
  role: Core R package that fits robust linear regressions with an interaction term
    per gene-gene link and generates group-specific subnetworks via generate_subnetworks,
    extract_sig_deggs, print_regressions
  repo: https://github.com/elisabettasciacca/DEGGs
- name: devtools
  role: Installs DEGGs from GitHub with build_vignettes = TRUE
  repo: https://github.com/elisabettasciacca/DEGGs
- name: Shiny
  role: Interactive exploration of subgroup networks via View_interactive_subnetwork
    (run non-blocking with callr::r_bg and pingr)
  repo: https://github.com/elisabettasciacca/DEGGs
- name: multiDEGGs
  role: Maintained successor package for differential gene-gene correlation analysis
    in single or multi-omic data (get_diffNetworks, View_diffNetworks, plot_regressions)
  repo: https://github.com/elisabettasciacca/multiDEGGs
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  - task_008
  - task_007
  - task_003
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad192
    title: 'DEGGs: an R package with shiny app for the identification of differentially
      expressed gene–gene interactions in high-throughput sequencing data'
  source_content_hash: sha256:4a8883150681a940cec70ea2b3ab66d35ae190c6a8852f8f4927d82e281dec54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elisabettasciacca/DEGGs
  tools:
  - DEGGs
  - R
  - Shiny
  - devtools
  - multiDEGGs
  - org.Hs.eg.db
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# differential-gene-gene-correlation-analysis

## Summary

Detects gene-gene pairs whose correlation differs across phenotypic groups by fitting a robust linear regression with an interaction term per gene pair, then filtering to group-specific significant links via DEGGs. Used to extract subtype-specific differential co-expression networks from high-throughput sequencing count data.

## When to use

Use when you have normalized sequencing count data plus sample metadata with a categorical subgroup variable (e.g. TCGA-BRCA SUBTYPE = BRCA_Her2 vs BRCA_LumA) and need group-specific gene-gene interaction networks.

## When NOT to use

- Your goal is detecting non-linear gene-gene associations — DEGGs relies on linear regressions with an interaction term and the authors explicitly state it cannot detect non-linear relations
- You want multi-omic differential network analysis — the DEGGs repository is no longer maintained and has been moved and significantly expanded into multiDEGGs
- You only have a single group/condition with no categorical subgroup variable to compare — the regression interaction-term design requires at least two phenotypic groups

## Inputs

- Normalized count matrix (e.g. BRCA_normCounts from TCGA-BRCA RNA-seq) with samples as columns
- Clinical/metadata table (e.g. BRCA_metadata) with samples as rows aligned to count columns
- Categorical subgroup variable name (e.g. 'SUBTYPE') and subgroup labels (e.g. c('BRCA_Her2','BRCA_LumA'))
- Optional gene ID settings: entrezIDs = TRUE, convert_to_gene_symbols = TRUE

## Outputs

- DEGGs-class object containing subgroup-specific network tables, total number of statistically significant links, and input data
- Table from extract_sig_deggs listing all significant gene-gene pairs found across all subgroups (e.g. CSV export)
- Per-link differential regression plots from print_regressions
- Interactive shiny network visualization via View_interactive_subnetwork

## How to apply

Install DEGGs from GitHub (devtools::install_github with build_vignettes = TRUE) and load the packaged TCGA-BRCA datasets BRCA_normCounts and BRCA_metadata, verifying alignment with identical(colnames(BRCA_normCounts), rownames(BRCA_metadata)). Call generate_subnetworks with normalised_counts = BRCA_normCounts, metadata = BRCA_metadata, subgroup_variable = 'SUBTYPE', subgroups = c('BRCA_Her2', 'BRCA_LumA'), entrezIDs = TRUE, convert_to_gene_symbols = TRUE, cores = 2; for each gene-gene link, differential statistical significance is assessed through robust linear regression with an interaction term (one-way ANOVA if more than two groups). Internally, node filtering first removes genes with average expression below a percolation cut-off optimized to maximize the number of significant differential interactions, then removes interactions common across all groups so only group-specific links remain. Extract the significant pairs with extract_sig_deggs on the returned DEGGs-class object, and inspect individual links with print_regressions (e.g. NOTCH2-DTX4 or the HER2-specific KDR-FGF17, SHC3-PDGFC links). Optionally explore networks interactively with View_interactive_subnetwork, launching the shiny app in the background via callr::r_bg with pingr to detect when it is up.

## Related tools

- **DEGGs** (Core R package that fits robust linear regressions with an interaction term per gene-gene link and generates group-specific subnetworks via generate_subnetworks, extract_sig_deggs, print_regressions) — https://github.com/elisabettasciacca/DEGGs
- **devtools** (Installs DEGGs from GitHub with build_vignettes = TRUE) — https://github.com/elisabettasciacca/DEGGs
- **Shiny** (Interactive exploration of subgroup networks via View_interactive_subnetwork (run non-blocking with callr::r_bg and pingr)) — https://github.com/elisabettasciacca/DEGGs
- **multiDEGGs** (Maintained successor package for differential gene-gene correlation analysis in single or multi-omic data (get_diffNetworks, View_diffNetworks, plot_regressions)) — https://github.com/elisabettasciacca/multiDEGGs

## Examples

```
subnetworks_object <- generate_subnetworks(normalised_counts = BRCA_normCounts, metadata = BRCA_metadata, subgroup_variable = "SUBTYPE", subgroups = c("BRCA_Her2","BRCA_LumA"), entrezIDs = TRUE, convert_to_gene_symbols = TRUE, cores = 2)
```

## Evaluation signals

- generate_subnetworks returns an object of class DEGGs
- identical(colnames(BRCA_normCounts), rownames(BRCA_metadata)) returns TRUE before analysis
- extract_sig_deggs yields a non-empty table of significant gene-gene pairs for both subgroups
- No retained edge in the group-specific table is significant in every subgroup (common-interaction filter invariant)
- Known HER2-positive-specific links (KDR-FGF17, SHC3-PDGFC) are recoverable from the extracted table

## Limitations

- DEGGs cannot detect non-linear gene-gene relations; it relies on linear regressions with an interaction term (stated limitation of the package)
- The original DEGGs repository is no longer maintained and will not receive further updates; the package moved to multiDEGGs
- The meta-pathway network is restricted to molecular interactions from KEGG, miRTarBase, miRecords and TransmiR, exported via MITHrIL exportgraph
- Shiny apps lock the R console until escape is pressed unless launched in the background

## Evidence

- [abstract] For each gene–gene link, differential statistical significance is provided through robust linear regression with an interaction term: "For each gene–gene link, differential statistical significance is provided through robust linear regression with an interaction term"
- [readme] generate_subnetworks call signature: "subnetworks_object <- generate_subnetworks(normalised_counts = BRCA_normCounts,
                                           metadata = BRCA_metadata,"
- [methods] extract_sig_deggs usage: "To get a table listing all the significant gene-gene pairs found in all subgroups, you will need to use the extract_sig_deggs"
- [methods] shiny blocking behavior: "Shiny typically locks the R console until the user presses escape to close the shiny app. However this behaviour can be bypassed by calling the app in background"
- [readme] deprecation notice: "This repository is no longer maintained. The package has been moved and significantly expanded."
- [results] percolation filtering finding: [evidence span withheld — anchor status: not_found]
