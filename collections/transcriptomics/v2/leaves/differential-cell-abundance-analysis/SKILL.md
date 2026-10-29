---
name: differential-cell-abundance-analysis
description: Use when you have a cell-type-by-sample abundance matrix (e.g., DCNet-inferred
  landscapes from bulk RNA-seq) with two labeled patient subtypes and need to find
  significantly differentially abundant cell types or build a subtype classifier from
  them.
when_to_use_negative:
- You only have raw bulk RNA-seq expression and have not yet run DCNet to infer the
  cell landscape (this skill operates on the inferred abundance matrix, not on expression
  directly).
- Groups are not defined subtypes/labels but continuous phenotypes requiring regression
  or survival modeling rather than two-group differential testing.
- The comparison methods' default parameter setting is acceptable only for benchmarking
  deconvolution accuracy, not for subtype-level differential testing — do not apply
  this skill when the goal is to benchmark DCNet against CIBERSORTx, EPIC, MCP-counter,
  quanTIseq, TIMER, or xCell.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_2640
tools:
- name: DCNet
  role: Deep learning framework that infers per-patient cell-type abundances (>400
    cell types) from bulk RNA-seq; its inferred landscape is the input matrix for
    differential testing
  repo: https://github.com/xindd/DCNet
- name: mxnet
  role: Framework in which the DCNet model (adam optimizer, relu activation, L2loss,
    lr 1e-4, 600 iterations, 256 samples per batch) is coded and run to infer abundances
  repo: https://github.com/xindd/DCNet
- name: Python
  role: Used for fold-change computation, statistical testing with q-value adjustment,
    CSV export, SVM training, and ROC/precision evaluation
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:0c6117a6c641ed7beae07f3fde2dac40e9c119f1f587f766135d29ff5fc55daf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  - mxnet
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# differential-cell-abundance-analysis

## Summary

Identify cell types whose DCNet-inferred abundances differ significantly between two patient groups (e.g., LUAD vs LUSC in TCGA) using per-cell-type fold change and multiple-testing-adjusted statistical tests, then optionally feed the retained differential cell types into an SVM subtype classifier.

## When to use

Use when you have a cell-type-by-sample abundance matrix (e.g., DCNet-inferred landscapes from bulk RNA-seq) with two labeled patient subtypes and need to find significantly differentially abundant cell types or build a subtype classifier from them.

## When NOT to use

- You only have raw bulk RNA-seq expression and have not yet run DCNet to infer the cell landscape (this skill operates on the inferred abundance matrix, not on expression directly).
- Groups are not defined subtypes/labels but continuous phenotypes requiring regression or survival modeling rather than two-group differential testing.
- The comparison methods' default parameter setting is acceptable only for benchmarking deconvolution accuracy, not for subtype-level differential testing — do not apply this skill when the goal is to benchmark DCNet against CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, or xCell.

## Inputs

- DCNet-inferred cell abundance matrix (cell types x samples) for TCGA LUAD and LUSC tumor samples with subtype labels
- TCGA lung cancer gene expression matrix (rows = human gene symbols, columns = samples, log2(x+1)-transformed) for DCNet inference
- DCNet pretrained parameters and index files (index.pkl neuron-to-gene mapping, rix.pkl gene-cell correspondence)

## Outputs

- Filtered differential cell table (differential_cells.csv) with cell type, fold change, and q-value columns and directionality
- Trained SVM subtype classifier with test-set ROC AUC (~0.87) and precision (~0.975) and an ROC curve plot (Figure 6d equivalent)

## How to apply

Load the DCNet-inferred cell abundance matrix for TCGA LUAD and LUSC tumor samples with subtype labels, compute per-cell-type fold change between groups in Python, then test abundance differences per cell type and adjust p-values for multiple testing to q-values. Retain cell types meeting fold change >2 or <0.5 with q<0.01, keeping directionality (higher in LUSC: Neuroepithelial, Myofibroblast, Basal; higher in LUAD: Granulocyte, Alveolar, Leukocyte, Secretory, MDSC, Ciliated). Optionally, split patients 80%/20% into train/test, train an SVM on the differential cell-type abundance features to predict subtype, and compute ROC AUC and precision on the held-out test set. Rationale: DCNet embeds relationships between 434 cell types and their marker genes in the network, so its inferred landscape is expressive enough that a small set of differential cell types discriminates subtypes well.

## Related tools

- **DCNet** (Deep learning framework that infers per-patient cell-type abundances (>400 cell types) from bulk RNA-seq; its inferred landscape is the input matrix for differential testing) — https://github.com/xindd/DCNet
- **mxnet** (Framework in which the DCNet model (adam optimizer, relu activation, L2loss, lr 1e-4, 600 iterations, 256 samples per batch) is coded and run to infer abundances) — https://github.com/xindd/DCNet
- **Python** (Used for fold-change computation, statistical testing with q-value adjustment, CSV export, SVM training, and ROC/precision evaluation)

## Examples

```
python train_luad_lusc_svm.py --expression tcga_lung_log2.csv --index index.pkl --rix rix.pkl --params net_M434_adam_T600_DCNetBnoise-D1.params --subtype-labels subtypes.csv --fc-high 2 --fc-low 0.5 --q 0.01 --out differential_cells.csv
```

## Evaluation signals

- Filtered differential cell set matches the reported directionality: Neuroepithelial, Myofibroblast, Basal higher in LUSC; Granulocyte, Alveolar, Leukocyte, Secretory, MDSC, Ciliated higher in LUAD
- Counts fall near the reported scale: 15 differential cells in LUSC and 18 in LUAD with 12 common differential cell types
- All retained cell types pass the stated thresholds: fold change >2 or <0.5 with q<0.01
- SVM classifier reproduces reported test-set performance: ROC AUC ~0.87 and precision ~0.975 within reasonable tolerance
- differential_cells.csv schema contains exactly cell type, fold change, and q-value columns

## Limitations

- DCNet model was coded with mxnet but no software version numbers, random seed, or reproducibility controls for training are reported
- Trained model weights/checkpoints availability and the fine-tuning procedure for the recommended dropout-0.1 DCNet on user sample data are not specified
- Comparison methods were run with default parameters and training was CPU-only with no hardware specification or runtime reported

## Evidence

- [results] Neuroepithelial, Myofibroblast and Basal cell are higher in LUSC, while Granulocyte, Alveolar, Leukocyte, Secretory, Myeloid-derived suppressor and Ciliated cells are higher in LUAD.: [evidence span withheld — anchor status: not_found]
- [results] Filter cell types meeting fold change >2 or <0.5 with q<0.01, retaining directionality.: [evidence span withheld — anchor status: not_found]
- [results] The SVM classifier constructed from the significantly different cell types distinguished LUAD and LUSC patients with a ROC AUC of 0.87.: [evidence span withheld — anchor status: not_found]
