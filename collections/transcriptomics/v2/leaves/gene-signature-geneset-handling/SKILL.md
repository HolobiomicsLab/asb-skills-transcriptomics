---
name: gene-signature-geneset-handling
description: Use when assembling gene signatures for UCell/AddModuleScore_UCell scoring
  — e.g. building Tcell, Myeloid, NK, Plasma_cell marker lists on a scRNA-seq matrix,
  marking negative markers with '-', or deciding how absent genes should be treated.
when_to_use_negative:
- Genes are binned by dataset-wide average expression for normalization (Seurat AddModuleScore-style
  context) — this skill's rank-based signatures target UCell, which depends only on
  per-cell relative ranks
- Input signatures use identifiers incompatible with the dataset gene symbols (e.g.
  Ensembl IDs) without prior mapping — UCell will treat unmapped genes as missing
- The task is cell-type classification/purification from markers, not signature scoring
  — scGate builds on UCell for that purpose instead
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: UCell
  role: Scores the assembled named gene-set list; supports positive/negative gene
    sets and missing_genes impute/skip handling
  repo: https://github.com/carmonalab/UCell
- name: Seurat
  role: Provides the Seurat object to which signatures are applied via AddModuleScore_UCell
  repo: https://github.com/satijalab/seurat
- name: SignatuR
  role: Stores and retrieves gene signatures (data.tree by species/category) for use
    as UCell features
  repo: https://github.com/carmonalab/SignatuR
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.06.043
    title: 'UCell: Robust and scalable single-cell gene signature scoring'
  source_content_hash: sha256:9fa7289dd0f7ca39077a3d5fa8def86fdad913d6fb70387a7f8f252009bb68a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/UCell
  tools:
  - R
  - Seurat
  - SignatuR
  - UCell
  - data.table
  - future
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gene-signature-geneset-handling

## Summary

Defines and formats cell-type gene signatures (named lists of marker genes, optionally with +/- positive/negative members) for UCell scoring, handling absent genes via 'impute' (set to rmax+1) or 'skip' (drop from rank sum).

## When to use

Use when assembling gene signatures for UCell/AddModuleScore_UCell scoring — e.g. building Tcell, Myeloid, NK, Plasma_cell marker lists on a scRNA-seq matrix, marking negative markers with '-', or deciding how absent genes should be treated.

## When NOT to use

- Genes are binned by dataset-wide average expression for normalization (Seurat AddModuleScore-style context) — this skill's rank-based signatures target UCell, which depends only on per-cell relative ranks
- Input signatures use identifiers incompatible with the dataset gene symbols (e.g. Ensembl IDs) without prior mapping — UCell will treat unmapped genes as missing
- The task is cell-type classification/purification from markers, not signature scoring — scGate builds on UCell for that purpose instead

## Inputs

- Named list of gene-set character vectors (gene symbols, optionally with +/- suffixes)
- Gene-vs-cell expression matrix or Seurat/SingleCellExperiment object whose gene names must match signature genes
- Curated signature database entries (e.g. SignatuR data.tree of species/category-organized signatures)

## Outputs

- Gene-signature list formatted as the 'features' argument for UCell scoring
- Per-cell signature score table (one column per signature) after scoring, scores bounded in [0,1]

## How to apply

Assemble signatures as a named R list of character vectors of gene symbols, e.g. the Zilionis NSCLC demo sets Tcell (CD3D, CD3E, CD3G, CD2, TRAC), Myeloid (CD14, LYZ, CSF1R, FCER1G, SPI1), NK (KLRD1, NCR1, NKG7) and Plasma_cell (MZB1, DERL3). Append '+' or '-' to gene names to define positive and negative sets within a signature (e.g. c("CD2+","CD8A+","CD4-")), which UCell evaluates within a single signature score. When a signature contains genes absent from the dataset (e.g. the deliberately missing 'not_a_gene'), choose missing_genes='impute' to set absent genes to rank rmax+1 (default 1500) or missing_genes='skip' to drop them from the rank sum — this choice changes the affected score columns. For retrieval of curated signatures, the SignatuR package stores marker sets by species and category. Pass the named list as the 'features' argument to ScoreSignatures_UCell or AddModuleScore_UCell.

## Related tools

- **UCell** (Scores the assembled named gene-set list; supports positive/negative gene sets and missing_genes impute/skip handling) — https://github.com/carmonalab/UCell
- **Seurat** (Provides the Seurat object to which signatures are applied via AddModuleScore_UCell) — https://github.com/satijalab/seurat
- **SignatuR** (Stores and retrieves gene signatures (data.tree by species/category) for use as UCell features) — https://github.com/carmonalab/SignatuR

## Examples

```
gene.sets <- list(Tcell=c("CD2","CD3E","CD3D"), Myeloid=c("SPI1","FCER1G","CSF1R")); scores <- ScoreSignatures_UCell(sample.matrix, features=gene.sets)
```

## Evaluation signals

- All signature genes with +/- suffixes resolve to genes present in the dataset matrix, or the missing_genes policy (impute/skip) is explicitly chosen
- Resulting per-cell scores are normalized U statistics (AUC-equivalent) bounded between 0 and 1
- Identical gene-set lists and parameter settings reproduce the vignette reference scores; only columns affected by changed missing_genes or maxRank settings differ
- Signatures with deliberately absent genes (e.g. 'not_a_gene' in Myeloid) produce different scores under missing_genes='impute' vs 'skip', confirming the policy took effect

## Limitations

- Genes absent from the dataset must be handled explicitly ('impute' sets them to rmax+1; 'skip' drops them from the rank sum); results differ between the two settings
- Single-cell data are sparse, so signature scores may need imputation by neighboring cells (SmoothKNN kNN-weighted averaging) to partially correct sparsity
- UCell's default rank cap rmax=1500 may need lowering (e.g. 800-1000) for low-depth datasets, which changes how many signature genes carry informative ranks

## Evidence

- [readme] Positive/negative gene sets supported: "Note that UCell supports **positive and negative gene sets** within a signature. Simply append + or - signs to the genes to include them in positive and negative sets"
- [readme] Named list input to scoring: "gene.sets <- list(Tcell_signature = c("CD2","CD3E","CD3D"),
			Myeloid_signature = c("SPI1","FCER1G","CSF1R"))

scores <- ScoreSignatures_UCell(sample.matrix, features=gene.sets)"
- [methods] Rank cap rationale: "we set rg,c = rmax + 1 for all rg,c > rmax, with rmax = 1500 by default"
- [methods] Score definition via Mann-Whitney U: "To evaluate a gene signature s composed of n genes (s1 ,…,sn), we calculate a UCell score U’ for each cell j in X"
- [methods] Rank-based robustness: "This score depends only on the gene expression ranks of individual cell, and therefore is robust across datasets."
- [other] Zilionis demo signatures: [evidence span withheld — anchor status: not_found]
