---
name: subgroup-specific-edge-table-extraction
description: Use when normalized RNA-seq counts and clinical metadata grouped by a
  subgroup variable (e.g. TCGA-BRCA SUBTYPE like BRCA_Her2 vs BRCA_LumA) must be reduced
  to group-specific gene-gene interaction edge lists after expression-based node filtering.
when_to_use_negative:
- The data are multi-omic (e.g. RNAseq + Proteomics + Olink) — the DEGGs repository
  is no longer maintained and this use case is covered by the expanded multiDEGGs
  package via get_diffNetworks()
- The goal is interactive visualization or single-pair regression inspection rather
  than producing filtered edge tables — use View_interactive_subnetwork() or print_regressions()
  instead
- Sample column names of the count matrix do not match the metadata row names (identical(colnames(BRCA_normCounts),
  rownames(BRCA_metadata)) is FALSE) — alignment must be fixed before subnetwork generation
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DEGGs
  role: R package whose generate_subnetworks() computes differential gene-gene significance
    (robust linear regression with an interaction term) and whose percolation-threshold
    filtering plus extract_sig_deggs() yield the subgroup edge tables
  repo: https://github.com/elisabettasciacca/DEGGs
- name: R
  role: Execution environment for the DEGGs workflow, including data loading, threshold
    sweeping, and CSV export
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad192
    title: 'DEGGs: an R package with shiny app for the identification of differentially
      expressed gene–gene interactions in high-throughput sequencing data'
  source_content_hash: sha256:2e5552c19056199ccaa90cfa48688e6d813cca633369e6ad0dd24349455c1278
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elisabettasciacca/DEGGs
  tools:
  - DEGGs
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# subgroup-specific-edge-table-extraction

## Summary

Filters DEGGs subgroup networks by sweeping a percolation cut-off on per-gene mean expression, keeping the threshold that maximizes the number of statistically significant differential gene-gene interactions, and exports the surviving edges as per-subgroup edge tables.

## When to use

Use when normalized RNA-seq counts and clinical metadata grouped by a subgroup variable (e.g. TCGA-BRCA SUBTYPE like BRCA_Her2 vs BRCA_LumA) must be reduced to group-specific gene-gene interaction edge lists after expression-based node filtering.

## When NOT to use

- The data are multi-omic (e.g. RNAseq + Proteomics + Olink) — the DEGGs repository is no longer maintained and this use case is covered by the expanded multiDEGGs package via get_diffNetworks()
- The goal is interactive visualization or single-pair regression inspection rather than producing filtered edge tables — use View_interactive_subnetwork() or print_regressions() instead
- Sample column names of the count matrix do not match the metadata row names (identical(colnames(BRCA_normCounts), rownames(BRCA_metadata)) is FALSE) — alignment must be fixed before subnetwork generation

## Inputs

- Normalized RNA-seq count matrix (BRCA_normCounts) from TCGA-BRCA with genes on columns/samples on rows as shipped by the DEGGs package
- Clinical metadata table (BRCA_metadata) keyed to sample names and containing a SUBTYPE column (e.g. BRCA_Her2, BRCA_LumA)
- DEGGs subnetworks object returned by generate_subnetworks() containing per-subgroup networks with significance per gene-gene link

## Outputs

- Per-subgroup filtered edge tables listing surviving gene-gene interactions (exported to subgroup_network_edges.csv)
- Optimized percolation cut-off value on mean gene expression for each subgroup network
- Cross-check table of all significant gene-gene pairs from extract_sig_deggs(subnetworks_object)

## How to apply

Load the BRCA_normCounts normalized count matrix and BRCA_metadata table shipped with DEGGs and verify alignment with identical(colnames(BRCA_normCounts), rownames(BRCA_metadata)). Run DEGGs::generate_subnetworks() with subgroup_variable='SUBTYPE', subgroups=c('BRCA_Her2','BRCA_LumA'), entrezIDs=TRUE, convert_to_gene_symbols=TRUE, and cores=2, so each gene-gene link's differential significance is computed by robust linear regression with an interaction term. Annotate each subgroup network with per-gene mean expression as node weights, then sweep the percolation cut-off and retain the threshold that maximizes the number of statistically significant differential gene-gene interactions before removing nodes below it. Export the per-subgroup filtered edge tables (subgroup_network_edges.csv) and validate them against the significant pairs recoverable with extract_sig_deggs(), confirming no filtered node participates in a retained interaction.

## Related tools

- **DEGGs** (R package whose generate_subnetworks() computes differential gene-gene significance (robust linear regression with an interaction term) and whose percolation-threshold filtering plus extract_sig_deggs() yield the subgroup edge tables) — https://github.com/elisabettasciacca/DEGGs
- **R** (Execution environment for the DEGGs workflow, including data loading, threshold sweeping, and CSV export)

## Examples

```
subnetworks_object <- generate_subnetworks(normalised_counts=BRCA_normCounts, metadata=BRCA_metadata, subgroup_variable="SUBTYPE", subgroups=c("BRCA_Her2","BRCA_LumA"), cores=2)
```

## Evaluation signals

- identical(colnames(BRCA_normCounts), rownames(BRCA_metadata)) returns TRUE before subnetwork generation
- The chosen percolation cut-off yields the maximum count of statistically significant differential gene-gene interactions across the swept threshold range
- Exported per-subgroup edge tables match the significant gene-gene pairs recovered by extract_sig_deggs(subnetworks_object)
- No node removed by the percolation filter participates in any retained significant interaction
- Both selected subgroups (BRCA_Her2 and BRCA_LumA) produce non-empty filtered edge tables when significant differential interactions exist

## Limitations

- The elisabettasciacca/DEGGs repository is no longer maintained; the package has been moved and significantly expanded into multiDEGGs
- Differential significance for each gene-gene link is computed via robust linear regression with an interaction term for two groups; the workflow notes one-way ANOVA replaces it if more than two groups are selected
- Node filtering is driven by a percolation cut-off on average expression, so lowly expressed genes are removed even if part of the raw differential network

## Evidence

- [abstract] For each gene–gene link, differential statistical significance is provided through robust linear regression with an interaction term: "For each gene–gene link, differential statistical significance is provided through robust linear regression with an interaction term"
- [methods] extract_sig_deggs lists significant pairs: "To get a table listing all the significant gene-gene pairs found in all subgroups, you will need to use the extract_sig_deggs"
- [readme] generate_subnetworks creates subtype-specific networks: [evidence span withheld — anchor status: external_doc]
