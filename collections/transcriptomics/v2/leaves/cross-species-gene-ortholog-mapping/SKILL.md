---
name: cross-species-gene-ortholog-mapping
description: Use when training data and query data come from different species (e.g.
  mouse reference, human query) and cell type labels must be transferred across species
  via orthologous genes.
when_to_use_negative:
- Training and query data are from the same species and share a gene namespace — direct
  gene-space intersection suffices without HomoloGene conversion
- Genes lack reliable one-to-one orthology (e.g. many-to-many paralogue families),
  since the method retains only one-to-one orthologue pairs
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0621
tools:
- name: SciBet
  role: Performs feature selection (SelectGene, ΔS statistic, k=500), multinomial
    model training (Learn), and cross-species classification (Bet) on the orthologue-mapped
    training matrix
  repo: https://github.com/PaulingLiu/scibet
- name: HomoloGene
  role: NCBI Build 68 database used to identify one-to-one homologous genes between
    human and mouse for gene-name conversion
- name: Tabula Muris
  role: Mouse training dataset (>100,000 cells, 20 organs) whose gene names are converted
    to human orthologues to form the cross-species reference
- name: Seurat v3
  role: Defines the UMI preprocessing scheme (size-factor 10,000, add one, log normalization)
    applied to the intersected matrices
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:6bb1e1fc13d8134cf320c77736739b5e7bf1fc5900d3d4eb76dd07c0e994d12f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/scibet
  tools:
  - HomoloGene
  - SciBet
  - Seurat v3
  - Tabula Muris
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-species-gene-ortholog-mapping

## Summary

Restrict a single-cell classifier's feature space to one-to-one human-mouse orthologues via the NCBI HomoloGene Build 68 lookup table, so a model trained on mouse Tabula Muris data can be applied directly to human scRNA-seq queries (achieving ~92% correct mapping of human pancreas cells).

## When to use

Use when training data and query data come from different species (e.g. mouse reference, human query) and cell type labels must be transferred across species via orthologous genes.

## When NOT to use

- Training and query data are from the same species and share a gene namespace — direct gene-space intersection suffices without HomoloGene conversion
- Genes lack reliable one-to-one orthology (e.g. many-to-many paralogue families), since the method retains only one-to-one orthologue pairs

## Inputs

- Annotated mouse scRNA-seq training matrix with cell type labels (e.g. Tabula Muris, >100,000 cells from 20 organs)
- NCBI HomoloGene Build 68 human-mouse homology lookup table
- Human scRNA-seq query expression matrix (e.g. human pancreas datasets)
- Query data normalization mode indicator (full-length vs UMI)

## Outputs

- Human-gene-space training matrix restricted to one-to-one orthologues
- Per-cell predicted cell type labels for the cross-species query cells
- Optional confidence scores (conf_score) for filtering low-confidence assignments
- Sankey diagram linking predicted mouse-derived labels to known human cell types (e.g. Fig. 3b)

## How to apply

Load the annotated training matrix (e.g. Tabula Muris) together with the NCBI HomoloGene Build 68 lookup table, and retain only one-to-one human-mouse orthologue pairs, intersecting gene names between the training set and the lookup table so the training matrix is expressed in human gene space. Normalize the intersected matrix using the paper's scheme — TPM with pseudo-value one plus log normalization for full-length data, or Seurat v3-style size-factor 10,000, add-one, log normalization for UMI data. Run feature selection with SelectGene using the ΔS entropy statistic (default k=500 genes), build the multinomial reference with Learn (Laplace/add-one smoothing), and classify query cells with Bet via maximum likelihood over the informative genes. Optionally compute conf_score against a null dataset with cutoffs from 0.001 to 0.5 to withhold low-confidence assignments. The rationale is that orthologue-restricted gene space makes the multinomial likelihood of the mouse-derived reference directly comparable to human query expression profiles.

## Related tools

- **SciBet** (Performs feature selection (SelectGene, ΔS statistic, k=500), multinomial model training (Learn), and cross-species classification (Bet) on the orthologue-mapped training matrix) — https://github.com/PaulingLiu/scibet
- **HomoloGene** (NCBI Build 68 database used to identify one-to-one homologous genes between human and mouse for gene-name conversion)
- **Tabula Muris** (Mouse training dataset (>100,000 cells, 20 organs) whose gene names are converted to human orthologues to form the cross-species reference)
- **Seurat v3** (Defines the UMI preprocessing scheme (size-factor 10,000, add one, log normalization) applied to the intersected matrices)

## Examples

```
devtools::install_github("PaulingLiu/scibet"); sciBet_res <- Bet(expr = human_pancras_query[, genes], result = sciBet_model)
```

## Evaluation signals

- Fraction of human pancreas cells assigned the correct orthologous cell type is approximately 92%, matching Fig. 3b
- Training matrix gene set equals the intersection of Tabula Muris genes with one-to-one pairs in HomoloGene Build 68 — verify no many-to-many orthologues remain
- Feature-selected gene count equals k=500 informative genes by default
- Cells with conf_score below the chosen cutoff (range 0.001–0.5) are left unassigned rather than force-labeled
- Sankey diagram links each mouse-derived predicted label to its expected orthologous human pancreas cell type

## Limitations

- Only one-to-one orthologues are retained, so genes with ambiguous cross-species homology are excluded from the feature space
- Software version numbers for dependencies (scikit-learn, Seurat v3 minor version, imbalanced-learn) are not specified in the article
- Seeds or randomization parameters for validation splits are unreported, affecting exact reproducibility of benchmark numbers

## Evidence

- [discussion] HomoloGene one-to-one conversion as the cross-species mapping mechanism: "We used the HomoloGene databases provided by NCBI (Build 68) to identify homologous genes between human and mouse"
- [discussion] Gene-name conversion produces a test-set-compatible training set: "the gene names were then converted to human gene names to obtain a test-set-compatible training set for classification"
- [discussion] Training/lookup intersection step: "the intersection gene set between the training dataset (Tabula Muris) and the look-up table"
- [discussion] Feature selection and model construction parameters: "we selected genes with top ΔS as informative genes (500 by default)"
- [discussion] Normalization scheme for UMI data: "normalizing the UMI count of each cell with size-factor 10,000, adding one and then log normalization"
- [readme] Installation of SciBet from GitHub: "devtools::install_github("PaulingLiu/scibet")"
