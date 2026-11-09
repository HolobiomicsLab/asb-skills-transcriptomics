---
name: cell-metadata-standardization-singlecellexperiment
description: Use when a scRNA-seq SingleCellExperiment (e.g., a Bioconductor ExperimentHub
  anchor dataset such as EH2259, the Kang PBMC IFN-β data) has heterogeneous or missing
  cell-level metadata columns and must be harmonized before muscat aggregation/DS
  analysis.
when_to_use_negative:
- The input is already pseudobulk or a feature table (aggregation has been performed;
  metadata is per-sample rather than per-cell)
- The analysis target is differential detection (DD) or cell-level mixed-model testing
  (mmDS) that does not rely on the standard aggregateData-by-cluster_id/sample_id
  path
- The object is not a SingleCellExperiment and metadata lives in a different structure
  (e.g., Seurat object metadata) without conversion
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: muscat
  role: Provides prepSCE to standardize SingleCellExperiment cell metadata and the
    downstream aggregateData/pbDS functions that consume the standardized sample_id,
    group_id, cluster_id columns
  repo: https://github.com/HelenaLC/muscat
- name: R
  role: Execution environment for muscat and Bioconductor SingleCellExperiment/ExperimentHub
    data loading
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:d5a56ef384ccaaba110a4b5b27104726a935456bdda6f4922766b1823495342b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HelenaLC/muscat
  tools:
  - R
  - edgeR
  - limma-voom
  - muscat
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Cell metadata standardization for SingleCellExperiment

## Summary

Standardize SingleCellExperiment cell metadata (colData) so that sample_id, group_id, and cluster_id columns are present, uniquely coded, and correctly typed before running muscat pseudobulk differential state analyses.

## When to use

Use when a scRNA-seq SingleCellExperiment (e.g., a Bioconductor ExperimentHub anchor dataset such as EH2259, the Kang PBMC IFN-β data) has heterogeneous or missing cell-level metadata columns and must be harmonized before muscat aggregation/DS analysis.

## When NOT to use

- The input is already pseudobulk or a feature table (aggregation has been performed; metadata is per-sample rather than per-cell)
- The analysis target is differential detection (DD) or cell-level mixed-model testing (mmDS) that does not rely on the standard aggregateData-by-cluster_id/sample_id path
- The object is not a SingleCellExperiment and metadata lives in a different structure (e.g., Seurat object metadata) without conversion

## Inputs

- SingleCellExperiment object with cell-level counts and colData (e.g., EH2259 anchor dataset from Bioconductor ExperimentHub)
- Mapping of existing metadata columns to sample_id, group_id, and cluster_id

## Outputs

- Standardized SingleCellExperiment with colData columns sample_id, group_id, and cluster_id ready for muscat::aggregateData and muscat::pbDS

## How to apply

After loading the SingleCellExperiment (e.g., via ExperimentHub id EH2259 for the Kang PBMC IFN-β dataset), pass it through muscat::prepSCE to standardize its colData so the three muscat-required columns exist with consistent coding: "sample_id" holding unique sample identifiers, "group_id" holding the experimental condition (control vs IFN-β stimulated), and "cluster_id" holding subpopulation (cell-type/cluster) assignments. This standardization is a prerequisite because muscat::aggregateData groups counts by c("cluster_id", "sample_id") and muscat::pbDS builds the design from group_id; inconsistent or absent columns break pseudobulk aggregation and the downstream edgeR differential state test. After prepSCE, verify that each sample maps to exactly one group and that cluster assignments cover all cells, then proceed to aggregateData(assay = "counts", fun = "sum").

## Related tools

- **muscat** (Provides prepSCE to standardize SingleCellExperiment cell metadata and the downstream aggregateData/pbDS functions that consume the standardized sample_id, group_id, cluster_id columns) — https://github.com/HelenaLC/muscat
- **R** (Execution environment for muscat and Bioconductor SingleCellExperiment/ExperimentHub data loading)

## Examples

```
library(muscat); sce <- prepSCE(sce, kid = "cell", gid = "stim", sid = "sample"); pb <- aggregateData(sce, assay = "counts", fun = "sum", by = c("cluster_id", "sample_id"))
```

## Evaluation signals

- The standardized object's colData contains exactly the columns sample_id, group_id, and cluster_id with no NA values
- Every sample_id maps to exactly one group_id (e.g., control vs stimulated in the Kang PBMC IFN-β dataset)
- Downstream aggregateData produces pseudobulk profiles for all sample x cluster combinations without errors
- Reproduced per-cluster DS hit counts (FDR < 5%, |logFC| > 1) from pbDS match the muscat vignette-reported numbers on the EH2259 dataset

## Limitations

- prepSCE requires the source colData to already contain information that can be mapped to sample/group/cluster identities; it does not perform clustering itself
- Statistical power in aggregation-based DS analyses is biased toward larger cell populations, so rare subpopulations may yield unreliable results even after correct standardization

## Evidence

- [readme] Let `sce` be a SingleCellExperiment object with cell metadata (colData) columns 1. "sample_id" specifying unique sample identifiers 2. "group_id" specifying each sample's experimental condition 3. "cluster_id" specifying subpopulation (cluster) assignments: [evidence span withheld — anchor status: not_found]
- [readme] Aggregation-based methods come down to the following simple commands: compute pseudobulks (sum of counts): [evidence span withheld — anchor status: external_doc]
- [methods] Load the EH2259 anchor SingleCellExperiment from Bioconductor's ExperimentHub and standardize its cell metadata with muscat::prepSCE.: [evidence span withheld — anchor status: not_found]
- [methods] pb <- aggregateData(sce): "We first load an example dataset and perform a pseudo-bulk differential state analysis"
