---
name: adt-background-threshold-positive-cell-calling
description: Use when a CITE-seq dataset contains ADT counts alongside gene expression
  and per-cell antibody positivity must be called rather than inferred only from clustering,
  e.g. surface-marker phenotyping on 10x/cellranger data.
when_to_use_negative:
- The dataset is single-modality scRNA-seq without ADT counts (no antibody expression
  to threshold).
- The ADT data has not been filtered to the cells retained by GEX QC, so the two modalities
  describe different cell sets.
- The goal is sample hashing deconvolution rather than per-cell antibody positivity
  assignment.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: gExcite
  role: Workflow providing the CITE-seq analysis layer in which joint BremSC clustering
    and per-antibody background-threshold positive-cell calling are executed
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: Snakemake
  role: Workflow manager embedding and executing the gExcite CITE-seq analysis steps
    with per-rule conda environments
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: scAmpi
  role: Included module providing the basic scRNA processing steps (QC, normalisation)
    whose retained cells define the cell set used for ADT analysis
  repo: https://github.com/ETH-NEXUS/scAmpi_single_cell_RNA
- name: Cell Ranger (cellranger)
  role: Performs cellranger count, the resource-intensive upstream step that must
    be bypassed (via dummy FASTQs) to start the quick test at the ADT analysis stage
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:df00d441a275e2a04d065a555ec3b644a8358d102ba52050704060a8760d5c49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ETH-NEXUS/gExcite_pipeline
  tools:
  - Cell Ranger (cellranger)
  - Snakemake
  - gExcite
  - scAmpi
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# adt-background-threshold-positive-cell-calling

## Summary

Part of gExcite's CITE-seq analysis layer, this step applies per-antibody background thresholds to ADT (antibody-derived tag) counts to classify individual cells as antibody-positive, run jointly with gene-plus-antibody (BremSC) clustering.

## When to use

Use when a CITE-seq dataset contains ADT counts alongside gene expression and per-cell antibody positivity must be called rather than inferred only from clustering, e.g. surface-marker phenotyping on 10x/cellranger data.

## When NOT to use

- The dataset is single-modality scRNA-seq without ADT counts (no antibody expression to threshold).
- The ADT data has not been filtered to the cells retained by GEX QC, so the two modalities describe different cell sets.
- The goal is sample hashing deconvolution rather than per-cell antibody positivity assignment.

## Inputs

- ADT (antibody-derived tag) count matrix from CITE-seq preprocessing, restricted to cells retained by GEX QC
- Gene expression (GEX) count matrix from cellranger count / scAmpi QC
- BremSC joint gene-plus-antibody cluster labels

## Outputs

- Per-antibody antibody-positive cell calls (per cell x per ADT binary classifications)
- Ridge plots showing per-antibody count distributions with thresholds
- Joint cluster labels from BremSC clustering

## How to apply

Within the gExcite Snakemake workflow, first restrict the ADT count data to the cells retained by gene-expression (GEX) QC so the two modalities describe the same cell set. Run joint gene-plus-antibody (BremSC) clustering over the combined GEX and ADT data, then compute a background threshold per antibody and call each cell positive or negative for that antibody based on its count relative to the antibody's background distribution. When an ADT-based UMAP is also produced, ensure the number of PCA dimensions does not exceed the number of ADTs in the experiment, otherwise the script fails. Report positive-cell calls together with ridge plots per antibody so threshold placement can be inspected visually.

## Related tools

- **gExcite** (Workflow providing the CITE-seq analysis layer in which joint BremSC clustering and per-antibody background-threshold positive-cell calling are executed) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **Snakemake** (Workflow manager embedding and executing the gExcite CITE-seq analysis steps with per-rule conda environments) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **scAmpi** (Included module providing the basic scRNA processing steps (QC, normalisation) whose retained cells define the cell set used for ADT analysis) — https://github.com/ETH-NEXUS/scAmpi_single_cell_RNA
- **Cell Ranger (cellranger)** (Performs cellranger count, the resource-intensive upstream step that must be bypassed (via dummy FASTQs) to start the quick test at the ADT analysis stage)

## Examples

```
snakemake --use-conda --printshellcmds --cores 1
```

## Evaluation signals

- The quick test run completes through the ADT analysis steps without errors and emits positive-cell calls for the three hashed PBMC samples
- Positive-cell calls and per-antibody ridge plots are produced together as CITE-seq analysis outputs
- ADT count matrix is restricted to exactly the cells retained by GEX QC (no cell mismatch between modalities)
- For any ADT-based UMAP, the number of PCA dimensions is <= the number of ADTs in the experiment (exceeding this fails the script)
- Per-antibody thresholds are computed independently (one background threshold per antibody), not a single global cutoff

## Limitations

- The number of PCA dimensions cannot be larger than the number of ADTs in the experiment at hand, otherwise the script fails
- gExcite is compatible with and tested on Linux only; other Unix systems including MacOS are currently not supported
- Starting at the ADT analysis stage (skipping cellranger count and CITE-Seq preprocessing) requires the presence of dummy files in place of the FASTQ files

## Evidence

- [other] Run joint gene-plus-antibody (BremSC) clustering and apply per-antibody background thresholds to call antibody-positive cells.: [evidence span withheld — anchor status: not_found]
- [discussion] PCA dimension limit for ADT UMAP: "The number of PCA dimensions cannot be larger than the number of ADTs in the experiment at hand, otherwise the script fails."
- [readme] CITE-seq analysis scope of gExcite: "gExcite is a start-to-end workflow embedded in Snakemake that provides both, gene expression and CITE-seq analysis, as well as hashing deconvolution."
