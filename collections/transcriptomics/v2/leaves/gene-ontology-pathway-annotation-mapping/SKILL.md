---
name: gene-ontology-pathway-annotation-mapping
description: Use when assigning functional titles to gene-set or module gene lists
  by consolidating enrichment evidence (GOTERM, KEGG, BioCarta, GSAn, RcisTarget motifs)
  across all 382 BloodGen3 modules.
when_to_use_negative:
- You are analyzing expression data with the BloodGen3 fingerprint workflow (Groupcomparison,
  Groupcomparisonlimma, gridplot, fingerprintplot) — this skill annotates the modules
  themselves, not sample fingerprints
- The transcriptome is from a non-blood tissue or a non-human species; the article
  states the BloodGen3 repertoire is specifically designed for human blood transcriptome
  data and other tissues/species require separate frameworks
- The input is already a per-module annotation table that only needs title curation
  — re-running the full multi-tool enrichment suite is unnecessary
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: BloodGen3Module (R package)
  role: Ships the Module_listGen3 dataset providing the constitutive gene lists for
    each of the 382 modules that are the input to the enrichment suite
  repo: github.com/Drinchai/BloodGen3Module
- name: DAVID
  role: Ontology-pathway enrichment profiling of each module gene list against GOTERM,
    KEGG, and BioCarta term categories
- name: GSAn
  role: Semantic-similarity annotation of module gene lists, whose labels feed the
    per-module functional title
- name: RcisTarget
  role: Transcription-factor motif enrichment run on the same module gene lists as
    a complementary evidence stream
- name: R
  role: Language in which the BloodGen3Module package and downstream annotation compilation
    are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:f7a6d8542104972f20b7b4a48145c696b8d92917303159b2ba04a6b6a76ee4f8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3Module (R package)
  - DAVID
  - GSAn
  - R
  - RcisTarget
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# gene-ontology-pathway-annotation-mapping

## Summary

Consolidates multi-tool functional enrichment outputs (DAVID-style GOTERM/KEGG/BioCarta, GSAn, RcisTarget, plus published IPA and Literature Lab results) on each of the 382 BloodGen3 module gene lists into a single per-module annotation resource from which top-ranked terms become module functional titles.

## When to use

Use when assigning functional titles to gene-set or module gene lists by consolidating enrichment evidence (GOTERM, KEGG, BioCarta, GSAn, RcisTarget motifs) across all 382 BloodGen3 modules.

## When NOT to use

- You are analyzing expression data with the BloodGen3 fingerprint workflow (Groupcomparison, Groupcomparisonlimma, gridplot, fingerprintplot) — this skill annotates the modules themselves, not sample fingerprints
- The transcriptome is from a non-blood tissue or a non-human species; the article states the BloodGen3 repertoire is specifically designed for human blood transcriptome data and other tissues/species require separate frameworks
- The input is already a per-module annotation table that only needs title curation — re-running the full multi-tool enrichment suite is unnecessary

## Inputs

- Module_listGen3 module-to-gene membership dataset shipped with the BloodGen3Module R package (382 modules)
- Gene symbol lists for each BloodGen3 module
- Annotation term libraries: GOTERM, KEGG, and BioCarta (accessed via DAVID-style profiling)
- Published commercial annotation outputs (IPA, Literature Lab) for cross-matching against GSAn/portal results

## Outputs

- Consolidated per-module annotation spreadsheet (Supplementary Data 1 analogue): module ID, aggregate, top GOTERM/KEGG/BioCarta terms, GSAn label, motif hits
- One assigned functional title per module and per module aggregate

## How to apply

Load the Module_listGen3 dataset shipped with the BloodGen3Module R package to extract the constitutive gene lists for each of the 382 modules of the Gen3 repertoire. For each module gene list, run DAVID-style ontology/pathway enrichment profiling against GOTERM, KEGG, and BioCarta term categories, then complement it with GSAn semantic-similarity annotation and RcisTarget transcription-factor motif enrichment on the same lists. Rank enriched terms per module and select the top annotation per module (and per module aggregate) to serve as the functional title, respecting the two-tiered module-level/aggregate-level grouping that is new for this third-generation repertoire. Rather than re-running commercial tools, match published IPA and Literature Lab outputs against the GSAn/annotation-portal results. Compile all per-module enrichment results into one consolidated spreadsheet (module ID, aggregate, top GOTERM/KEGG/BioCarta terms, GSAn label, motif hits), analogous to Supplementary Data 1.

## Related tools

- **BloodGen3Module (R package)** (Ships the Module_listGen3 dataset providing the constitutive gene lists for each of the 382 modules that are the input to the enrichment suite) — github.com/Drinchai/BloodGen3Module
- **DAVID** (Ontology-pathway enrichment profiling of each module gene list against GOTERM, KEGG, and BioCarta term categories)
- **GSAn** (Semantic-similarity annotation of module gene lists, whose labels feed the per-module functional title)
- **RcisTarget** (Transcription-factor motif enrichment run on the same module gene lists as a complementary evidence stream)
- **R** (Language in which the BloodGen3Module package and downstream annotation compilation are executed)

## Examples

```
devtools::install_github("Drinchai/BloodGen3Module"); library(BloodGen3Module); data("Module_listGen3")
```

## Evaluation signals

- The consolidated spreadsheet contains a row for each of the 382 modules, each with at least one assigned functional annotation
- Each assigned functional title corresponds to the top-ranked enrichment term for that module (and its module aggregate) in the enrichment run
- Module titles nest consistently within the two-tiered module-level/aggregate-level grouping, with modules grouped first by functional annotation then alphabetically within an aggregate
- Commercial annotation outputs (IPA, Literature Lab) are matched against the published GSAn/annotation-portal results rather than treated as independent re-runs
- Module membership resolves against the Gen3 repertoire's probe-level origin (Illumina HT12 v3), with the 388 duplicate probes across modules handled by an explicit choice

## Limitations

- The module repertoire and its functional annotation map are not a ground truth: some aggregates are not homogeneous in functionality, heterogeneity exists within modules, and annotation titles confer subjectivity
- The functional annotation map is expected to continue to evolve even while the fixed framework stays stable
- The repertoire was constructed at the Illumina HT12 v3 BeadArray probe level; 388 out of 14,168 probes are duplicate genes found in different modules, requiring an explicit handling choice

## Evidence

- [intro] extensive functional annotations and reference transcriptome profiles for each of the 382 modules comprising the repertoire have been made available via interactive circle packing plots: "extensive functional annotations and reference transcriptome profiles for each of the 382 modules comprising the repertoire have been made available via interactive circle packing plots"
- [readme] The BloodGen3Module package provides functions for R users to perform module repertoire analyses and generate fingerprint representations.: "The BloodGen3Module package provides functions for R users to perform module repertoire analyses and generate fingerprint representations."
- [discussion] modules within a given aggregate (=row) are grouped first according to functional annotations and then among the different annotations: "modules within a given aggregate (=row) are grouped first according to functional annotations and then among the different annotations"
- [discussion] our module repertoire is not meant to constitute a ground truth. Indeed, some aggregates are clearly not homogeneous in terms of functionality: "our module repertoire is not meant to constitute a ground truth. Indeed, some aggregates are clearly not homogeneous in terms of functionality"
- [discussion] it is likely that the functional annotation map will continue to evolve for the foreseeable future: "it is likely that the functional annotation map will continue to evolve for the foreseeable future"
