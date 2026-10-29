---
name: immune-response-gene-restriction-analysis
description: 'Use when you need intercellular interaction predictions tied to a specific
  biological function (e.g., immune response GO: 0006955) on scRNA-seq data, rather
  than the fixed built-in criteria of tools like CellPhoneDB or CellChat.'
when_to_use_negative:
- 'When the goal is unbiased, function-agnostic interaction discovery rather than
  interactions tied to a specific GO biological process — the log2FC > 0.25 and GO:
  0006955 restriction deliberately narrows the candidate set'
- When you require downstream signal transduction prediction inside receiving cells
  — the paper notes NicheNet offers this and InterCellDB does not target it
- When datasets are outside human or mouse — InterCellDB covers only human and mouse
  databases
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: InterCellDB
  role: 'Primary platform for user-defined, function-restricted (GO: 0006955) intercellular
    interaction inference and filtering of gene pairs by action mode and effect'
  repo: https://github.com/ZJUDBlab/InterCellDB
- name: Seurat
  role: Preprocessing of the Davidson melanoma scRNA-seq counts (normalize, scale,
    cluster, cell-type annotation) before interaction analysis
- name: CellPhoneDB
  role: Comparison tool run on the same processed dataset and CAF1-myeloid comparison
    with identical gene filtering for accuracy benchmarking (75%)
- name: NicheNet
  role: Comparison tool run on the same filtered dataset for accuracy benchmarking
    (75%)
- name: CellChat
  role: Comparison tool run on the same filtered dataset for accuracy benchmarking
    (75%)
- name: R
  role: Runtime environment for InterCellDB data calculation and visualization and
    for Seurat preprocessing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/advs.202200045
    title: 'InterCellDB: A User‐Defined Database for Inferring Intercellular Networks'
  source_content_hash: sha256:945e1dd0d05a389b75b7897f9289780ac79a604a5ccdf789cde8e369a2dd455e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUDBlab/InterCellDB
  tools:
  - CellChat
  - CellPhoneDB
  - InterCellDB
  - NicheNet
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# immune-response-gene-restriction-analysis

## Summary

A user-defined InterCellDB analysis that restricts cell–cell interaction inference to differentially expressed genes belonging to a specific GO biological process (here immune response, GO: 0006955), enabling function-focused intercellular network analysis and accuracy benchmarking against other tools.

## When to use

Use when you need intercellular interaction predictions tied to a specific biological function (e.g., immune response GO: 0006955) on scRNA-seq data, rather than the fixed built-in criteria of tools like CellPhoneDB or CellChat.

## When NOT to use

- When the goal is unbiased, function-agnostic interaction discovery rather than interactions tied to a specific GO biological process — the log2FC > 0.25 and GO: 0006955 restriction deliberately narrows the candidate set
- When you require downstream signal transduction prediction inside receiving cells — the paper notes NicheNet offers this and InterCellDB does not target it
- When datasets are outside human or mouse — InterCellDB covers only human and mouse databases

## Inputs

- Mouse scRNA-seq raw count matrix and cell metadata (Davidson et al. melanoma, E-EHCA-2) or equivalent human/mouse scRNA-seq data
- Seurat v3.2 processed object with annotated cell types (CAF1-3, myeloid, cDC, pDC, NK, T, B, endothelial)
- Differentially expressed gene list with log2 fold change values (threshold log2FC > 0.25)
- InterCellDB gene database annotations (cellular localization from COMPARTMENTS, functional classification from UniProt and GO)
- Literature-curated validation labels for predicted gene pairs (e.g., Table S4 basis: experimentally validated for carcinogenesis or anti-tumor function)

## Outputs

- CAF1-to-myeloid interaction network of gene pairs restricted to immune-response (GO: 0006955) genes
- List of predicted ligand–receptor / protein–protein gene pairs with action mode ('binding', 'activation') and effect ('positive') attributes
- Per-tool accuracy table (fraction of literature-supported predicted pairs): InterCellDB 86.6% vs CellPhoneDB/NicheNet/CellChat at 75%
- Network, action-mode, intercellular, and spatial-pattern visualizations from InterCellDB multilayer analysis

## How to apply

