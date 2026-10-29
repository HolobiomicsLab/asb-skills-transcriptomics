---
name: group-specific-interaction-filtering
description: Use when DEGGs generate_subnetworks has produced per-subgroup significant
  edge lists (e.g. BRCA_Her2 vs BRCA_LumA) and you must restrict each network to interactions
  specific to one group rather than links common to all groups.
when_to_use_negative:
- You need interactions significant in all subgroups (common links) rather than group-specific
  ones — this skill removes exactly those edges
- You are starting from raw count data rather than a fitted DEGGs subnetworks_object;
  run generate_subnetworks first
- Your goal is multi-omic differential networks; the maintained multiDEGGs package
  (get_diffNetworks) is the successor tool for that scope
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: DEGGs
  role: Provides generate_subnetworks to build per-subgroup networks and extract_sig_deggs
    to list significant gene-gene pairs; this skill implements the common-interaction
    removal step on those outputs
  repo: https://github.com/elisabettasciacca/DEGGs
- name: R
  role: Language in which the intersection/subtraction of per-subgroup edge sets is
    implemented
- name: multiDEGGs
  role: Maintained successor package; users needing group-specific interaction filtering
    should prefer it over the archived DEGGs
  repo: https://github.com/elisabettasciacca/multiDEGGs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad192
    title: 'DEGGs: an R package with shiny app for the identification of differentially
      expressed gene–gene interactions in high-throughput sequencing data'
  source_content_hash: sha256:1cee76f65dc700189b9173b8b28a5df79deb5c4b1f82a3e8efd3e012d5e56f1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elisabettasciacca/DEGGs
  tools:
  - DEGGs
  - R
  - multiDEGGs
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# group-specific-interaction-filtering

## Summary

Post-processing step for DEGGs differential-network results that removes gene-gene interactions significant in every subgroup, leaving only candidate group-specific (differentially expressed) links in each subgroup-annotated network.

## When to use

Use when DEGGs generate_subnetworks has produced per-subgroup significant edge lists (e.g. BRCA_Her2 vs BRCA_LumA) and you must restrict each network to interactions specific to one group rather than links common to all groups.

## When NOT to use

- You need interactions significant in all subgroups (common links) rather than group-specific ones — this skill removes exactly those edges
- You are starting from raw count data rather than a fitted DEGGs subnetworks_object; run generate_subnetworks first
- Your goal is multi-omic differential networks; the maintained multiDEGGs package (get_diffNetworks) is the successor tool for that scope

## Inputs

- DEGGs subnetworks object from generate_subnetworks(normalised_counts = BRCA_normCounts, metadata = BRCA_metadata, subgroup_variable = 'SUBTYPE', subgroups = c('BRCA_Her2','BRCA_LumA'))
- Per-subgroup significant edge tables from extract_sig_deggs(subnetworks_object)

## Outputs

- Per-subgroup table of group-specific (non-common) gene-gene edges with gene-pair and subgroup columns
- Filtered edge table written to group_specific_deggs_edges.csv

## How to apply

Run extract_sig_deggs on the DEGGs subnetworks_object to obtain per-subgroup tables of statistically significant gene-gene pairs. Intersect the per-subgroup edge sets to identify pairs significant in ALL subgroups; these common interactions are exactly what the second filtering step of the DEGGs pipeline removes. Subtract the common edge set from each subgroup's edge list so only non-common, candidate group-specific interactions remain. Assemble the retained pairs into a table with gene-pair and subgroup-of-significance columns (e.g. group_specific_deggs_edges.csv) and validate that no retained pair appears significant in every subgroup of the full extract_sig_deggs listing.

## Related tools

- **DEGGs** (Provides generate_subnetworks to build per-subgroup networks and extract_sig_deggs to list significant gene-gene pairs; this skill implements the common-interaction removal step on those outputs) — https://github.com/elisabettasciacca/DEGGs
- **R** (Language in which the intersection/subtraction of per-subgroup edge sets is implemented)
- **multiDEGGs** (Maintained successor package; users needing group-specific interaction filtering should prefer it over the archived DEGGs) — https://github.com/elisabettasciacca/multiDEGGs

## Examples

```
subnetworks_object <- generate_subnetworks(normalised_counts = BRCA_normCounts, metadata = BRCA_metadata, subgroup_variable = "SUBTYPE", subgroups = c("BRCA_Her2", "BRCA_LumA"), entrezIDs = TRUE, convert_to_gene_symbols = TRUE, cores = 2); sig <- extract_sig_deggs(subnetworks_object)
```

## Evaluation signals

- Every retained gene-gene pair is significant in at least one subgroup but not in every subgroup of the extract_sig_deggs output
- No row in the filtered table occurs in the intersection of all per-subgroup edge sets
- Output table contains the gene-pair and subgroup-of-significance columns
- Filtering can be reproduced deterministically: the common-interaction set equals the intersection of per-subgroup significant edges

## Limitations

- The DEGGs repository is no longer maintained and has been moved and significantly expanded into multiDEGGs
- Differential significance derives from robust linear regression with an interaction term; results depend on the subgroup definitions supplied to generate_subnetworks
- Only interactions common across ALL user-defined groups are removed; pairs shared by some but not all groups are retained as group-specific

## Evidence

- [abstract] The presented R package, provided with a count and design matrix, extract networks of group-specific interactions: "The presented R package, provided with a count and design matrix, extract networks of group-specific interactions"
- [methods] To get a table listing all the significant gene-gene pairs found in all subgroups, you will need to use the extract_sig_deggs: "To get a table listing all the significant gene-gene pairs found in all subgroups, you will need to use the extract_sig_deggs"
- [readme] Get a table listing all the significant gene-gene interactions found in each subtype extract_sig_deggs(subnetworks_object): "Get a table listing all the significant gene-gene interactions found in each subtype  
`extract_sig_deggs(subnetworks_object)`"
- [abstract] For each gene–gene link, differential statistical significance is provided through robust linear regression with an interaction term: "For each gene–gene link, differential statistical significance is provided through robust linear regression with an interaction term"
- [intro] This repository is no longer maintained. The package has been moved and significantly expanded.: "This repository is no longer maintained. The package has been moved and significantly expanded."
