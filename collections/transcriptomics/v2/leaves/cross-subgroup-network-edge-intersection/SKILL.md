---
name: cross-subgroup-network-edge-intersection
description: Use when you have a DEGGs subnetworks object built over multiple subgroups
  (e.g. BRCA_Her2 and BRCA_LumA from TCGA-BRCA) and need each subgroup's network restricted
  to interactions not significant in all groups.
when_to_use_negative:
- The analysis goal is the set of interactions shared across all groups (the intersection
  itself) rather than the group-specific remainder after removing them
- Data are multi-omic — use the successor multiDEGGs package (get_multiOmics_diffNetworks)
  instead, since the DEGGs repository is no longer maintained and has been moved and
  significantly expanded into multiDEGGs
- Input is raw counts or a single subgroup only — the common-edge removal requires
  normalised counts and at least two subgroups in the subnetworks_object
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: DEGGs
  role: Provides generate_subnetworks to build per-subgroup differential interaction
    networks (robust linear regression with an interaction term per gene-gene link)
    and extract_sig_deggs to list the significant gene-gene pairs per subgroup that
    feed the intersection step
  repo: https://github.com/elisabettasciacca/DEGGs
- name: R
  role: Implementation language in which the per-subgroup edge-set intersection and
    common-edge removal are performed on the DEGGs object
  repo: https://github.com/elisabettasciacca/DEGGs
- name: multiDEGGs
  role: Successor package (single or multiomic differential network analysis) to which
    the workflow has been moved; relevant when reusing this skill on multi-omic data
  repo: https://github.com/elisabettasciacca/multiDEGGs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad192
    title: 'DEGGs: an R package with shiny app for the identification of differentially
      expressed gene–gene interactions in high-throughput sequencing data'
  source_content_hash: sha256:6afb4d4322032ca342ee8d836d27f1ca818244b69c5bb5292943822cf9f1974e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elisabettasciacca/DEGGs
  tools:
  - DEGGs
  - R
  - multiDEGGs
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cross-subgroup-network-edge-intersection

## Summary

Identifies gene-gene interactions that are significant in every phenotypical subgroup produced by DEGGs' generate_subnetworks and removes them, retaining only group-specific (non-common) links per subgroup. This implements DEGGs' second filtering step, which drops links shared across all user-defined groups.

## When to use

Use when you have a DEGGs subnetworks object built over multiple subgroups (e.g. BRCA_Her2 and BRCA_LumA from TCGA-BRCA) and need each subgroup's network restricted to interactions not significant in all groups.

## When NOT to use

- The analysis goal is the set of interactions shared across all groups (the intersection itself) rather than the group-specific remainder after removing them
- Data are multi-omic — use the successor multiDEGGs package (get_multiOmics_diffNetworks) instead, since the DEGGs repository is no longer maintained and has been moved and significantly expanded into multiDEGGs
- Input is raw counts or a single subgroup only — the common-edge removal requires normalised counts and at least two subgroups in the subnetworks_object

## Inputs

- DEGGs subnetworks object of class 'DEGGs' from generate_subnetworks (e.g. built from TCGA-BRCA BRCA_normCounts and BRCA_metadata with subgroups = c('BRCA_Her2', 'BRCA_LumA'))
- Per-subgroup significant gene-gene interaction tables from extract_sig_deggs

## Outputs

- group_specific_deggs_edges.csv — table of gene pairs significant in at least one but not every subgroup, with gene-pair and subgroup-of-significance columns
- Per-subgroup sets of candidate group-specific (non-common) edges

## How to apply

Start from the subnetworks_object returned by generate_subnetworks (run on normalised counts and metadata with subgroups such as c('BRCA_Her2', 'BRCA_LumA')). For each subgroup, extract its statistically significant gene-gene links using extract_sig_deggs on the subnetworks_object. Intersect the per-subgroup edge sets to identify the pairs that are significant in ALL subgroups — these are the common interactions. Remove the common edges from each subgroup edge set so only candidate group-specific (non-common) interactions remain, then assemble the retained edges into a per-group table with gene-pair and subgroup-of-significance columns and write it to group_specific_deggs_edges.csv. Rationale: DEGGs extracts group-specific interaction networks, so the differential value of a link comes from its absence of significance in at least one other group; links significant everywhere carry no group-specific signal.

## Related tools

- **DEGGs** (Provides generate_subnetworks to build per-subgroup differential interaction networks (robust linear regression with an interaction term per gene-gene link) and extract_sig_deggs to list the significant gene-gene pairs per subgroup that feed the intersection step) — https://github.com/elisabettasciacca/DEGGs
- **R** (Implementation language in which the per-subgroup edge-set intersection and common-edge removal are performed on the DEGGs object) — https://github.com/elisabettasciacca/DEGGs
- **multiDEGGs** (Successor package (single or multiomic differential network analysis) to which the workflow has been moved; relevant when reusing this skill on multi-omic data) — https://github.com/elisabettasciacca/multiDEGGs

## Examples

```
extract_sig_deggs(subnetworks_object)
```

## Evaluation signals

- No retained gene-gene pair in group_specific_deggs_edges.csv is significant in every subgroup — i.e. no output row appears in the full all-subgroups extract_sig_deggs listing for every group
- Every retained pair is significant in at least one subgroup, i.e. it occurs in at least one subgroup's extract_sig_deggs output
- The output table carries the expected schema: gene-pair columns plus a subgroup-of-significance column
- The number of retained edges per subgroup equals the subgroup's extract_sig_deggs edge count minus the number of edges in the all-subgroups intersection
- Retained edge sets derive from the same subnetworks_object (same normalised counts, metadata, and subgroup definitions) with no re-running of generate_subnetworks in between

## Limitations

- The DEGGs repository is no longer maintained and has been moved and significantly expanded into multiDEGGs, so this skill is tied to an archived package
- Differential significance depends on DEGGs' robust linear regression with an interaction term; group-specificity is defined by statistical significance per subgroup, not by effect-size contrasts
- The extracted networks are limited to gene-gene interactions from count and design matrices; multi-omic inputs require the multiDEGGs successor (get_multiOmics_diffNetworks)

## Evidence

- [abstract] The presented R package, provided with a count and design matrix, extract networks of group-specific interactions: "The presented R package, provided with a count and design matrix, extract networks of group-specific interactions"
- [methods] To get a table listing all the significant gene-gene pairs found in all subgroups, you will need to use the extract_sig_deggs: "To get a table listing all the significant gene-gene pairs found in all subgroups, you will need to use the extract_sig_deggs"
- [readme] The DEGGs package test for differential gene-gene correlations across different groups of samples in count data from high-throughput sequencing assays.: "The DEGGs package test for differential gene-gene correlations across different groups of samples in count data from high-throughput 
sequencing assays."
- [readme] This repository is no longer maintained. The package has been moved and significantly expanded.: "This repository is no longer maintained. The package has been moved and significantly expanded."
