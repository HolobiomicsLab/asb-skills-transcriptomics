---
name: gene-gene-interaction-extraction
description: Use when you have a DEGGs subnetworks object of class Deggs (e.g. from
  generate_subnetworks on TCGA-BRCA counts) and need the significant gene-gene pairs
  listed per subgroup as candidate features.
when_to_use_negative:
- Input is already a feature table or a plain edge list — extract_sig_deggs requires
  a Deggs-class subnetworks object, not a matrix or CSV.
- You need multi-omic (e.g. RNAseq + Proteomics + Olink) differential networks — the
  DEGGs repository is no longer maintained and this capability moved to multiDEGGs
  via get_diffNetworks.
- You want all pairwise correlations within one group rather than differential interactions
  across phenotypical groups — DEGGs specifically tests differential gene-gene correlations
  across groups.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: DEGGs
  role: R package whose generate_subnetworks builds the subgroup-specific differential
    networks and whose extract_sig_deggs returns the table of significant gene-gene
    pairs
  repo: https://github.com/elisabettasciacca/DEGGs
- name: devtools
  role: used to install DEGGs from GitHub with vignettes (devtools::install_github('elisabettasciacca/DEGGs',
    build_vignettes = TRUE))
- name: multiDEGGs
  role: expanded successor package for the same differential gene-gene interaction
    analysis (CRAN install.packages('multiDEGGs'))
  repo: https://github.com/elisabettasciacca/multiDEGGs
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad192
    title: 'DEGGs: an R package with shiny app for the identification of differentially
      expressed gene–gene interactions in high-throughput sequencing data'
  source_content_hash: sha256:ce7ea4cc42fe60a5fca8499e10d8ccb8889e85a4b6bf2410d499fd3a2bed504d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elisabettasciacca/DEGGs
  tools:
  - DEGGs
  - R
  - devtools
  - multiDEGGs
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gene-gene-interaction-extraction

## Summary

Extracts the table of significant gene-gene pairs from DEGGs subgroup-specific networks using extract_sig_deggs, optionally removing interactions common to all subgroups, so the retained group-specific gene pairs can be reused as machine-learning features.

## When to use

Use when you have a DEGGs subnetworks object of class Deggs (e.g. from generate_subnetworks on TCGA-BRCA counts) and need the significant gene-gene pairs listed per subgroup as candidate features.

## When NOT to use

- Input is already a feature table or a plain edge list — extract_sig_deggs requires a Deggs-class subnetworks object, not a matrix or CSV.
- You need multi-omic (e.g. RNAseq + Proteomics + Olink) differential networks — the DEGGs repository is no longer maintained and this capability moved to multiDEGGs via get_diffNetworks.
- You want all pairwise correlations within one group rather than differential interactions across phenotypical groups — DEGGs specifically tests differential gene-gene correlations across groups.

## Inputs

- subnetworks object of class Deggs produced by generate_subnetworks
- normalised RNA-seq count matrix (e.g. BRCA_normCounts from the DEGGs vignette)
- sample metadata with subgroup labels (e.g. BRCA_metadata with SUBTYPE variable BRCA_Her2 / BRCA_LumA)

## Outputs

- table of significant gene-gene pairs with gene-pair fields and subgroup of significance (e.g. significant_gene_pairs.csv)
- filtered per-group edge table retaining only group-specific (non-common) interactions (e.g. group_specific_deggs_edges.csv)

## How to apply

Generate the subgroup-specific networks with generate_subnetworks (e.g. subgroups = c('BRCA_Her2', 'BRCA_LumA'), subgroup_variable = 'SUBTYPE', entrezIDs = TRUE, convert_to_gene_symbols = TRUE) on the normalised count matrix with metadata rows aligned to count columns (verify with identical()). Then call extract_sig_deggs on the subnetworks_object to obtain a table of all significant gene-gene pairs found across all subgroups, as computed by DEGGs' robust linear regression with an interaction term. To restrict to group-specific interactions, intersect the per-subgroup edge sets and remove pairs significant in every subgroup, keeping only pairs significant in at least one but not all groups. Write the resulting gene-pair table (e.g. significant_gene_pairs.csv or group_specific_deggs_edges.csv) for downstream use as features in machine-learning models.

## Related tools

- **DEGGs** (R package whose generate_subnetworks builds the subgroup-specific differential networks and whose extract_sig_deggs returns the table of significant gene-gene pairs) — https://github.com/elisabettasciacca/DEGGs
- **devtools** (used to install DEGGs from GitHub with vignettes (devtools::install_github('elisabettasciacca/DEGGs', build_vignettes = TRUE)))
- **multiDEGGs** (expanded successor package for the same differential gene-gene interaction analysis (CRAN install.packages('multiDEGGs'))) — https://github.com/elisabettasciacca/multiDEGGs

## Examples

```
subnetworks_object <- generate_subnetworks(normalised_counts = BRCA_normCounts, metadata = BRCA_metadata, subgroup_variable = "SUBTYPE", subgroups = c("BRCA_Her2", "BRCA_LumA"), entrezIDs = TRUE, convert_to_gene_symbols = TRUE, cores = 2); sig_pairs <- extract_sig_deggs(subnetworks_object)
```

## Evaluation signals

- Output CSV exists and is a well-formed table with gene-pair fields for the compared subgroups (e.g. BRCA_Her2, BRCA_LumA).
- Rows of the written CSV match the rows returned by extract_sig_deggs(subnetworks_object).
- For the group-specific variant: no retained row occurs as significant in every subgroup when cross-checked against the full all-subgroups extract_sig_deggs listing; every retained pair is significant in at least one but not every subgroup.
- Count-matrix columns and metadata rows were aligned (identical() check) before generate_subnetworks was run.

## Limitations

- The DEGGs repository is no longer maintained; the package has been moved and significantly expanded into multiDEGGs, so new work may need get_diffNetworks/get_multiOmics_diffNetworks with a sig_threshold (e.g. 0.05).
- extract_sig_deggs operates on the subnetworks object, so the subgroup set and gene-symbol conversion (entrezIDs = TRUE, convert_to_gene_symbols = TRUE) must be fixed upstream in generate_subnetworks.
- Significant pairs listed across all subgroups include interactions common to all groups; group-specificity requires an explicit removal of edges shared across all user-defined groups.

## Evidence

- [methods] To get a table listing all the significant gene-gene pairs found in all subgroups, you will need to use the extract_sig_deggs: "To get a table listing all the significant gene-gene pairs found in all subgroups, you will need to use the extract_sig_deggs"
- [other] the second filtering step removes gene-gene interactions that are common across all user-defined groups: [evidence span withheld — anchor status: not_found]
- [readme] Get a table listing all the significant gene-gene interactions found in each subtype extract_sig_deggs(subnetworks_object): "Get a table listing all the significant gene-gene interactions found in each subtype  
`extract_sig_deggs(subnetworks_object)`"
- [readme] This repository is no longer maintained. The package has been moved and significantly expanded.: "This repository is no longer maintained. The package has been moved and significantly expanded."
- [abstract] For each gene–gene link, differential statistical significance is provided through robust linear regression with an interaction term: "For each gene–gene link, differential statistical significance is provided through robust linear regression with an interaction term"
