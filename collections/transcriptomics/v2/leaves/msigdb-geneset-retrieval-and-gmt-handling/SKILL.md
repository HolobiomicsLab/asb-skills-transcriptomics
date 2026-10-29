---
name: msigdb-geneset-retrieval-and-gmt-handling
description: Use when a pathway/module activity analysis (e.g., rROMA) needs a curated
  gene set collection and the user names a database like MSigDB HALLMARK or supplies
  a GMT file as the module list.
when_to_use_negative:
- The gene sets of interest already exist as an rRoma module object or the bundled
  ACSN_Global.v1.1 collection, so no MSigDB retrieval is needed
- The expression matrix is raw counts — normalize first (e.g., cpm from edgeR) because
  rRoma requires normalized data
- The analysis goal is differential expression per gene rather than coordinated gene
  set/module activity
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0203
tools:
- name: rRoma
  role: Provides SelectFromMSIGdb (MSigDB v6.2 lookup) and ReadGMTFile to build the
    module list passed to rRoma.R
  repo: https://github.com/sysbio-curie/rROMA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:414e420ae0a969c4c67138de6e2d79f9cd9420263ac818ad8a82b5437130926d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/rROMA
  tools:
  - R
  - edgeR
  - irlba
  - rROMA
  - rRoma
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# msigdb-geneset-retrieval-and-gmt-handling

## Summary

Retrieve pathway gene sets (e.g., all 50 MSigDB HALLMARK sets from MSigDB v6.2) for module-activity analysis using rRoma's SelectFromMSIGdb, or load custom modules from a GMT file with ReadGMTFile, then optionally rename sets (e.g., strip 'HALLMARK_') for cleaner plotting.

## When to use

Use when a pathway/module activity analysis (e.g., rROMA) needs a curated gene set collection and the user names a database like MSigDB HALLMARK or supplies a GMT file as the module list.

## When NOT to use

- The gene sets of interest already exist as an rRoma module object or the bundled ACSN_Global.v1.1 collection, so no MSigDB retrieval is needed
- The expression matrix is raw counts — normalize first (e.g., cpm from edgeR) because rRoma requires normalized data
- The analysis goal is differential expression per gene rather than coordinated gene set/module activity

## Inputs

- Keyword for MSigDB subset selection (e.g., "HALLMARK"), resolved against Msigdb.v6.2.symbols.all shipped with rRoma
- Custom gene set definitions in GMT format (for ReadGMTFile)
- Normalized gene expression matrix (genes as rows, samples as columns) that the module list will be applied to

## Outputs

- rRoma module list object (e.g., AllHall) with Name and gene membership per geneset, suitable as input to rRoma.R
- Renamed module list with database prefixes removed for plotting

## How to apply

Define the pathway list to test before running rRoma.R by either calling SelectFromMSIGdb("HALLMARK"), which searches MsigDB v6.2 and returns the 50 HALLMARK genesets with gene counts, or loading a GMT file via ReadGMTFile. For compact plot labels, strip the database prefix from module names, e.g. lapply(AllHall, function(x){ x$Name <- sub("HALLMARK_", "", x$Name); x }). Pass the resulting module list as the second argument to rRoma.R together with the normalized (cpm) expression matrix. For a more complete biological view, the authors recommend combining multiple databases rather than relying on HALLMARK alone.

## Related tools

- **rRoma** (Provides SelectFromMSIGdb (MSigDB v6.2 lookup) and ReadGMTFile to build the module list passed to rRoma.R) — https://github.com/sysbio-curie/rROMA

## Examples

```
AllHall <- SelectFromMSIGdb("HALLMARK"); AllHall <- lapply(AllHall, function(x){ x$Name <- sub("HALLMARK_", "", x$Name); x })
```

## Evaluation signals

- SelectFromMSIGdb("HALLMARK") returns exactly 50 genesets, e.g., 'HALLMARK_APICAL_SURFACE (44 genes)' through 'HALLMARK_PANCREAS_BETA_CELLS (40 genes)', with a 'Searching in MsigDB v6.2' message
- After prefix stripping, no module Name still contains 'HALLMARK_'
- Every gene in each module is present in the row names of the expression matrix so rRoma.R can compute PC1 statistics
- A user-supplied GMT loaded via ReadGMTFile yields the same module names and gene memberships as the source file

## Limitations

- MSigDB version is fixed by the rRoma bundle (v6.2 in this vignette); other versions require an external GMT file
- The article notes HALLMARK alone was chosen for explanatory purposes and recommends combining multiple databases for a complete view
- The CF case study lacked statistical power, and the authors suggest testing several gene set reference databases

## Evidence

- [methods] SelectFromMSIGdb retrieves HALLMARK sets: "AllHall <- SelectFromMSIGdb("HALLMARK")"
- [methods] GMT loading alternative: "you can also load them by using the `ReadGMTFile` function"
- [readme] Prefix stripping for plots: "We also remove "HALLMARK_" from the names to simplify graphical representations."
- [readme] Recommendation to combine databases: "we recommend using rRoma with multiple databases combined together."
