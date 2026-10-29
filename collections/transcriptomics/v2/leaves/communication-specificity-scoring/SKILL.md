---
name: communication-specificity-scoring
description: Use when building or auditing cell-to-cell communication networks from
  single-cell expression data and you need to score ligand-receptor edges by expression
  magnitude and cell-type specificity, as NATMI does.
when_to_use_negative:
- The input includes heteromeric ligand or receptor complexes — NATMI does not model
  them
- The goal is to predict downstream signalling consequences (e.g. NicheNet-style)
  or spatial transcriptomics patterns rather than score ligand-receptor co-expression
  edges
- You need estimates of ligand-receptor binding kinetics or protein-level concentrations
  — NATMI scores are mRNA-based and have no binding-affinity estimates
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: NATMI
  role: Provides ExtractEdges.py, which computes the mean-expression, specificity,
    and total-expression edge weights from the expression matrix and annotation file
  repo: https://github.com/forrest-lab/NATMI/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:ebd514d48f501a1946e12270b9573745e734dd8dda2d333135a712b5f9b745a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - NATMI
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# communication-specificity-scoring

## Summary

Reconstructs NATMI's edge-weight computation, producing a mean-expression weight (product of mean ligand expression in sender and mean receptor expression in receiver) and a specificity weight (product of ligand and receptor cell-type-normalised expression fractions, ranging 0–1), plus an optional total-expression (sum) variant.

## When to use

Use when building or auditing cell-to-cell communication networks from single-cell expression data and you need to score ligand-receptor edges by expression magnitude and cell-type specificity, as NATMI does.

## When NOT to use

- The input includes heteromeric ligand or receptor complexes — NATMI does not model them
- The goal is to predict downstream signalling consequences (e.g. NicheNet-style) or spatial transcriptomics patterns rather than score ligand-receptor co-expression edges
- You need estimates of ligand-receptor binding kinetics or protein-level concentrations — NATMI scores are mRNA-based and have no binding-affinity estimates

## Inputs

- Normalised single-cell gene expression matrix (gene × cell, CPM-rescaled), e.g. toy.sc.em.txt
- Cell labels metafile mapping each cell barcode to a cell-type annotation, e.g. toy.sc.ann.txt
- Ligand-receptor pair list (connectomeDB2020 lrc2p by default, or user-supplied binary pairs)

## Outputs

- Edge table (edge_weights.csv) with one row per filtered ligand-receptor edge per sender-receiver cell-type pair, containing mean-expression weight, specificity weight (0–1), and total-expression weight columns

## How to apply

Load the normalised (CPM-rescaled, counts per million UMIs/reads multiplied by 1,000,000) expression matrix and cell-label metafile with pandas, mapping each cell barcode to its cell-type annotation and aggregating expression per cell type. Restrict analysis to connectomeDB2020 ligand-receptor pairs (lrc2p), mapping gene symbols to human pairs via --species mouse or another supported species from the NCBI HomoloGene homologs. Apply the detection-rate filter — a gene counts as expressed only if detected in more than 20% of the cell type's cells (detectionThreshold default 0.2 in ExtractEdges.py/VisInteractions.py). For each passing ligand-receptor pair and cell-type pair, compute the mean-expression weight as the product of mean ligand expression in the sender and mean receptor expression in the receiver (weightType mean), the specificity weight as the product of the ligand and receptor specificities, where specificity is the cell type's share of mean expression so values fall in 0–1 with 1 meaning the gene is expressed only in that single cell type, and optionally the total-expression weight using summed expression (weightType sum). Assemble sender cell type, receiver cell type, ligand, receptor, and the weight columns into a single edge table (e.g. edge_weights.csv).

## Related tools

- **NATMI** (Provides ExtractEdges.py, which computes the mean-expression, specificity, and total-expression edge weights from the expression matrix and annotation file) — https://github.com/forrest-lab/NATMI/

## Examples

```
python ExtractEdges.py --emFile toy.sc.em.txt --annFile toy.sc.ann.txt --out outfolder --species mouse
```

## Evaluation signals

- edge_weights.csv exists with one row per filtered ligand-receptor edge per cell-type pair
- All mean-expression and total-expression weights are non-negative
- All specificity values fall within 0–1, with 1 occurring only when the gene is detected exclusively in a single cell type
- The Efnb2-Pecam1 edge shows a specificity less than 1, with the endothelial-to-endothelial pair among its highest-scoring connections (worked example check)
- Genes detected in 20% or fewer of a cell type's cells are excluded from edge scoring (detection threshold 0.2 respected)

## Limitations

- Specificity weights are local/dataset-specific, calculated only from the input dataset; global specificities based on e.g. the Human Cell Atlas are deferred to future NATMI versions
- Predictions are based on mRNA expression rather than protein concentration and are subject to dropouts
- Weights depend on the original authors' cell annotations, which can potentially affect accuracy of the predicted networks

## Evidence

- [readme] mean-expression and specificity edge weights definition: "NATMI uses connectomeDB2020 (a database of 2293 manually curated ligand-receptor pairs with literature support)"
- [discussion] specificity weight range interpretation: "the specificity weights were calculated based on the input dataset and thus should be considered local or dataset specific"
- [discussion] detection threshold 20%: "we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene"
- [methods] detectionThreshold default 0.2: "--detectionThreshold DETECTIONTHRESHOLD
                        do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2)"
- [methods] CPM normalisation method: "normalized each expression profile by dividing it by the total number of unique molecular identifiers and multiplying by 1,000,000"
- [discussion] ExtractEdges.py invocation: [evidence span withheld — anchor status: not_found]
