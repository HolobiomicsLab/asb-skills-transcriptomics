---
name: ligand-receptor-edge-quantification
description: Use when constructing or reproducing ligand-receptor edge weights from
  a normalised single-cell expression matrix plus cell-type annotations, e.g. to validate
  NATMI's ExtractEdges.py weighting scheme or recompute edge_weights.csv independently.
when_to_use_negative:
- You need heteromeric ligand/receptor complexes, co-receptor requirements, downstream
  signalling consequences (NicheNet/SoptSC), or spatial context — NATMI models only
  simple binary ligand-receptor pairs from mRNA abundance
- You need absolute binding kinetics or protein-level concentrations — the weights
  are mRNA-based and subject to dropouts, not protein measurements
- The input is raw, unnormalised counts — weights are only comparable if profiles
  are first normalised per total UMI and rescaled ×1,000,000
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0605
tools:
- name: NATMI
  role: 'Reference implementation: ExtractEdges.py computes mean-expression, specificity,
    and total-expression edge weights from expression and annotation files'
  repo: https://github.com/forrest-lab/NATMI/
- name: connectomeDB2020
  role: Source of the 2293 manually curated literature-supported ligand-receptor pairs
    used as candidate edges (lrc2p)
  repo: https://github.com/forrest-lab/NATMI/
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:ef3c6ed123b58879651ec9049f97e86fd533aa810fcc3804accdfc68450101d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - NATMI
  - Python
  - connectomeDB2020
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# ligand-receptor-edge-quantification

## Summary

Quantify cell-to-cell communication edges as two NATMI weights: a mean-expression weight (mean ligand in sender × mean receptor in receiver) and a specificity weight (product of cell-type-normalised expression fractions, 0–1), after filtering edges through the >20% detection-rate rule on connectomeDB2020 (lrc2p) ligand-receptor pairs.

## When to use

Use when constructing or reproducing ligand-receptor edge weights from a normalised single-cell expression matrix plus cell-type annotations, e.g. to validate NATMI's ExtractEdges.py weighting scheme or recompute edge_weights.csv independently.

## When NOT to use

- You need heteromeric ligand/receptor complexes, co-receptor requirements, downstream signalling consequences (NicheNet/SoptSC), or spatial context — NATMI models only simple binary ligand-receptor pairs from mRNA abundance
- You need absolute binding kinetics or protein-level concentrations — the weights are mRNA-based and subject to dropouts, not protein measurements
- The input is raw, unnormalised counts — weights are only comparable if profiles are first normalised per total UMI and rescaled ×1,000,000

## Inputs

- Counts-per-million-normalised gene expression matrix (e.g. counts divided by total UMI × 1,000,000; toy.sc.em.txt format)
- Cell-label metafile mapping each cell barcode to a cell-type annotation (toy.sc.ann.txt format)
- connectomeDB2020 (lrc2p) curated ligand-receptor pair list, or a user-supplied binary pair list

## Outputs

- edge_weights.csv with one row per filtered ligand-receptor edge per cell-type pair, containing sender cell type, receiver cell type, ligand, receptor, mean-expression weight, specificity weight, and total-expression weight

## How to apply

Load the CPM-normalised expression matrix (divided by total UMIs and multiplied by 1,000,000) and the cell-label metafile with pandas, then aggregate expression per cell type and map each barcode to its annotation. Restrict to literature-supported connectomeDB2020 (lrc2p) pairs, translating mouse gene symbols to the human pair list via NCBI HomoloGene as in ExtractEdges.py --species mouse. Apply the detection filter: a gene counts as expressed only if detected in more than 20% of the cell type's cells (detectionThreshold 0.2 default). For each surviving pair and cell-type combination, compute the mean-expression weight as mean ligand × mean receptor expression (weightType mean), the specificity weight as the product of each gene's share of total mean expression across cell types (range 0–1, where 1 means the pair is detected only in that cell-type combination), and optionally a total-expression weight using summed expression (weightType sum). Emit one row per edge with sender, receiver, ligand, receptor and weight columns.

## Related tools

- **NATMI** (Reference implementation: ExtractEdges.py computes mean-expression, specificity, and total-expression edge weights from expression and annotation files) — https://github.com/forrest-lab/NATMI/
- **connectomeDB2020** (Source of the 2293 manually curated literature-supported ligand-receptor pairs used as candidate edges (lrc2p)) — https://github.com/forrest-lab/NATMI/

## Examples

```
python ExtractEdges.py --species mouse --emFile toy.sc.em.txt --annFile toy.sc.ann.txt --interDB lrc2p --out outfolder
```

## Evaluation signals

- edge_weights.csv exists with one row per filtered ligand-receptor edge per cell-type pair; mean and total weights are non-negative
- All specificity values fall within 0–1; a specificity of 1 occurs only when ligand and receptor are each detected in a single cell type
- The Efnb2-Pecam1 edge shows specificity < 1 with the endothelial-to-endothelial pair among its highest-scoring connections, matching the worked example
- Genes failing the >20% detection-rate threshold in either cell type are absent from the output (detectionThreshold 0.2)
- Recomputed mean-expression weights reproduce ExtractEdges.py output when run with --species mouse --interDB lrc2p on the same CPM-normalised input

## Limitations

- Specificity weights are dataset-specific (local, computed from the input dataset), not global; a Human Cell Atlas-based global specificity reference is not yet available
- Predictions are based on mRNA expression rather than protein concentration and are subject to dropouts; no binding-kinetic estimates are provided
- Heteromeric complexes, heterodimerisation, and co-receptor requirements are not modelled

## Evidence

- [methods] NATMI produces two edge weights per ligand–receptor edge: a mean-expression weight and a specificity weight: [evidence span withheld — anchor status: not_found]
- [discussion] detection threshold 20% of cells: "we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene"
- [discussion] specificity weights are local/dataset-specific: "the specificity weights were calculated based on the input dataset and thus should be considered local or dataset specific"
- [methods] default detectionThreshold 0.2 in VisInteractions: "--detectionThreshold DETECTIONTHRESHOLD do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2)"
- [methods] CPM normalisation rule: "normalized each expression profile by dividing it by the total number of unique molecular identifiers and multiplying by 1,000,000"
