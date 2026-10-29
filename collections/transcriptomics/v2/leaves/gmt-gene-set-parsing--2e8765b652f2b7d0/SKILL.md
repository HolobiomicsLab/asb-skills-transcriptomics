---
name: gmt-gene-set-parsing
description: Use when a SingCellaR workflow requires curated signature gene sets from
  a .gmt file (e.g. lineage signatures for AUCell scoring or hematopoiesis sets for
  fGSEA) and the sets must be read in and verified first.
when_to_use_negative:
- The input file is not in GMT format (e.g. gene lists as CSV/Excel) — get_gmtGeneSets
  expects a .gmt file
- The goal is de novo discovery of signature gene sets rather than loading already-curated
  sets; the article notes users must compile and curate customized gene sets themselves
  beyond the 75 provided
- The downstream tool is Seurat or another package — SingCellaR lacks automatic object
  transformation to interact with such packages
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SingCellaR
  role: Provides get_gmtGeneSets(gmt.file) and listAvailableGeneSets(gmt.file) to
    parse and verify GMT gene sets used for fGSEA and AUCell annotation
  repo: github.com/supatt-lab/SingCellaR
- name: AUCell
  role: Downstream consumer of the parsed GMT sets for per-cell gene-set enrichment
    scoring (Run_AUCell with geneSets.gmt.file)
- name: R
  role: Runtime environment in which the GMT parsing functions and SingCellaR object
    are executed
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:6a64852c59c5b5a1bf296577483ef8198bcc11414919f9987dcfe6916fce5727
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AUCell
  - R
  - SingCellaR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# gmt-gene-set-parsing

## Summary

Load and verify curated signature gene sets from a GMT file within SingCellaR (e.g. the protocol's 75 hematopoiesis gene sets) using get_gmtGeneSets() and listAvailableGeneSets() before feeding them into GSEA (fGSEA) or AUCell-based cell type annotation.

## When to use

Use when a SingCellaR workflow requires curated signature gene sets from a .gmt file (e.g. lineage signatures for AUCell scoring or hematopoiesis sets for fGSEA) and the sets must be read in and verified first.

## When NOT to use

- The input file is not in GMT format (e.g. gene lists as CSV/Excel) — get_gmtGeneSets expects a .gmt file
- The goal is de novo discovery of signature gene sets rather than loading already-curated sets; the article notes users must compile and curate customized gene sets themselves beyond the 75 provided
- The downstream tool is Seurat or another package — SingCellaR lacks automatic object transformation to interact with such packages

## Inputs

- GMT-format gene set file (e.g. the protocol's 75 curated hematopoiesis gene sets)
- SingCellaR object in R (for downstream AUCell/fGSEA consumption of the parsed sets)

## Outputs

- Parsed gene set list (gene set names with member genes) retrievable via get_gmtGeneSets(gmt.file)
- Verified inventory of available gene sets via listAvailableGeneSets(gmt.file) confirming all 75 curated sets

## How to apply

Read the lineage signature gene sets from the GMT file with get_gmtGeneSets(gmt.file) and confirm the expected sets with listAvailableGeneSets(gmt.file); in this protocol the GMT holds 75 pre-curated hematopoiesis gene sets compiled from previous studies. Verification matters because these parsed sets are subsequently consumed by Run_fGSEA_for_multiple_comparisons (with minSize/maxSize filtering on set size) and Run_AUCell to score per-cell lineage enrichment, so a missing or misnamed set would silently drop out of the NES matrix or the AUCell annotations. Rationale: SingCellaR requires signature gene sets to perform cell type annotation analysis, and the user would otherwise have to compile customized gene sets for a system of interest; parsing the supplied GMT guarantees the annotation step operates on the intended curated universe.

## Related tools

- **SingCellaR** (Provides get_gmtGeneSets(gmt.file) and listAvailableGeneSets(gmt.file) to parse and verify GMT gene sets used for fGSEA and AUCell annotation) — github.com/supatt-lab/SingCellaR
- **AUCell** (Downstream consumer of the parsed GMT sets for per-cell gene-set enrichment scoring (Run_AUCell with geneSets.gmt.file))
- **R** (Runtime environment in which the GMT parsing functions and SingCellaR object are executed)

## Examples

```
R: geneSets <- get_gmtGeneSets(gmt.file); listAvailableGeneSets(gmt.file)
```

## Evaluation signals

- listAvailableGeneSets(gmt.file) reports exactly the expected 75 curated hematopoiesis gene sets
- Downstream AUCell score table and fGSEA NES matrix cover all 75 gene sets with none silently dropped
- fGSEA enrichment directions are consistent with the pre-ranked log2 fold-change ordering per cluster
- AUCell-based UMAP annotations color cells by lineage labels as in the protocol's Figure 7A

## Limitations

- Only 75 pre-curated hematopoiesis gene sets are provided; for other systems the user must compile and curate customized gene sets
- AUCell_buildRankings for large-scale datasets can cause running time and memory issues (mitigated by Build_AUCell_Rankings_Fast with block.cell.size)

## Evidence

- [methods] Read the lineage signature gene sets from the GMT file using get_gmtGeneSets(gmt.file) and confirm the expected sets with listAvailableGeneSets(gmt.file): [evidence span withheld — anchor status: not_found]
- [discussion] SingCellaR requires signature gene sets to perform the cell type annotation analysis.: "SingCellaR requires signature gene sets to perform the cell type annotation analysis."
- [discussion] we provide 75 gene sets curated from previous studies relevant to hematopoiesis: "the user would have to compile and curate customized gene sets for the relevant system of interest. In this protocol, we provide 75 gene sets curated from previous studies relevant to hematopoiesis."
- [readme] The tutorial of how to use the software can be found at https://supatt-lab.github.io/SingCellaR.Doc/: "The tutorial of how to use the software can be found at https://supatt-lab.github.io/SingCellaR.Doc/"
