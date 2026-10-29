---
name: cell-cell-communication-inference-from-scrnaseq
description: Use when scRNA-seq data has cluster/cell-type annotations and the goal
  is to compute transcriptome-based communication scores between sender and receiver
  clusters and rank specific ligand-receptor pairs (e.g., checkpoints, chemokines).
when_to_use_negative:
- The input is bulk or sorted-population transcriptomic data where a per-cluster mean
  of single cells is not the target — use the sorted cell-population workflow (Primary
  Cell Atlas reference, 33 cell types) instead
- Downstream signaling pathway effects or intracellular effectors must be modeled
  — ICELLNET explicitly does not integrate downstream effectors at this stage
- Only single-cell-level spatially restricted contact detection is needed and distant-source
  signals should be excluded — ICELLNET is designed to integrate signals from both
  proximal and distal cells
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: ICELLNET
  role: 'Core framework: provides the expert-curated ligand-receptor database, communication-score
    quantification, and visualization of cell-cell communication from scRNA-seq profiles'
  repo: https://github.com/soumelis-lab/ICELLNET
- name: devtools
  role: R package used to install ICELLNET from GitHub via install_github
- name: data.table
  role: R dependency for loading and manipulating the expression matrix and cluster
    annotations
- name: R
  role: Runtime environment; ICELLNET v2.0.0 requires R < 4.1, use ICELLNET v2.2.x
    for R > 4.1.x
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:d5be5dec27252039e5c35f4349dc0d46f520227c015f9d0cbdd250dc164cce03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soumelis-lab/ICELLNET
  tools:
  - ICELLNET
  - R
  - data.table
  - devtools
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# cell-cell-communication-inference-from-scrnaseq

## Summary

Quantify intercellular communication potential between cell clusters from single-cell RNA-seq data using the ICELLNET framework, which scores a manually curated human ligand-receptor database (>1600 interactions, accounting for multi-subunit complexes) against cluster-mean expression profiles.

## When to use

Use when scRNA-seq data has cluster/cell-type annotations and the goal is to compute transcriptome-based communication scores between sender and receiver clusters and rank specific ligand-receptor pairs (e.g., checkpoints, chemokines).

## When NOT to use

- The input is bulk or sorted-population transcriptomic data where a per-cluster mean of single cells is not the target — use the sorted cell-population workflow (Primary Cell Atlas reference, 33 cell types) instead
- Downstream signaling pathway effects or intracellular effectors must be modeled — ICELLNET explicitly does not integrate downstream effectors at this stage
- Only single-cell-level spatially restricted contact detection is needed and distant-source signals should be excluded — ICELLNET is designed to integrate signals from both proximal and distal cells

## Inputs

- single-cell RNA-seq expression matrix (normalized counts)
- cell-to-cluster annotation table mapping cells to clusters (e.g., CM3, CT0a, CT3b)
- ICELLNET manually curated human ligand-receptor interactions database (>1600 interactions; 1669 in v2)

## Outputs

- global ICELLNET communication score per cell-pair (e.g., CM3->CT3b=1527, CM3->CT0a=1123)
- per-interaction score matrix with individual ligand-receptor scores (e.g., CD86/CD28, CCL22/CCR4)
- scores written to CSV (e.g., icellnet_scores.csv)

## How to apply

In R (ICELLNET requires R < 4.1 for v2.0.0, or ICELLNET v2.2.x for R > 4.1), install via install_github('soumelis-lab/ICELLNET', ref='master', subdir='icellnet'). Load the scRNA-seq expression matrix and cell-to-cluster annotations, subset the clusters of interest (e.g., a cDC cluster CM3 and T-cell clusters CT0a/CT3b), and compute per-cluster mean normalized expression to build a gene x cluster average-expression profile matrix. Assemble these profiles into the cell-by-gene input format required by the ICELLNET scoring function, which applies the package's expert-curated ligand-receptor database that introduces logical rules for co-expression of protein subunits of ligand and receptor complexes. Run ICELLNET communication-score quantification for each sender->receiver pair to obtain global scores, then extract individual interaction scores (e.g., CD86/CD28, CD86/CTLA4, PDCD1LG2/PDCD1, CCL22/CCR4, VEGFA/FLT1) from the score matrix and write them to a CSV. Validate by comparing reproduced global and per-interaction scores against published values.

## Related tools

- **ICELLNET** (Core framework: provides the expert-curated ligand-receptor database, communication-score quantification, and visualization of cell-cell communication from scRNA-seq profiles) — https://github.com/soumelis-lab/ICELLNET
- **devtools** (R package used to install ICELLNET from GitHub via install_github)
- **data.table** (R dependency for loading and manipulating the expression matrix and cluster annotations)
- **R** (Runtime environment; ICELLNET v2.0.0 requires R < 4.1, use ICELLNET v2.2.x for R > 4.1.x)

## Examples

```
library(devtools); install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet"); library(icellnet)
```

## Evaluation signals

- Reproduced global communication score for CM3->CT3b equals 1527 and CM3->CT0a equals 1123
- Per-interaction scores match published values: CD86/CD28 92 vs 40, CD86/CTLA4 92 vs 14, PDCD1LG2/PDCD1 72 vs 19, CCL22/CCR4 100 vs 39, VEGFA/FLT1 21 vs 0
- Output CSV contains both global and per-interaction scores for every queried pair with no missing values for the database-curated interactions
- Input matrix passed to the scoring function is in the required cell-by-gene (cluster-mean profile) format compatible with the ICELLNET database gene symbols

## Limitations

- ICELLNET includes a relatively small number of interactions compared to other databases; at the original publication, chemokine and checkpoint interactions were not yet included in the standard distribution
- Downstream effectors are not integrated; approaches integrating downstream signaling pathways can lead to unpredictable false positive and false negative predictions
- ICELLNET v2.0.0 requires R version < 4.1; for R > 4.1.x use ICELLNET v2.2.x
- Ligand-receptor database predictions are not experimentally validated for every interaction; some receptors lack known ligands (DR6/TNFRSF21, RELT, TROY, NGFR) and some ligands lack known receptors (IL17D)

## Evidence

- [readme] manually curated ligand-receptor interactions database: "manually curated ligand-receptor interactions database: curated exclusively from human studies, ICELLNET database takes into account the multiple subunits of ligand and receptor complexes (more than"
- [readme] ICELLNET install command: "install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet")"
- [methods] R version constraint: "ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x."
- [discussion] multi-subunit handling: "Similar to CellPhoneDB10, ICELLNET takes into account the multiple subunits of ligands and receptors, by introducing logical rules for co-expression of protein subunits."
- [methods] database size in v2: "New structure and extension of ICELLNET database up to 1669 interactions."
- [methods] scRNAseq tutorial pointer: "ICELLNET analysis on scRNAseq data: [Tutorial](https://github.com/soumelis-lab/ICELLNET/blob/master/exemples_tutorials/Exemple2_scRNAseq.md)"
- [discussion] downstream effectors limitation: "Due to these limitations, we have decided not to integrate downstream effectors in ICELLNET, at this stage."
