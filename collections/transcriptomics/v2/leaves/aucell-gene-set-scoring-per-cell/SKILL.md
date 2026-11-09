---
name: aucell-gene-set-scoring-per-cell
description: Use when you have single-cell/snRNA-seq expression data plus a partitioned
  gene regulatory network (e.g., Leiden subnetworks from a SCING GRN) and need per-cell
  module activity scores for phenotypic association or clustering.
when_to_use_negative:
- The input is already a per-cell feature/module score table rather than a gene expression
  matrix plus gene sets — AUCell rescoring would be redundant.
- You only need TF-target regulon activity from known TFBS motifs (SCENIC's edge pruning
  by TF binding sites), not data-driven GRN subnetwork modules from SCING.
- The expression data are bulk or spatially aggregated without single-cell resolution,
  since AUCell scores are computed per cell.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SCING
  role: Produces the merged, edge-pruned GRN whose Leiden subnetworks define the gene
    sets scored by AUCell
  repo: https://github.com/XiaYangLabOrg/SCING
- name: SCENIC
  role: Provides the AUCell method (via pyscenic) used to compute module-specific
    expression scores for each cell
- name: leiden
  role: Partitions the merged GRN into subnetwork gene modules that become the AUCell
    scoring inputs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:e01f5c2f9429d543da20b018cdc06d50ec0c769201a74c47e9321652de07bae6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - SCENIC
  - SCING
  - enrichR
  - leiden
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# aucell-gene-set-scoring-per-cell

## Summary

Scores each single cell for expression of gene modules (GRN subnetworks) using the AUCell method from SCENIC (pyscenic), enabling downstream clustering, phenotypic association, and pathway-based biological annotation of SCING-derived modules.

## When to use

Use when you have single-cell/snRNA-seq expression data plus a partitioned gene regulatory network (e.g., Leiden subnetworks from a SCING GRN) and need per-cell module activity scores for phenotypic association or clustering.

## When NOT to use

- The input is already a per-cell feature/module score table rather than a gene expression matrix plus gene sets — AUCell rescoring would be redundant.
- You only need TF-target regulon activity from known TFBS motifs (SCENIC's edge pruning by TF binding sites), not data-driven GRN subnetwork modules from SCING.
- The expression data are bulk or spatially aggregated without single-cell resolution, since AUCell scores are computed per cell.

## Inputs

- Merged, edge-pruned SCING gene regulatory network (output of BuildNetwork.ipynb)
- Leiden-partitioned GRN subnetworks (gene modules)
- Single-cell or snRNA-seq expression matrix with per-cell metadata (e.g., diagnosis, plaque stage, tangle stage, sex for the human prefrontal cortex AD snRNAseq data)
- pyscenic installation for AUCell

## Outputs

- Per-cell module-specific expression scores (AUCell module score matrix, modules x cells)
- Module assignments per cell usable for clustering or phenotypic association
- Module statistics (e.g., regression coefficients and FDR) feeding into module_pathway_annotations.csv

## How to apply

After building and edge-pruning the merged GRN with SCING's BuildNetwork procedure, apply the leiden graph partitioning algorithm to divide the network into gene subnetwork modules (ModuleBasedDimensionalityReduction.ipynb). For each Leiden subnetwork, compute AUCell scores (from SCENIC/pyscenic) for every cell in the dataset, yielding a module-by-cell score matrix. In the Alzheimer's disease application, these per-cell module scores were then regressed on diagnosis, plaque stage, and tangle stage with sex as a covariate, and modules accepted as disease-associated at Benjamini-Hochberg FDR < 0.05. Rationale: AUCell converts gene-set membership into ranked per-cell activity, so modules can be used for clustering or phenotypic association without requiring normalized expression means. Ensure pyscenic is installed before running the AUCell step.

## Related tools

- **SCING** (Produces the merged, edge-pruned GRN whose Leiden subnetworks define the gene sets scored by AUCell) — https://github.com/XiaYangLabOrg/SCING
- **SCENIC** (Provides the AUCell method (via pyscenic) used to compute module-specific expression scores for each cell)
- **leiden** (Partitions the merged GRN into subnetwork gene modules that become the AUCell scoring inputs)

## Examples

```
conda activate scing && pip install pyscenic; jupyter notebook tutorials/ModuleBasedDimensionalityReduction.ipynb
```

## Evaluation signals

- Every cell in the dataset receives a finite AUCell score for each Leiden subnetwork module (no missing scores).
- The module score matrix dimensionality matches (number of Leiden modules) x (number of cells in the expression matrix).
- Downstream linear regression of module scores on AD traits (diagnosis, plaque stage, tangle stage) with sex as covariate yields BH-corrected FDR values; ~43% of modules significant at FDR < 0.05 and ~24% significant for all three traits in the reference microglia analysis.
- Significant modules' genes return enrichment terms from GO biological process, DisGeNET, Reactome, BioCarta, and KEGG via enrichR.
- pyscenic is installed and the AUCell step runs without error within the scing conda environment.

## Limitations

- AUCell-based module scores depend on the upstream GRN quality; many SCING hyperparameters require selection and optimization can be dataset dependent.
- Inferred GRN links feeding the modules are not necessarily causal or directional, so module scores reflect co-expression/regulatory structure rather than causation.
- Module interpretation requires the separate PathwayEnrichmentOfModule step (enrichR); AUCell scores alone give no biological annotation.

## Evidence

- [methods] We combine these subnetworks with the AUCell method to get module specific expression for each cell: "We combine these subnetworks with the AUCell method to get module specific expression for each cell"
- [intro] SCING GRNs are then partitioned into modules, to compute module specific expression for each cell. These modules can be used for clustering, phenotypic association: "SCING GRNs are then partitioned into modules, to compute module specific expression for each cell. These modules can be used for clustering, phenotypic association"
- [readme] ModuleBasedDimensionalityReduction.ipynb: Take the merged network, and divide it into subnetworks with the leiden graph partitioning algorithm, finally run the AUCell approach: "ModuleBasedDimensionalityReduction.ipynb: Take the merged network, and divide it into subnetworks with the leiden graph partitioning algorithm, finally run the AUCell approach to get module scores"
- [readme] If you want to use the AUCell from SCENIC for graph based dimensionality reduction you must install pyscenic: "If you want to use the AUCell from SCENIC for graph based dimensionality reduction you must install pyscenic"
- [methods] We apply the leiden graph partitioning algorithm to identify GRN subnetworks: "We apply the leiden graph partitioning algorithm to identify GRN subnetworks"
