---
name: network-percolation-node-filtering
description: Use when constructing group-specific subnetworks from DEGGs differential
  interaction networks and needing to prune low-expression nodes via an expression-based
  percolation cut-off optimized for maximum significant links.
when_to_use_negative:
- Nodes carry no average-expression weights, so an expression-based percolation cut-off
  cannot be defined
- The goal is per-gene differential expression testing rather than pruning a differential
  gene-gene interaction network
- The input is already a final filtered edge table (e.g. subgroup_network_edges.csv)
  requiring no further node removal
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DEGGs
  role: Generates the subgroup-specific differential interaction networks and provides
    the percolation-based node filtering step with threshold optimization
  repo: https://github.com/elisabettasciacca/DEGGs
- name: R
  role: Implementation language in which generate_subnetworks() and the percolation
    threshold sweep are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad192
    title: 'DEGGs: an R package with shiny app for the identification of differentially
      expressed gene–gene interactions in high-throughput sequencing data'
  source_content_hash: sha256:4f709ca3ea37a12b244e66e58e45d7ef0ddbf8f735e4bca2822b55c229b6d6a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elisabettasciacca/DEGGs
  tools:
  - DEGGs
  - R
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# network-percolation-node-filtering

## Summary

Filters group-specific DEGGs differential gene-gene interaction subnetworks by removing nodes whose average expression falls below a percolation cut-off, where the cut-off threshold is optimized to maximize the number of statistically significant differential interactions retained.

## When to use

Use when constructing group-specific subnetworks from DEGGs differential interaction networks and needing to prune low-expression nodes via an expression-based percolation cut-off optimized for maximum significant links.

## When NOT to use

- Nodes carry no average-expression weights, so an expression-based percolation cut-off cannot be defined
- The goal is per-gene differential expression testing rather than pruning a differential gene-gene interaction network
- The input is already a final filtered edge table (e.g. subgroup_network_edges.csv) requiring no further node removal

## Inputs

- Normalized RNA-seq count matrix (BRCA_normCounts from the DEGGs package)
- Clinical metadata table (BRCA_metadata) with subgroup variable SUBTYPE
- DEGGs subnetworks object containing significant differential gene-gene interactions per subgroup
- Per-gene mean expression values used as node weights
- Sweep range of candidate percolation cut-off values

## Outputs

- Optimized percolation threshold (cut-off maximizing significant differential interactions)
- Per-subgroup filtered edge tables of surviving gene-gene interactions (e.g. subgroup_network_edges.csv)
- Set of retained significant gene-gene pairs consistent with extract_sig_deggs() output

## How to apply

Run DEGGs::generate_subnetworks() on normalized counts (e.g. BRCA_normCounts) with metadata (BRCA_metadata), subgroup_variable='SUBTYPE', subgroups=c('BRCA_Her2','BRCA_LumA'), entrezIDs=TRUE, convert_to_gene_symbols=TRUE, and cores=2; each gene-gene link's differential significance is computed by robust linear regression with an interaction term. Annotate each subgroup network with mean expression per gene as node weights, then sweep the percolation cut-off and retain the threshold that maximizes the number of statistically significant differential gene-gene interactions. Remove nodes whose average expression falls below the optimized cut-off and export the per-subgroup filtered edge tables (e.g. subgroup_network_edges.csv) listing surviving gene-gene interactions. Verify alignment of count columns to metadata rows with identical(colnames(BRCA_normCounts), rownames(BRCA_metadata)) before filtering.

## Related tools

- **DEGGs** (Generates the subgroup-specific differential interaction networks and provides the percolation-based node filtering step with threshold optimization) — https://github.com/elisabettasciacca/DEGGs
- **R** (Implementation language in which generate_subnetworks() and the percolation threshold sweep are executed)

## Examples

```
subnetworks_object <- DEGGs::generate_subnetworks(normalised_counts = BRCA_normCounts, metadata = BRCA_metadata, subgroup_variable = "SUBTYPE", subgroups = c("BRCA_Her2", "BRCA_LumA"), entrezIDs = TRUE, convert_to_gene_symbols = TRUE, cores = 2)
```

## Evaluation signals

- Exported filtered edge tables match the significant gene-gene pairs recoverable from the DEGGs object via extract_sig_deggs()
- No filtered (removed) node participates in a retained significant interaction
- The selected percolation threshold is the argmax of the count of statistically significant differential gene-gene interactions across the sweep
- identical(colnames(BRCA_normCounts), rownames(BRCA_metadata)) returns TRUE before filtering
- All nodes surviving the cut-off have mean expression at or above the optimized percolation threshold

## Limitations

- The DEGGs GitHub repository is no longer maintained; the package has been moved and significantly expanded into multiDEGGs
- Differential significance is computed for two subgroups via robust linear regression with an interaction term, replaced by one-way ANOVA if more than two groups are selected, so the filtering optimization depends on the chosen regression scheme
- The percolation cut-off is defined on average expression per gene, so genes with uniformly low expression are pruned regardless of interaction significance

## Evidence

- [other] The first filtering step removes nodes with average expression levels below a percolation cut-off: [evidence span withheld — anchor status: not_found]
- [other] the percolation threshold is optimized to maximize the number of statistically significant differential interactions: [evidence span withheld — anchor status: not_found]
- [abstract] For each gene–gene link, differential statistical significance is provided through robust linear regression with an interaction term: "For each gene–gene link, differential statistical significance is provided through robust linear regression with an interaction term"
- [readme] Generate specific gene-gene networks for each subtype: [evidence span withheld — anchor status: external_doc]
- [readme] Get a table listing all the significant gene-gene interactions found in each subtype: [evidence span withheld — anchor status: external_doc]
