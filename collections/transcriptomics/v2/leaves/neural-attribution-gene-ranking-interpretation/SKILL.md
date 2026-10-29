---
name: neural-attribution-gene-ranking-interpretation
description: Use when a trained scSHARP GCN cell-type classifier needs feature interpretation
  — i.e., you must rank genes by DeepLIFT attribution per cell type and/or validate
  those attributions against differential expression marker lists.
when_to_use_negative:
- You need a standalone marker-gene discovery pipeline with no trained GCN — the attributions
  are only defined relative to the scSHARP GCN classifier
- 'You expect DeepLIFT attributions to replace differential expression: the paper
  found many genes not jointly identified by both methods, so it serves as a complementary
  analysis, not a substitute'
- Input is not raw single-cell counts — the scSHARP tooling explicitly requests raw
  counts ('Please input raw counts.')
tools:
- name: scSHARP
  role: Provides the scSHARP.interpret module used to retrieve GCN DeepLIFT gene attributions
    per cell type
  repo: https://github.com/lewinsohndp/scSHARP
- name: scSHARP_tool
  role: Python package implementing the GCN label propagation model whose features
    are interpreted via DeepLIFT
  repo: https://github.com/mperozek11/scSHARP_tool
- name: R4scSHARP
  role: R package supplying the single-cell classification tool suite surrounding
    the GCN; runs within the R-based DE comparison workflow
  repo: https://github.com/W-Holtz/R4scSHARP
- name: PyTorch
  role: Framework in which the GCN pipeline (and hence the DeepLIFT attribution layer)
    is written
- name: R
  role: Runs the Wilcoxon rank-sum differential expression analysis with multiple-testing
    correction used for the overlap comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad360
    title: Consensus label propagation with graph convolutional networks for single-cell
      RNA sequencing cell type annotation
  source_content_hash: sha256:5ff850496e62eba43ab380cc2938c23f9f766c15426ab908aeecaa9af39d43ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lewinsohndp/scSHARP
  tools:
  - PyTorch
  - Python
  - R
  - R4scSHARP
  - scSHARP
  - scSHARP_tool
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# Neural attribution gene ranking interpretation

## Summary

Extracts per-cell-type gene importance scores from a trained scSHARP GCN using DeepLIFT (via the scSHARP.interpret module), then compares the top-ten attributed genes with Wilcoxon differential-expression rankings to assess whether network attributions identify cell type marker genes.

## When to use

Use when a trained scSHARP GCN cell-type classifier needs feature interpretation — i.e., you must rank genes by DeepLIFT attribution per cell type and/or validate those attributions against differential expression marker lists.

## When NOT to use

- You need a standalone marker-gene discovery pipeline with no trained GCN — the attributions are only defined relative to the scSHARP GCN classifier
- You expect DeepLIFT attributions to replace differential expression: the paper found many genes not jointly identified by both methods, so it serves as a complementary analysis, not a substitute
- Input is not raw single-cell counts — the scSHARP tooling explicitly requests raw counts ('Please input raw counts.')

## Inputs

- Normalized scRNA-seq count matrices (raw counts per the README), e.g. testis and PBMC datasets
- Trained scSHARP GCN model with per-cell-type label assignments from the consensus label propagation step
- Prior task output listing top-ten DeepLIFT-attributed genes per cell type
- DE analysis results (Wilcoxon rank-sum adjusted p-value rankings) per cell type

## Outputs

- Per-cell-type top-ten DeepLIFT-attributed gene lists
- Per-dataset/per-cell-type joint overlap genes, DeepLIFT-unique genes, and DE-unique genes
- Tidy CSV table (gene_overlap_table.csv) with one row per dataset/cell-type and joint/DeepLIFT-unique/DE-unique gene fields

## How to apply

Load the normalized scRNA-seq count matrices and the scSHARP interpret module (scSHARP.interpret) to retrieve the GCN DeepLIFT gene attributions for each cell type, extracting the top ten attributed genes per type. In parallel, perform differential expression per cell type in R using a Wilcoxon rank-sum test with multiple-testing correction and rank genes by adjusted significance. Compute, per dataset and cell type, the set of genes jointly identified by DeepLIFT and differential expression plus the method-unique genes for each. Assemble the overlap statistics and gene lists into a tidy one-row-per-dataset/cell-type table (e.g. gene_overlap_table.csv). The rationale, per the paper, is that the GCN's feature interpretation identifies important genes for cell type classification, and the DE comparison reveals whether attributions behave as complementary candidate marker discovery rather than redundancy with classical DE.

## Related tools

- **scSHARP** (Provides the scSHARP.interpret module used to retrieve GCN DeepLIFT gene attributions per cell type) — https://github.com/lewinsohndp/scSHARP
- **scSHARP_tool** (Python package implementing the GCN label propagation model whose features are interpreted via DeepLIFT) — https://github.com/mperozek11/scSHARP_tool
- **R4scSHARP** (R package supplying the single-cell classification tool suite surrounding the GCN; runs within the R-based DE comparison workflow) — https://github.com/W-Holtz/R4scSHARP
- **PyTorch** (Framework in which the GCN pipeline (and hence the DeepLIFT attribution layer) is written)
- **R** (Runs the Wilcoxon rank-sum differential expression analysis with multiple-testing correction used for the overlap comparison)

## Examples

```
import scSHARP.interpret; attributions = scSHARP.interpret.get_deeplift_attributions(gcn_model, counts_matrix)
```

## Evaluation signals

- gene_overlap_table.csv exists and contains a row for every cell type in both datasets (testis and PBMC)
- Each row includes joint / DeepLIFT-unique / DE-unique gene fields
- Overlapping gene entries match the top-ten DeepLIFT list from the prior attribution run
- Overlapping gene entries match the DE gene rankings (Wilcoxon adjusted significance ordering)
- Many common genes are expected between DeepLIFT and DE per cell type, but non-zero method-unique gene sets are expected and are the informative result

## Limitations

- DeepLIFT attributions and DE identify many genes not jointly found by both methods, so attributions are complementary candidate markers rather than confirmation of DE
- Software version numbers for scSHARP and its dependencies are not documented
- Torch and torch geometric must be installed before the pip install of scSHARP; torch-cluster and other PyTorch Geometric dependencies may need manual installation
- The interpretability claim is demonstrated only on the paper's PBMC and testis (Jung) datasets in the provided context

## Evidence

- [abstract] GCN feature interpretation claim: "We then demonstrate that our GCN method allows for feature interpretation, identifying important genes for cell type classification."
- [results] DeepLIFT complementary to DE: [evidence span withheld — anchor status: not_found]
- [methods] interpret module usage: [evidence span withheld — anchor status: not_found]
- [readme] raw counts input requirement: "See demo.ipynb for example work flow.
Please input raw counts."
- [readme] installation dependency ordering: [evidence span withheld — anchor status: external_doc]
