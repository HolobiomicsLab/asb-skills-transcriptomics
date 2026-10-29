---
name: module-gene-list-enrichment-analysis
description: Use when you have the constitutive gene lists of a fixed transcriptome
  module repertoire and need per-module functional titles or consolidated enrichment
  annotations to organize modules by biological function.
when_to_use_negative:
- Analyzing transcriptomes from tissues or species other than human blood — the BloodGen3
  repertoire is specifically designed for human blood transcriptome profiling data
- Treating module annotation titles as definitive ground truth when aggregates are
  heterogeneous — the authors state some aggregates are 'clearly not homogeneous in
  terms of functionality'
- The task is differential expression testing or fingerprint visualization on expression
  data (use Groupcomparison/Groupcomparisonlimma and gridplot instead of enrichment
  annotation)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BloodGen3Module
  role: R package supplying the Module_listGen3 module gene lists that serve as enrichment
    inputs
  repo: github.com/Drinchai/BloodGen3Module
- name: DAVID
  role: Ontology-pathway enrichment profiling of module gene lists against GOTERM,
    KEGG, and BioCarta
- name: GSAn
  role: Semantic-similarity based annotation of module gene lists, used to attribute
    functional titles
- name: RcisTarget
  role: Transcription-factor motif enrichment on module gene lists
- name: GSEA
  role: Gene set enrichment analysis framework referenced as supporting functional
    interpretation of module results
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:2d66037ea50ba5f22343c85604dbb23b65d5f3f9ff2b2d0a546a9e267670ad2e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3Module
  - DAVID
  - GSAn
  - GSEA
  - R
  - RcisTarget
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# module-gene-list-enrichment-analysis

## Summary

Functional profiling of each of the 382 BloodGen3 module gene lists (Module_listGen3) using a multi-tool enrichment suite (GSAn, DAVID GOTERM/KEGG/BioCarta, RcisTarget motif analysis, complemented by IPA and Literature Lab), consolidated into a single per-module annotation report used to assign functional titles to modules and aggregates.

## When to use

Use when you have the constitutive gene lists of a fixed transcriptome module repertoire and need per-module functional titles or consolidated enrichment annotations to organize modules by biological function.

## When NOT to use

- Analyzing transcriptomes from tissues or species other than human blood — the BloodGen3 repertoire is specifically designed for human blood transcriptome profiling data
- Treating module annotation titles as definitive ground truth when aggregates are heterogeneous — the authors state some aggregates are 'clearly not homogeneous in terms of functionality'
- The task is differential expression testing or fingerprint visualization on expression data (use Groupcomparison/Groupcomparisonlimma and gridplot instead of enrichment annotation)

## Inputs

- Module_listGen3 dataset (382 module gene lists) from the BloodGen3Module R package
- GOTERM, KEGG, and BioCarta term category libraries
- Transcription-factor motif collection for RcisTarget
- Published/commercial annotation outputs (GSAn, IPA, Literature Lab) for cross-checking

## Outputs

- Consolidated per-module annotation spreadsheet (one row per module: module ID, aggregate, top GOTERM/KEGG/BioCarta terms, GSAn label, motif hits)
- Functional title assigned to each of the 382 modules and their two-tiered module aggregates
- Annotated module repertoire supporting fingerprint grid plot ordering and interactive circle-packing plots

## How to apply

Load the Module_listGen3 dataset shipped with the BloodGen3Module R package to extract the constitutive gene list for each of the 382 modules. For each gene list, run ontology-pathway enrichment profiling against GOTERM, KEGG, and BioCarta term categories in a DAVID-style workflow, and complement it with GSAn semantic-similarity annotation and RcisTarget transcription-factor motif enrichment. Rank enriched terms per module and per module aggregate, select the top-ranked annotation to serve as the functional title, and match commercial outputs (IPA, Literature Lab) against the published GSAn/annotation-portal results rather than re-running commercial tools. Compile all results into one consolidated spreadsheet (module ID, aggregate, top GOTERM/KEGG/BioCarta terms, GSAn label, motif hits) analogous to Supplementary Data 1 of the Nature Communications BloodGen3 paper.

## Related tools

- **BloodGen3Module** (R package supplying the Module_listGen3 module gene lists that serve as enrichment inputs) — github.com/Drinchai/BloodGen3Module
- **DAVID** (Ontology-pathway enrichment profiling of module gene lists against GOTERM, KEGG, and BioCarta)
- **GSAn** (Semantic-similarity based annotation of module gene lists, used to attribute functional titles)
- **RcisTarget** (Transcription-factor motif enrichment on module gene lists)
- **GSEA** (Gene set enrichment analysis framework referenced as supporting functional interpretation of module results)

## Examples

```
library(BloodGen3Module); data(Module_listGen3)
```

## Evaluation signals

- The consolidated spreadsheet contains a row for each of the 382 modules with at least one assigned functional annotation
- Each assigned functional title corresponds to a top-ranked enrichment term for that module (GOTERM, KEGG, BioCarta, GSAn, or motif hit)
- Module membership and aggregate assignment in the annotation table are consistent with the two-tiered module/aggregate grouping of the BloodGen3 repertoire
- Annotated modules, once ordered by annotation then alphabetically, reproduce the ordering of modules within aggregate rows on the fingerprint grid plots
- Unannotated modules are tracked explicitly and listed last (by module ID) rather than dropped from the report

## Limitations

- The module repertoire and its functional annotation map are not ground truth: some aggregates are heterogeneous and the functional annotation titles confer subjectivity
- The functional annotation map is expected to continue evolving while the fixed framework stays in place
- Commercial annotation outputs (IPA, Literature Lab) were matched against published results rather than independently re-run, so reproducibility depends on the published GSAn/portal annotations
- Duplicate genes/probes (388 out of 14,168 Illumina probes) can appear in different modules, requiring a handling choice when building gene lists

## Evidence

- [intro] extensive functional annotations and reference transcriptome profiles for each of the 382 modules comprising the repertoire have been made available via interactive circle packing plots: "extensive functional annotations and reference transcriptome profiles for each of the 382 modules comprising the repertoire have been made available via interactive circle packing plots"
- [discussion] modules within a given aggregate (=row) are grouped first according to functional annotations and then among the different annotations, from left to right according to alphabetical order: "modules within a given aggregate (=row) are grouped first according to functional annotations and then among the different annotations, from left to right according to alphabetical order"
- [discussion] our module repertoire is not meant to constitute a ground truth. Indeed, some aggregates are clearly not homogeneous in terms of functionality, and heterogeneity also exists within modules: "our module repertoire is not meant to constitute a ground truth. Indeed, some aggregates are clearly not homogeneous in terms of functionality"
- [discussion] it is likely that the functional annotation map will continue to evolve for the foreseeable future: "it is likely that the functional annotation map will continue to evolve for the foreseeable future"
- [readme] The BloodGen3Module package provides functions for R users to perform module repertoire analyses and generate fingerprint representations.: "The **BloodGen3Module** package provides functions for R users to perform module repertoire analyses and generate fingerprint representations."
