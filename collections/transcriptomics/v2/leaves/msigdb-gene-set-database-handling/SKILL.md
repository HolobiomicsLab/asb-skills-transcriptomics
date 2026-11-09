---
name: msigdb-gene-set-database-handling
description: Use when transforming gene expression profiles into functional spectra
  with DeepCC and you must select, load, or swap the MSigDB gene set database (version,
  count, or custom msigdbr sets) used by getFunctionalSpectra.
when_to_use_negative:
- The input is already a functional spectra matrix (samples x gene sets enrichment
  scores) — no gene set retrieval is needed
- You require gene sets outside the supported sources (MSigDB v5.0, v6.0, v7, or msigdbr
  MSigDBr) — DeepCC's get_gene_sets only serves these MSigDB versions
- You are performing unsupervised clustering of patients — DeepCC is specifically
  developed for classification, not clustering of cancer patients
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: DeepCC
  role: Provides get_gene_sets() to fetch the MSigDB gene set list and getFunctionalSpectra()
    that consumes it to compute one enrichment score per gene set
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: msigdbr
  role: Supplies a custom MSigDB database (MSigDBr) via get_msigdbr() as an alternative
    to the pre-bundled MSigDB versions
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: R
  role: Runtime environment in which get_gene_sets and get_msigdbr are invoked
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:d374ebe18f77654b3e67dcfaf1e8cdfca11dedb019eafd161df01c1e38f8e10a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CityUHK-CompBio/DeepCC
  tools:
  - DeepCC
  - R
  - msigdbr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# msigdb-gene-set-database-handling

## Summary

Retrieves and version-manages MSigDB gene set collections (v5.0: 10,348 sets; v6.0: 17,779 sets; v7: 22,596 sets) inside DeepCC, either via get_gene_sets or the msigdbr R package, to serve as the gene-set universe for GSEA-based functional spectra.

## When to use

Use when transforming gene expression profiles into functional spectra with DeepCC and you must select, load, or swap the MSigDB gene set database (version, count, or custom msigdbr sets) used by getFunctionalSpectra.

## When NOT to use

- The input is already a functional spectra matrix (samples x gene sets enrichment scores) — no gene set retrieval is needed
- You require gene sets outside the supported sources (MSigDB v5.0, v6.0, v7, or msigdbr MSigDBr) — DeepCC's get_gene_sets only serves these MSigDB versions
- You are performing unsupervised clustering of patients — DeepCC is specifically developed for classification, not clustering of cancer patients

## Inputs

- Character/string specification of the MSigDB version (e.g., 'MSigDBv6' for v6.0, 17,779 gene sets)
- MSigDBr object from R package msigdbr via get_msigdbr() (custom gene set database)
- Gene expression profile (data.frame, rows = samples, columns = Entrez gene identifiers) to which the gene sets will be applied

## Outputs

- Gene set list object (gene sets from the chosen MSigDB version, e.g., 17,779 sets for v6.0)
- Functional spectrum matrix (samples x gene sets), one enrichment score per MSigDB gene set per sample

## How to apply

Decide which MSigDB version to use: DeepCC defaults to MSigDB v7 (22,596 gene sets), but the paper's case studies used MSigDB v6.0 with 17,779 gene sets. Load the gene set list by calling get_gene_sets with geneSets set to 'MSigDBv5', 'MSigDBv6', or 'MSigDBv7', or supply a custom database as MSigDBr obtained from the R package msigdbr via get_msigdbr(). Pass the resulting gene sets object into getFunctionalSpectra(eps, geneSets, scale, cores) so that calcEnrichmentScore is computed once per gene set, yielding one functional-spectrum entry per gene set per sample. Rationale: comprehensive, high-quality curated gene sets avoid potential bias in the pathway-activity transformation, so the database should cover the major characteristic signaling pathways for the cancer subtype under study.

## Related tools

- **DeepCC** (Provides get_gene_sets() to fetch the MSigDB gene set list and getFunctionalSpectra() that consumes it to compute one enrichment score per gene set) — https://github.com/CityUHK-CompBio/DeepCC
- **msigdbr** (Supplies a custom MSigDB database (MSigDBr) via get_msigdbr() as an alternative to the pre-bundled MSigDB versions) — https://github.com/CityUHK-CompBio/DeepCC
- **R** (Runtime environment in which get_gene_sets and get_msigdbr are invoked)

## Examples

```
library(msigdbr); MSigDBr <- get_msigdbr(); geneSets <- get_gene_sets(geneSets = "MSigDBv6"); fs <- getFunctionalSpectra(eps, geneSets = geneSets)
```

## Evaluation signals

- The returned gene set list contains the expected count for the chosen version: 17,779 for MSigDB v6.0, 10,348 for v5.0, or 22,596 for v7
- The resulting functional_spectra matrix has exactly one enrichment-score column per gene set (17,779 columns for v6.0), one row per sample
- Every gene set used by calcEnrichmentScore originates from the selected MSigDB version; no mixing of versions across samples
- getFunctionalSpectra completes without gene-set-matching errors, confirming Entrez identifiers align between eps and the gene set database

## Limitations

- Gene set database choice changes the functional spectrum dimensionality (10,348 / 17,779 / 22,596), so models trained on one version cannot be reused with spectra from another
- The prior knowledge of gene sets needs to be comprehensive to avoid potential bias; major characteristic signaling pathways or biological processes for each cancer subtype should be included in the gene set database

## Evidence

- [methods] MSigDB gene sets version selection: "`geneSets = MSigDBr` (MSigDBr attained from R package `msigdbr` with function get_msigdbr() in DeepCC)"
- [discussion] MSigDB v6.0 case-study database: "MSigDB v6.0 database employed in our case studies, which includes 17,779 gene sets"
- [readme] README default version and alternatives: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
- [methods] get_gene_sets helper: "`get_gene_sets` get gene sets list from MSigDB."
- [readme] README msigdbr usage: "MSigDBr attained from R package `msigdbr` with function get_msigdbr() in DeepCC"
