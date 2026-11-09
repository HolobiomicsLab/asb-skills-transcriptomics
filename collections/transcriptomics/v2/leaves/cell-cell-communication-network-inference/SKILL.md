---
name: cell-cell-communication-network-inference
description: Use when single-cell-resolution spatial transcriptomics data (e.g., seqFISH+
  mouse cortex, Seq-Scope mouse colon) with a gene expression matrix plus cell spatial
  coordinates and cell-type labels is available, and the goal is to estimate which
  neighboring cell types regulate HVG expression.
when_to_use_negative:
- Spatial data lacks single-cell resolution (e.g., spot-level ST without deconvolved
  cell-type labels), since raw adjacent cell-type scores require per-cell coordinates
  and cell-type labels
- Input is bulk RNA-seq or scRNA-seq without spatial coordinates — no adjacent cell-type
  scores can be computed from the coordinate matrix O
- The goal is MLP-based classification of communication-associated genes or MIMO graph
  construction (compare_hvg.zip / divide folder workflows), not the CPPLS network
  inference step itself
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0659
tools:
- name: CPPLS-MLP
  role: R package providing the CPPLS regression workflow (2023_01_seqFISH+.RMD, 2023_01_Seq-Scope.RMD
    scripts) for building the cell-to-cell communication network
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: pls
  role: R package used to fit the Constrained Partial Least Squares regression model
    with 10-fold cross-validation
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: Seurat
  role: R package used to filter the top 2000 HVGs via FindVariableFeatures() before
    regression
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: ComplexHeatmap
  role: Bioconductor dependency installed for visualizing the inferred communication
    network
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: R (version <= 4.2.3)
  role: Runtime environment constraint under which the CPPLS-MLP scripts are executed
  repo: https://github.com/wuzhenao/CPPLS-MLP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:d4f473c91ea1fae58a43ec1869d47df6f5ab71ceec7c2d95dbbc513b5bc8234b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CPPLS-MLP
  - ComplexHeatmap
  - R
  - R (version <= 4.2.3)
  - R (version <=)
  - Seurat
  - pls
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cell-cell-communication-network-inference

## Summary

Infer cell-cell communication networks from single-cell-resolution spatial transcriptomics by regressing highly variable gene (HVG) expression on spatially weighted neighboring cell-type scores using Constrained Partial Least Squares (CPPLS), as implemented in the CPPLS-MLP R package on seqFISH+ and Seq-Scope data.

## When to use

Use when single-cell-resolution spatial transcriptomics data (e.g., seqFISH+ mouse cortex, Seq-Scope mouse colon) with a gene expression matrix plus cell spatial coordinates and cell-type labels is available, and the goal is to estimate which neighboring cell types regulate HVG expression.

## When NOT to use

- Spatial data lacks single-cell resolution (e.g., spot-level ST without deconvolved cell-type labels), since raw adjacent cell-type scores require per-cell coordinates and cell-type labels
- Input is bulk RNA-seq or scRNA-seq without spatial coordinates — no adjacent cell-type scores can be computed from the coordinate matrix O
- The goal is MLP-based classification of communication-associated genes or MIMO graph construction (compare_hvg.zip / divide folder workflows), not the CPPLS network inference step itself

## Inputs

- Gene expression matrix from single-cell-resolution spatial transcriptomics (e.g., seqFISH+ mouse cortex, Seq-Scope mouse colon), provided as a .zip data package (e.g., 2023_01_seqFISH+.zip)
- Cell spatial coordinate matrix O
- Cell-type label vector D
- Top 2000 HVGs selected via Seurat FindVariableFeatures()

## Outputs

- Cell-cell communication network as a coefficient edge list between cell types (CSV or RDS file)
- Filtered CPPLS regression coefficients w_b,h for neighboring cell-type effects on HVGs

## How to apply

Load the gene expression matrix and spatial coordinate matrix into R (version <= 4.2.3 with pls, Seurat, cluster, dplyr, purrr, stringr, ComplexHeatmap installed), remove genes with zero expression across all cells within each cell type, and z-score normalize the remaining genes. Compute raw adjacent cell-type scores from the coordinate matrix O and cell-type label vector D using the log10(distance/dist0) decay formula, then z-score transform them per cell type. Fit a CPPLS regression model for each cell type (e.g., L5 and L6 eNeuron in seqFISH+) with 10-fold cross-validation to estimate coefficients w_b,h for neighboring cell-type effects on HVGs. Filter coefficients in two passes using t-tests on factor loadings with Benjamini-Hochberg FDR correction at threshold delta = 0.05, zeroing non-significant coefficients in each pass, and save the resulting coefficient edge list as a non-empty CSV or RDS file.

## Related tools

- **CPPLS-MLP** (R package providing the CPPLS regression workflow (2023_01_seqFISH+.RMD, 2023_01_Seq-Scope.RMD scripts) for building the cell-to-cell communication network) — https://github.com/wuzhenao/CPPLS-MLP
- **pls** (R package used to fit the Constrained Partial Least Squares regression model with 10-fold cross-validation) — https://github.com/wuzhenao/CPPLS-MLP
- **Seurat** (R package used to filter the top 2000 HVGs via FindVariableFeatures() before regression) — https://github.com/wuzhenao/CPPLS-MLP
- **ComplexHeatmap** (Bioconductor dependency installed for visualizing the inferred communication network) — https://github.com/wuzhenao/CPPLS-MLP
- **R (version <= 4.2.3)** (Runtime environment constraint under which the CPPLS-MLP scripts are executed) — https://github.com/wuzhenao/CPPLS-MLP

## Examples

```
install.packages(c("cluster", "circlize", "dplyr", "pls", "purrr", "Seurat", "stringr"), dependencies = TRUE)
```

## Evaluation signals

- The R script (2023_01_seqFISH+.RMD) runs to completion without errors
- The resulting network output file (CSV or RDS) exists and is non-empty
- Inferred edges capture the L5-L6 eNeuron communication reported for the seqFISH+ dataset
- Two-pass coefficient filtering applied Benjamini-Hochberg FDR correction with threshold delta = 0.05 before producing the final edge list
- On seqFISH+, the full CPPLS-MLP pipeline is expected to yield ~354 cell-communication-related genes as a reference magnitude

## Limitations

- Requires R version <= 4.2.3; the scripts were not validated on newer R versions
- Performance was assessed on only two benchmark datasets (seqFISH+ mouse cortex and Seq-Scope mouse colon)
- No accession IDs or download sources are stated for the seqFISH+ and Seq-Scope datasets beyond the GitHub .zip packages
- Average HVG overlap ratio between cell types was only 0.25 in the seqFISH+ dataset, indicating cell-type-specific HVG sets may share limited genes
- Tool naming is inconsistent across the paper and repository (CPPLS-MLP vs CPPLS_MLP), which can complicate reproducing results

## Evidence

- [methods] Adjacent score computation from coordinates and labels: [evidence span withheld — anchor status: not_found]
- [methods] 10-fold cross-validation input setup: "we input gene expression data and spatial coordinate information into the model and divide these data into 10 subsets of approximately equal size"
- [methods] First coefficient filtering pass: [evidence span withheld — anchor status: not_found]
- [readme] README execution instructions for the CPPLS network scripts: "If you want to use CPPLS to build a cell-to-cell communication network, you can decompress the 2023_01_seqFISH+.zip and 2023_01_Seq-Scope.zip packages and run them in R"
- [methods] HVG pre-filtering step: "CPPLS-MLP initially filters the top 2000 HVGs using the FindVariableFeatures() method"
