---
name: ligand-receptor-network-inference
description: Use when you have a single-cell expression matrix plus cell-type annotations
  and need to predict which ligand-receptor pairs mediate communication between cell
  types, including autocrine/self-signalling and network visualisation.
when_to_use_negative:
- Spatial transcriptomics data — NATMI has not been specifically designed for spatial
  data
- Questions requiring downstream signalling consequence prediction or ligand-receptor
  binding kinetics — NATMI does not model these
- Interactions requiring heteromeric complexes, heterodimerization or co-receptors
  — NATMI does not model them
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: NATMI
  role: 'Core toolkit: ExtractEdges.py infers ligand-receptor edges with specificity
    weights, VisInteractions.py visualises networks, DiffEdges.py compares conditions'
  repo: https://github.com/forrest-lab/NATMI/
- name: LRdb
  role: Alternative ligand-receptor database from SingleCellSignalR that can supplement
    user-supplied interaction lists
  repo: https://github.com/SCA-IRCM/LRdb/blob/master/LRdb_122019.txt
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:15609d9f019e91e7f4d1856b3ff90a0fff032bf6385061e5ec44efd6c3333f03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - LRdb
  - NATMI
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ligand-receptor-network-inference

## Summary

Infer and visualise cell-to-cell communication networks from single-cell gene expression data using NATMI, which scores ligand-receptor edges from connectomeDB2020 by expression detection rates and dataset-specific summed-specificity weights.

## When to use

Use when you have a single-cell expression matrix plus cell-type annotations and need to predict which ligand-receptor pairs mediate communication between cell types, including autocrine/self-signalling and network visualisation.

## When NOT to use

- Spatial transcriptomics data — NATMI has not been specifically designed for spatial data
- Questions requiring downstream signalling consequence prediction or ligand-receptor binding kinetics — NATMI does not model these
- Interactions requiring heteromeric complexes, heterodimerization or co-receptors — NATMI does not model them

## Inputs

- Gene/protein abundance matrix (single-cell or bulk) with official gene symbols or HGNC/Ensembl IDs
- Cell labels metafile mapping each cell to a cell-type annotation (single-cell analysis only)
- Ligand-receptor interaction list: precompiled connectomeDB2020 (2,293 curated pairs) or user-supplied binary pairs

## Outputs

- Per-pair edge tables of ligand-receptor-mediated interactions between cell types with expression, detection rate and specificity weights (ExtractEdges.py output folder)
- Network visualisation PDFs (e.g. network_total-specificity-based_layout_circle.pdf in Network_exp_0_spe_0_det_0.2_top_0_signal_lrc2p_weight_mean folder)
- Delta network of edge-weight changes between two conditions (DiffEdges.py compare folder)

## How to apply

Normalise each expression profile to counts per million (divide by total UMI counts, multiply by 1,000,000), then run NATMI's ExtractEdges.py with --species mouse (or appropriate species via HomoloGene homologues), --interDB lrc2p (the 2,293 literature-supported connectomeDB2020 pairs), and --coreNum 4 to extract edges between all cell types. Apply the default detection threshold of 0.2 (a gene counts as expressed if detected in more than 20% of cells of a cell type) and a 10 CPM expression threshold to retain reliable edges, with NATMI computing summed-specificity weights per edge. Aggregate autocrine edges per cell type or merge edges sharing sending/receiving cell types into summary edges weighted by summed ligand-receptor specificity, then run VisInteractions.py (e.g. --weightType mean --detectionThreshold 0.2 --layout circle) to produce network PDFs. For condition comparisons, use DiffEdges.py on two ExtractEdges output folders to build a delta network. Specificity weights are dataset-specific (local), so interpret rankings within the input dataset only.

## Related tools

- **NATMI** (Core toolkit: ExtractEdges.py infers ligand-receptor edges with specificity weights, VisInteractions.py visualises networks, DiffEdges.py compares conditions) — https://github.com/forrest-lab/NATMI/
- **LRdb** (Alternative ligand-receptor database from SingleCellSignalR that can supplement user-supplied interaction lists) — https://github.com/SCA-IRCM/LRdb/blob/master/LRdb_122019.txt

## Examples

```
python ExtractEdges.py --emFile expression.matrix.txt --annFile annotation.txt --species mouse --interDB lrc2p --out outfolder --coreNum 4
```

## Evaluation signals

- On Tabula Muris, ~31% (719/2293) of connectomeDB2020 ligand-receptor pairs are co-detected in at least 20% of cells of at least one cell type at 10 CPM; median ~54 co-detected pairs per cell type
- Autocrine edges show higher summed-specificity ranks than intra-organ/inter-organ edges
- Top summary edges reproduce manuscript assignments: Nppa-Npr3 (cardiac muscle to endocardial), Sst-Sstr1/Sstr5, Thpo-Mpl (hepatocyte to megakaryocyte-erythroid), fibrinogen Fga/Fgb/Fgg to integrins
- VisInteractions.py output file exists in the expected folder named Network_exp_0_spe_0_det_0.2_top_0_signal_lrc2p_weight_mean
- Edge tables report non-zero specificity weights only for pairs passing the 0.2 detection threshold and expression threshold

## Limitations

- Specificity weights are calculated from the input dataset and are local/dataset-specific, not global; global specificities based on the Human Cell Atlas are deferred to future versions
- Predictions rely on mRNA expression rather than protein concentration and are subject to dropouts
- Networks depend on the original authors' cell annotations, which can affect accuracy; highlighted edges are based on experiments without replicates
- Heteromeric complexes, heterodimerization, co-receptor requirements and binding kinetics are not modelled

## Evidence

- [abstract] NATMI uses connectomeDB2020 (a database of 2293 manually curated ligand-receptor pairs with literature support): "NATMI uses connectomeDB2020 (a database of 2293 manually curated ligand-receptor pairs with literature support)"
- [discussion] normalised by total number of unique molecular identifiers (or reads if the data does not use UMIs) and then rescaled by multiplying by 1,000,000: "normalised by total number of unique molecular identifiers (or reads if the data does not use UMIs) and then rescaled by multiplying by 1,000,000"
- [discussion] we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene: "we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene"
- [methods] --detectionThreshold DETECTIONTHRESHOLD do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2): "--detectionThreshold DETECTIONTHRESHOLD
                        do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2)"
- [discussion] the specificity weights were calculated based on the input dataset and thus should be considered local or dataset specific: "the specificity weights were calculated based on the input dataset and thus should be considered local or dataset specific"
- [readme] NATMI and our updated ligand-receptor lists are freely available to the research community.: "NATMI and our updated ligand-receptor lists are freely available to the research community."
