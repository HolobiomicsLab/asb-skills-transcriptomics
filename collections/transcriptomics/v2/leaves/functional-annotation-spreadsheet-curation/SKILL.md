---
name: functional-annotation-spreadsheet-curation
description: Use when you need to build or reproduce a consolidated per-module functional
  annotation table for a fixed transcriptional module repertoire, attaching a functional
  title to each module based on top-ranked enrichment terms.
when_to_use_negative:
- Input transcriptome data is from tissues other than blood or from non-human species
  — the BloodGen3 repertoire is specifically designed for human blood transcriptome
  profiling data
- The module repertoire is being treated as a ground truth of function without re-running
  or checking the underlying enrichment evidence — the authors state some aggregates
  are not homogeneous and annotation titles confer subjectivity
- The goal is differential expression testing or fingerprint visualization (Groupcomparison,
  gridplot, fingerprintplot) rather than curating per-module functional annotations
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: BloodGen3Module (R package)
  role: Provides the Module_listGen3 dataset and the fixed 382-module repertoire whose
    gene lists are annotated
  repo: https://github.com/Drinchai/BloodGen3Module
- name: R
  role: Environment used to load module membership and compile the consolidated annotation
    spreadsheet
- name: BloodGen3
  role: The fixed module repertoire (382 modules) whose per-module functional annotations
    are curated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:27ba2628c0b796fcd1025efcda3485fc19ed35b1de5de996b19ff88d8fecade9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Drinchai/BloodGen3Module
  tools:
  - BloodGen3
  - BloodGen3Module (R package)
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# functional-annotation-spreadsheet-curation

## Summary

Consolidates the outputs of a multi-tool enrichment suite (GSAn, DAVID-style GOTERM/KEGG/BioCarta, RcisTarget, plus commercial IPA and Literature Lab evidence) run on each of the 382 BloodGen3 module gene lists into a single per-module annotation spreadsheet (analogous to Supplementary Data 1), from which convergent evidence is used to assign functional titles to modules and module aggregates.

## When to use

Use when you need to build or reproduce a consolidated per-module functional annotation table for a fixed transcriptional module repertoire, attaching a functional title to each module based on top-ranked enrichment terms.

## When NOT to use

- Input transcriptome data is from tissues other than blood or from non-human species — the BloodGen3 repertoire is specifically designed for human blood transcriptome profiling data
- The module repertoire is being treated as a ground truth of function without re-running or checking the underlying enrichment evidence — the authors state some aggregates are not homogeneous and annotation titles confer subjectivity
- The goal is differential expression testing or fingerprint visualization (Groupcomparison, gridplot, fingerprintplot) rather than curating per-module functional annotations

## Inputs

- Module_listGen3 dataset from the BloodGen3Module R package (constitutive gene lists for 382 modules)
- Module gene list per module (Gene Symbols)
- Enrichment outputs: GOTERM, KEGG, BioCarta (DAVID-style), GSAn annotations, RcisTarget motif hits
- Published commercial annotation evidence (IPA, Literature Lab) for cross-checking

## Outputs

- Consolidated per-module annotation spreadsheet (one row per module: module ID, aggregate, top GOTERM/KEGG/BioCarta terms, GSAn label, motif hits), analogous to Supplementary Data 1
- Assigned functional title per module and per module aggregate

## How to apply

Load the Module_listGen3 dataset shipped with the BloodGen3Module R package (installable via devtools::install_github("Drinchai/BloodGen3Module") or BiocManager) to extract the constitutive gene list for each of the 382 modules. For each module gene list, run ontology-pathway enrichment against GOTERM, KEGG, and BioCarta term categories (DAVID-style), complement it with GSAn semantic-similarity annotation and RcisTarget transcription-factor motif enrichment on the same lists, and rank enriched terms per module. Match commercial annotation outputs (IPA, Literature Lab) against the published GSAn/annotation-portal results rather than re-running the commercial tools, then select the top-ranked annotation per module and per module aggregate to serve as the functional title. Compile everything into one spreadsheet with columns for module ID, aggregate, top GOTERM/KEGG/BioCarta terms, GSAn label, and motif hits — the two-tiered grouping at module level and aggregate level is specific to the Gen3 repertoire.

## Related tools

- **BloodGen3Module (R package)** (Provides the Module_listGen3 dataset and the fixed 382-module repertoire whose gene lists are annotated) — https://github.com/Drinchai/BloodGen3Module
- **R** (Environment used to load module membership and compile the consolidated annotation spreadsheet)
- **BloodGen3** (The fixed module repertoire (382 modules) whose per-module functional annotations are curated)

## Examples

```
install.packages("devtools"); devtools::install_github("Drinchai/BloodGen3Module")
```

## Evaluation signals

- The spreadsheet contains exactly one row for each of the 382 modules, each with at least one assigned functional annotation
- Each assigned functional title corresponds to the top-ranked enrichment term for that module (GOTERM/KEGG/BioCarta, GSAn label, or motif hit)
- Module IDs and aggregate names match the two-tiered module/aggregate grouping of the Gen3 repertoire
- Commercial tool annotations (IPA, Literature Lab) agree with the published GSAn/annotation-portal results rather than contradicting them
- Modules without usable enrichment hits are explicitly left unannotated rather than assigned spurious titles

## Limitations

- The module repertoire and its functional annotation map are not meant to constitute a ground truth; some aggregates are clearly not homogeneous in terms of functionality, and heterogeneity also exists within modules
- The functional annotation map is expected to continue evolving for the foreseeable future, so titles assigned here may change while the module framework itself stays fixed
- Assigning functional titles introduces subjectivity, since convergent evidence from multiple annotation tools must be reconciled into a single title per module

## Evidence

- [intro] Provide extensive functional annotations and reference transcriptome profiles for each of the 382 modules: "extensive functional annotations and reference transcriptome profiles for each of the 382 modules comprising the repertoire have been made available"
- [discussion] Annotations are organized by the two-tiered module/aggregate grouping: "modules within a given aggregate (=row) are grouped first according to functional annotations and then among the different annotations"
- [discussion] Annotations are not ground truth: "our module repertoire is not meant to constitute a ground truth. Indeed, some aggregates are clearly not homogeneous in terms of functionality"
- [methods] Package installation entry point for loading module gene lists: "devtools::install_github("Drinchai/BloodGen3Module")"
- [readme] Scope of the package relative to annotation use: [evidence span withheld — anchor status: not_found]