Preprocess the scRNA-seq dataset (e.g., Davidson et al. mouse melanoma, E-EHCA-2) with Seurat v3.2 (normalize, scale, cluster, annotate cell types such as CAF1-3, myeloid, cDC, pDC, NK, T, B, endothelial cells). For a chosen cell-pair comparison (e.g., CAF1-to-myeloid), restrict candidate genes to those differentially expressed with log2 fold change > 0.25 AND annotated to the functional category of interest (immune response, GO: 0006955) — InterCellDB supports this via its gene subset selection by subcellular location, molecular function, and involving biological process. Extract matched protein–protein interaction pairs filtered by action mode and effect (e.g., pairs labelled 'activation'/'positive' between CAF1 and myeloid cells, with 'binding' action mode so factors released from source cells can bind receptors in target cells). To benchmark accuracy, curate literature support for each predicted gene pair (counting pairs experimentally validated for carcinogenesis or anti-tumor function) and compute precision as the fraction of literature-supported pairs — in the paper this yielded 86.6% for InterCellDB versus 75% for CellPhoneDB, NicheNet, and CellChat under identical gene filtering. Install InterCellDB from GitHub using package-release version 0.9.2 to run the released code.

## Related tools

- **InterCellDB** (Primary platform for user-defined, function-restricted (GO: 0006955) intercellular interaction inference and filtering of gene pairs by action mode and effect) — https://github.com/ZJUDBlab/InterCellDB
- **Seurat** (Preprocessing of the Davidson melanoma scRNA-seq counts (normalize, scale, cluster, cell-type annotation) before interaction analysis)
- **CellPhoneDB** (Comparison tool run on the same processed dataset and CAF1-myeloid comparison with identical gene filtering for accuracy benchmarking (75%))
- **NicheNet** (Comparison tool run on the same filtered dataset for accuracy benchmarking (75%))
- **CellChat** (Comparison tool run on the same filtered dataset for accuracy benchmarking (75%))
- **R** (Runtime environment for InterCellDB data calculation and visualization and for Seurat preprocessing)

## Examples

```
devtools::install_github("ZJUDBlab/InterCellDB")  # install InterCellDB release v0.9.2, then restrict CAF1-vs-myeloid gene pairs to DE genes (log2FC > 0.25) annotated to GO: 0006955
```

## Evaluation signals

- Verify the output accuracy table contains a value for each of the four tools (InterCellDB, CellPhoneDB, NicheNet, CellChat) and that InterCellDB's accuracy reproduces the reported 86.6% benchmark
- Check that every retained gene pair maps to the interaction database and belongs to the immune response GO term (GO: 0006955) with log2 fold change > 0.25
- Confirm gene-pair filtering used identical gene sets across all four tools so the number of predicted interactions is comparable
- Confirm extracted protein pairs carry the expected attributes: action mode 'binding' and, for CAF1–myeloid, 'activation'/'positive' labels
- Expert review of the literature-curation judgment: each pair counted as supported only if experimentally validated for carcinogenesis or anti-tumor function

## Limitations

- Accuracy estimates depend on manual literature curation of gene pairs, which introduces subjective judgment (the paper used expert_review validation for this step)
- No version information is stated for InterCellDB or NicheNet in the article discussion; users must pin InterCellDB to release v0.9.2 for the tutorial code to run
- Restricting to a single GO term and a single cell-pair comparison (CAF1–myeloid) limits generalization of the accuracy comparison

## Evidence

- [results] We applied InterCellDB on Davidson's dataset for further intercellular network analysis with a focus on genes related to immune response (GO: 0006955).: [evidence span withheld — anchor status: not_found]
- [methods] all protein pairs labelled with "activation" (action mode) and "positive" (action effect) between CAF1 and myeloid cells were extracted.: [evidence span withheld — anchor status: not_found]
- [readme] Gene subset selection by subcellular location, molecular function and involving biological process: "Gene subset selection by subcellular location, molecular function and involving biological process"
- [readme] Please use **package-release version 0.9.2** to run all the codes.: "Please use **package-release version 0.9.2** to run all the codes."
- [results] On the Davidson mouse melanoma dataset, InterCellDB achieved a higher predicted interaction accuracy (86.6%) than the other three tools (CellPhoneDB, NicheNet, CellChat), which reached 75%.: [evidence span withheld — anchor status: not_found]
