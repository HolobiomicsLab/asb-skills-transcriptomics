---
name: transcription-factor-motif-enrichment-interpretation
description: Use when assigning functional titles or annotations to BloodGen3 (or
  similar fixed) module gene lists, and transcription-factor motif enrichment (e.g.
  RcisTarget) is needed as convergent evidence alongside ontology/pathway enrichment.
when_to_use_negative:
- The transcriptome data are from non-blood tissues or non-human species — the BloodGen3
  repertoire is specifically designed for human blood transcriptome data, and other
  contexts require separate frameworks
- The goal is to re-run commercial annotation tools (IPA, Literature Lab) from scratch
  — the workflow matches commercial outputs against published GSAn/annotation-portal
  results instead of re-running them
- The module repertoire is being treated as a ground truth for functional assignment
  — the authors explicitly state the repertoire is not a ground truth and annotation
  titles confer subjectivity
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: RcisTarget
  role: Performs transcription-factor motif enrichment on each module's gene list
    as convergent evidence for functional titles
- name: BloodGen3Module
  role: Supplies the Module_listGen3 constitutive gene lists for the 382 modules and
    the visualization functions (gridplot, fingerprintplot) ordered by the resulting
    annotations
  repo: github.com/Drinchai/BloodGen3Module
- name: BloodGen3
  role: The fixed third-generation module repertoire whose 382 module gene lists are
    the inputs to the motif enrichment analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:fb3fa17a74df7ce6d05c41478d0b68116614a0906f364eb554eb5562d141f451
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3Module
  - R
  - RcisTarget
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# transcription-factor-motif-enrichment-interpretation

## Summary

Interpret transcription-factor motif enrichment computed on fixed BloodGen3 module gene lists as one convergent evidence stream for assigning functional titles to each of the 382 modules in the consolidated per-module annotation report (Supplementary Data 1).

## When to use

Use when assigning functional titles or annotations to BloodGen3 (or similar fixed) module gene lists, and transcription-factor motif enrichment (e.g. RcisTarget) is needed as convergent evidence alongside ontology/pathway enrichment.

## When NOT to use

- The transcriptome data are from non-blood tissues or non-human species — the BloodGen3 repertoire is specifically designed for human blood transcriptome data, and other contexts require separate frameworks
- The goal is to re-run commercial annotation tools (IPA, Literature Lab) from scratch — the workflow matches commercial outputs against published GSAn/annotation-portal results instead of re-running them
- The module repertoire is being treated as a ground truth for functional assignment — the authors explicitly state the repertoire is not a ground truth and annotation titles confer subjectivity

## Inputs

- Module_listGen3 dataset from the BloodGen3Module R package (382 constitutive module gene lists)
- Normalized human blood transcriptome gene lists keyed by valid Gene Symbols
- Published annotation-portal and GSAn results used to match commercial IPA / Literature Lab outputs

## Outputs

- Consolidated per-module annotation spreadsheet (module ID, aggregate, top GOTERM/KEGG/BioCarta terms, GSAn label, motif hits) analogous to Supplementary Data 1
- A functional title per module and per module aggregate based on top-ranked enrichment evidence
- Ordered module groupings for fingerprint grid plots (annotated modules first, alphabetically; unannotated modules last by module ID)

## How to apply

Extract the constitutive gene list for each of the 382 modules by loading the Module_listGen3 dataset shipped with the BloodGen3Module R package. Run TF motif enrichment (RcisTarget) on each module gene list, complementing DAVID-style GOTERM/KEGG/BioCarta profiling and GSAn semantic-similarity annotation on the same lists. Rank enriched terms and motif hits per module and per module aggregate, then select the top-ranked evidence as the module's functional title; for commercial outputs such as IPA or Literature Lab, match against published GSAn/annotation-portal results rather than re-running the tools. Compile all per-module enrichment results (module ID, aggregate, top GOTERM/KEGG/BioCarta terms, GSAn label, motif hits) into one consolidated spreadsheet analogous to Supplementary Data 1, which then drives the ordering of modules on fingerprint grid plots.

## Related tools

- **RcisTarget** (Performs transcription-factor motif enrichment on each module's gene list as convergent evidence for functional titles)
- **BloodGen3Module** (Supplies the Module_listGen3 constitutive gene lists for the 382 modules and the visualization functions (gridplot, fingerprintplot) ordered by the resulting annotations) — github.com/Drinchai/BloodGen3Module
- **BloodGen3** (The fixed third-generation module repertoire whose 382 module gene lists are the inputs to the motif enrichment analysis)

## Examples

```
library(BloodGen3Module); data(Module_listGen3)
```

## Evaluation signals

- The consolidated spreadsheet contains one row for each of the 382 modules with at least one assigned functional annotation
- Each assigned functional title corresponds to the top-ranked enrichment term (including motif hits) for that module
- Each module row carries its two-tiered grouping: module ID plus its module aggregate designation
- Annotated modules are grouped by functional annotation then alphabetically within aggregates, with unannotated modules placed last by module ID, matching grid plot conventions
- Commercial IPA/Literature Lab annotations are traceable to published GSAn/annotation-portal results rather than independently re-run outputs

## Limitations

- Functional annotation titles are subjective and some module aggregates are heterogeneous in functionality; heterogeneity also exists within modules
- The functional annotation map is expected to continue evolving even though the module framework itself stays fixed
- The repertoire was constructed at the probe level (Illumina HT12 v3 BeadArrays); 388 out of 14,168 probes are duplicate genes found in different modules, requiring a handling choice
- Validation of motif-derived titles relies on convergence with multiple annotation sources; motif evidence alone does not establish module function

## Evidence

- [other] workflow source card: [evidence span withheld — anchor status: not_found]
- [intro] intro: "extensive functional annotations and reference transcriptome profiles for each of the 382 modules comprising the repertoire have been made available via interactive circle packing plots"
- [discussion] discussion: "our module repertoire is not meant to constitute a ground truth. Indeed, some aggregates are clearly not homogeneous in terms of functionality, and heterogeneity also exists within modules"
