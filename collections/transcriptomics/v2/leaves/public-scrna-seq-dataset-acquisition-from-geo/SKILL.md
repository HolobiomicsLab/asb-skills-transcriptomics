---
name: public-scrna-seq-dataset-acquisition-from-geo
description: Use when an analysis needs the original public mouse pre-implantation
  time-course single-cell count matrices (e.g. GSE38495 for MII oocytes, GSE45719
  for E1-E4.0 stages) plus matched low-input bulk RNA-seq held outside GEO in the
  TimeTalk Figshare deposit.
when_to_use_negative:
- Input is already a processed Seurat object or expression matrix (e.g. the TimeTalk
  package demo seu_demo.rds) — re-download from GEO is unnecessary
- The analysis targets other communication types or maternal-embryo interactions,
  which the TimeTalk framework does not cover — its autocrine hypothesis precludes
  maternal-embryo analysis
- Only simulated/theoretical Granger-causality datasets are needed, which the authors
  generated in-model rather than downloading
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: TimeTalk
  role: R package providing custom scripts and the Figshare data deposit used alongside
    GEO downloads to build pseudo-bulk profiles for each pre-implantation stage
  repo: https://github.com/ChengLiLab/TimeTalk
- name: Seurat 3.2.3
  role: Loads downloaded count matrices into Seurat objects with CreateSeuratObject
    (min.features = 2000, min.cells = 0) prior to QC and downstream analysis
- name: preprocessCore
  role: Quantile-normalizes the merged expression matrix after download and pseudo-bulk
    aggregation (normalize.quantiles, version 1.48.0)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:824824b794d8410393e9a0b984579353470fe952da9fa8aee17ef9b910e38569
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - Seurat
  - Seurat 3.2.3
  - TimeTalk
  - preprocessCore
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Public scRNA-seq dataset acquisition from GEO

## Summary

Retrieves public mouse pre-implantation scRNA-seq count matrices from GEO (GSE38495 MII-oocyte and GSE45719 E1-E4.0) together with the companion low-input bulk RNA-seq files from the TimeTalk Figshare project deposit, providing the raw inputs for pseudo-bulk QC and TimeTalk cell-cell communication analyses.

## When to use

Use when an analysis needs the original public mouse pre-implantation time-course single-cell count matrices (e.g. GSE38495 for MII oocytes, GSE45719 for E1-E4.0 stages) plus matched low-input bulk RNA-seq held outside GEO in the TimeTalk Figshare deposit.

## When NOT to use

- Input is already a processed Seurat object or expression matrix (e.g. the TimeTalk package demo seu_demo.rds) — re-download from GEO is unnecessary
- The analysis targets other communication types or maternal-embryo interactions, which the TimeTalk framework does not cover — its autocrine hypothesis precludes maternal-embryo analysis
- Only simulated/theoretical Granger-causality datasets are needed, which the authors generated in-model rather than downloading

## Inputs

- GEO accession GSE38495 (MII-oocyte scRNA-seq count matrices)
- GEO accession GSE45719 (mouse pre-implantation E1-E4.0 scRNA-seq count matrices)
- Low-input bulk RNA-seq files from the TimeTalk Figshare project deposit (https://figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498)
- CellTalkDB mouse ligand-receptor pairs (2034 pairs) for downstream TimeTalk use

## Outputs

- Raw per-stage scRNA-seq count matrices (genes x cells) for each pre-implantation stage
- Seurat objects built with CreateSeuratObject (min.features = 2000, min.cells = 0) carrying stage labels
- Matched low-input bulk RNA-seq expression profiles per stage for pseudo-bulk correlation QC

## How to apply

Download the GSE38495 (MII-oocyte) and GSE45719 (E1-E4.0) scRNA-seq count matrices from the GEO database, as the TimeTalk authors did under 'Availability of data and materials'; the low-input bulk RNA-seq profiles are not in GEO and must be fetched from the TimeTalk Figshare project deposit (figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498). Work in R (the paper used R 3.6.3) and install the ChengLiLab/TimeTalk package from GitHub (devtools::install_github("ChengLiLab/TimeTalk",ref="main")) to obtain the custom scripts shipped with the deposit. Load the count matrices into Seurat 3.2.3 with CreateSeuratObject using min.features = 2000 and min.cells = 0 to apply the paper's cell-filtering step. Keep stage labels attached to cells so that per-stage pseudo-bulk expression (average single cell per pre-implantation stage) can be computed downstream with the TimeTalk custom R scripts, followed by quantile normalization via preprocessCore::normalize.quantiles.

## Related tools

- **TimeTalk** (R package providing custom scripts and the Figshare data deposit used alongside GEO downloads to build pseudo-bulk profiles for each pre-implantation stage) — https://github.com/ChengLiLab/TimeTalk
- **Seurat 3.2.3** (Loads downloaded count matrices into Seurat objects with CreateSeuratObject (min.features = 2000, min.cells = 0) prior to QC and downstream analysis)
- **preprocessCore** (Quantile-normalizes the merged expression matrix after download and pseudo-bulk aggregation (normalize.quantiles, version 1.48.0))

## Examples

```
devtools::install_github("ChengLiLab/TimeTalk",ref="main")
```

## Evaluation signals

- Both GEO accessions (GSE38495, GSE45719) resolve to count matrices whose cells pass CreateSeuratObject filtering at min.features = 2000
- Low-input bulk RNA-seq files for matching pre-implantation stages are retrievable from the Figshare project 174498 (not GEO)
- After pseudo-bulk aggregation and quantile normalization, pseudo-bulk vs bulk Pearson correlation reaches high agreement (0.91 for the late 2-cell stage), confirming the correct stage labels were retained
- Downloaded scRNA-seq data feed into kBET batch-effect checks in PCA space that show batches well-mixed within random neighborhoods of fixed size
- TimeTalk package installs via devtools::install_github("ChengLiLab/TimeTalk",ref="main") alongside monocle3, cellAlign, and RTN dependencies

## Limitations

- The low-input bulk RNA-seq comparator data are not deposited in GEO; they only exist in the TimeTalk Figshare project deposit, so GEO alone is insufficient
- The TimeTalk README targets Ubuntu (20.04) and notes users can test other Linux distributions; system libraries such as libcairo2-dev are required before installation
- The authors advise users to experiment with various parameter combinations for dependable results, and warn that Granger causality conclusions are affected by the choice of parameter winsz

## Evidence

- [discussion] scRNA-seq datasets were downloaded from the GEO database: "scRNA-seq datasets were downloaded from the GEO database (see Availability of data and materials)."
- [discussion] we have uploaded our code and extensive data to Figshare: "we have uploaded our code and extensive data to Figshare (https://figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498)"
- [discussion] Seurat cell filtering parameters: [evidence span withheld — anchor status: not_found]
- [readme] devtools install of TimeTalk: "devtools::install_github("ChengLiLab/TimeTalk",ref="main")"
- [discussion] Seurat version used: "The merged expression matrix was input to Seurat (version 3.2.3)70 to perform t-SNE dimension reduction"
