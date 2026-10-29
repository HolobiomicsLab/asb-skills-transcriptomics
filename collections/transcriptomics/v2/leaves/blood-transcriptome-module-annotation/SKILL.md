---
name: blood-transcriptome-module-annotation
description: Use when you need per-module functional annotations or titles for BloodGen3
  modules, or need to annotate a normalized blood expression matrix (e.g. GSE13015/EH5429)
  with module membership for aggregates A1–A38.
when_to_use_negative:
- Analyzing tissues other than blood or non-human species — the repertoire is specifically
  designed for human blood transcriptome data
- When you need a ground-truth functional classification — the article states the
  repertoire is not meant to be a ground truth and some aggregates are heterogeneous
- Input data is already at module level (e.g. an existing Group_df or Individual_df),
  so re-annotating gene membership is redundant
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_1775
tools:
- name: BloodGen3Module (R package)
  role: Provides Module_listGen3/Gen3_ann repertoire and module-membership annotation
    of expression matrices used for fingerprint analyses
  repo: github.com/Drinchai/BloodGen3Module
- name: R
  role: Runtime for loading the repertoire, running enrichment workflows, and compiling
    the annotation spreadsheet
- name: BloodGen3
  role: The fixed third-generation repertoire of 382 modules being annotated
provenance:
  source_task_ids:
  - task_003
  - task_008
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:84bd9db32e6e14f7b2fc0b4ac961839748417740df29ce6a47e660e1eeb88ff7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3Module (R package)
  - BloodGen3module (R package)
  - R
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_1775
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# blood-transcriptome-module-annotation

## Summary

Annotates each of the 382 BloodGen3 modules (and their aggregates A1–A38) with functional titles by synthesizing enrichment outputs (GOTERM, KEGG, BioCarta, GSAn, RcisTarget) into a consolidated per-module annotation resource, and annotates expression matrices with module membership via Module_listGen3.

## When to use

Use when you need per-module functional annotations or titles for BloodGen3 modules, or need to annotate a normalized blood expression matrix (e.g. GSE13015/EH5429) with module membership for aggregates A1–A38.

## When NOT to use

- Analyzing tissues other than blood or non-human species — the repertoire is specifically designed for human blood transcriptome data
- When you need a ground-truth functional classification — the article states the repertoire is not meant to be a ground truth and some aggregates are heterogeneous
- Input data is already at module level (e.g. an existing Group_df or Individual_df), so re-annotating gene membership is redundant

## Inputs

- Module_listGen3 / Gen3_ann module repertoire from the BloodGen3Module R package (382 constitutive gene lists with aggregate membership)
- Normalized (non-log2) blood expression matrix with valid Gene Symbols as row names, e.g. GSE13015 via ExperimentHub resource EH5429 (SummarizedExperiment)
- Enrichment tool outputs: DAVID GOTERM/KEGG/BioCarta terms, GSAn labels, RcisTarget motif hits

## Outputs

- Consolidated per-module functional annotation spreadsheet (module ID, aggregate, top GOTERM/KEGG/BioCarta terms, GSAn label, motif hits), analogous to Supplementary Data 1
- Annotated expression matrix with module membership for aggregates A1–A38 used downstream by Groupcomparison/gridplot

## How to apply

Load the Module_listGen3/Gen3_ann repertoire shipped with the BloodGen3Module R package and extract the constitutive gene list for each of the 382 modules. Run DAVID-style ontology-pathway enrichment (GOTERM, KEGG, BioCarta) per module, complemented by GSAn semantic-similarity annotation and RcisTarget transcription-factor motif enrichment on the same gene lists. Rank enriched terms per module and select the top annotation per module and aggregate as the functional title; match commercial outputs (IPA, Literature Lab) against published GSAn/annotation portal results rather than re-running commercial tools. Compile everything into one consolidated spreadsheet analogous to Supplementary Data 1 with module ID, aggregate, top terms, GSAn label, and motif hits. The same Module_listGen3 membership table is also used to annotate expression matrices (e.g. GSE13015 loaded via ExperimentHub resource EH5429) before Groupcomparison or Individualcomparison runs, respecting the fixed grid layout of aggregates A1–A38. Note that 388 of 14,168 probes are duplicates mapping to multiple modules and require a handling decision, though either option yields comparable results.

## Related tools

- **BloodGen3Module (R package)** (Provides Module_listGen3/Gen3_ann repertoire and module-membership annotation of expression matrices used for fingerprint analyses) — github.com/Drinchai/BloodGen3Module
- **R** (Runtime for loading the repertoire, running enrichment workflows, and compiling the annotation spreadsheet)
- **BloodGen3** (The fixed third-generation repertoire of 382 modules being annotated)

## Examples

```
library(BloodGen3Module); data(Module_listGen3); dat = ExperimentHub(); res = query(dat, "GSE13015"); GSE13015 = res[["EH5429"]]
```

## Evaluation signals

- Spreadsheet contains exactly one row for each of the 382 modules, each with at least one assigned functional annotation
- Assigned functional titles correspond to the top-ranked enrichment terms for each module
- Every module maps to one of the 38 aggregates (A1–A38) consistent with the fixed grid layout
- Annotated expression matrix keeps row names as valid Gene Symbols and retains the non-log2 scale required by Groupcomparison/Individualcomparison
- Duplicate probe handling (388/14,168 probes) is recorded, and per-module percent values from either handling option are minimally different

## Limitations

- The module repertoire is not a ground truth; some aggregates are heterogeneous in functionality and heterogeneity also exists within modules
- Functional annotation titles confer subjectivity, and the annotation map is expected to continue evolving even while the framework stays fixed
- The repertoire was constructed at the probe level on Illumina HT12 v3 BeadArrays; 388 of 14,168 probes are duplicates found in different modules requiring a handling choice

## Evidence

- [intro] extensive functional annotations and reference transcriptome profiles for each of the 382 modules comprising the repertoire have been made available via interactive circle packing plots: "extensive functional annotations and reference transcriptome profiles for each of the 382 modules comprising the repertoire have been made available via interactive circle packing plots"
- [methods] Annotating the gene expression data matrix with module membership information.: "Annotating the gene expression data matrix with module membership information."
- [readme] Install BloodGen3Module from GitHub: "devtools::install_github("Drinchai/BloodGen3Module")"
- [discussion] Duplicate probes are a small minority of genes: "which in our hands shows to have very little impact on the results since they concern only a small minority of genes (388 out of 14,168 probes)"
- [readme] Matrix should not be log2 transformed; gene symbols required: [evidence span withheld — anchor status: external_doc]
