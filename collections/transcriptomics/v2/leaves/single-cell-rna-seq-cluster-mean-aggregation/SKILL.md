---
name: single-cell-rna-seq-cluster-mean-aggregation
description: Use when you have a scRNA-seq expression matrix with cell-to-cluster
  annotations and need pseudo-bulk per-cluster average profiles as input to a transcriptome-based
  cell-cell communication framework.
when_to_use_negative:
- Input is already a bulk or sorted cell-population transcriptome profile, where cluster
  aggregation is unnecessary (use ICELLNET's sorted-population workflow instead).
- You need single-cell resolution of communicating cell pairs or spatial proximity
  information; ICELLNET infers communication from transcriptomic profiles and does
  not integrate spatial or downstream-effectors signals.
- Data are non-human; the ICELLNET database is curated exclusively from human studies.
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: ICELLNET
  role: Computes communication scores from cluster-mean profiles using its expert-curated
    ligand-receptor database and multi-subunit complex rules
  repo: https://github.com/soumelis-lab/ICELLNET
- name: R
  role: Environment for subsetting clusters, computing per-cluster mean expression,
    and running ICELLNET
- name: devtools
  role: Installs the icellnet package from GitHub (install_github with ref='master',
    subdir='icellnet')
- name: data.table
  role: Dependency used for fast reading and manipulation of expression and annotation
    tables
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:a06486789fee83e3e198f2b18b29edccb2036a9667b1f0f9c997a24daa1b7484
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
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-rna-seq-cluster-mean-aggregation

## Summary

Aggregates single-cell RNA-seq expression into per-cluster mean expression profiles (gene x cluster matrix) so that cell-type-level communication methods such as ICELLNET can compute ligand-receptor communication scores between defined cell populations (e.g., CM3 cDC vs CT0a/CT3b T-cell clusters in lupus nephritis).

## When to use

Use when you have a scRNA-seq expression matrix with cell-to-cluster annotations and need pseudo-bulk per-cluster average profiles as input to a transcriptome-based cell-cell communication framework.

## When NOT to use

- Input is already a bulk or sorted cell-population transcriptome profile, where cluster aggregation is unnecessary (use ICELLNET's sorted-population workflow instead).
- You need single-cell resolution of communicating cell pairs or spatial proximity information; ICELLNET infers communication from transcriptomic profiles and does not integrate spatial or downstream-effectors signals.
- Data are non-human; the ICELLNET database is curated exclusively from human studies.

## Inputs

- single-cell RNA-seq normalized expression matrix (genes x cells)
- cell-to-cluster annotation table (e.g., CM3 cDC, CT0a effector memory CD4+ T cells, CT3b TFH-like cells)
- ICELLNET curated ligand-receptor interactions database (>1600 interactions)

## Outputs

- gene x cluster mean-expression profile matrix in ICELLNET cell-by-gene input format
- global ICELLNET communication scores per sender-receiver pair (e.g., CM3->CT3b = 1527, CM3->CT0a = 1123)
- per-interaction score matrix (e.g., CD86/CD28, CD86/CTLA4, PDCD1LG2/PDCD1, CCL22/CCR4, VEGFA/FLT1)
- icellnet_scores.csv

## How to apply

Load the scRNA-seq expression matrix and cell-to-cluster annotations in R, then subset cells belonging to the clusters of interest (e.g., CM3 cDC, CT0a and CT3b T cells) and compute per-cluster mean normalized expression for every gene, producing a gene x cluster average-expression profile matrix. Assemble the cluster-mean profiles into the cell-by-gene input format required by the ICELLNET scoring function (install_github('soumelis-lab/ICELLNET', ref='master', subdir='icellnet')), which applies its manually curated human ligand-receptor database (>1600 interactions) that accounts for multi-subunit ligand and receptor complexes via co-expression logical rules. Run the communication-score quantification for each sender-receiver pair, then extract global and per-interaction scores (e.g., CD86/CD28, CCL22/CCR4) from the score matrix for export (icellnet_scores.csv). Rationale: averaging normalized expression within clusters gives stable transcriptomic signatures per population, which ICELLNET scores against curated interaction families to quantify communication potential.

## Related tools

- **ICELLNET** (Computes communication scores from cluster-mean profiles using its expert-curated ligand-receptor database and multi-subunit complex rules) — https://github.com/soumelis-lab/ICELLNET
- **R** (Environment for subsetting clusters, computing per-cluster mean expression, and running ICELLNET)
- **devtools** (Installs the icellnet package from GitHub (install_github with ref='master', subdir='icellnet'))
- **data.table** (Dependency used for fast reading and manipulation of expression and annotation tables)

## Examples

```
library(devtools); install_github("soumelis-lab/ICELLNET",ref="master",subdir="icellnet"); profiles <- sapply(clusters, function(cl) rowMeans(norm[, anno$cluster==cl])); cc_score <- icellnet(CC.genes=LR.db, CCI=LR.db, family=LR.families, data=profiles, target=0, source=1)
```

## Evaluation signals

- Reproduced global ICELLNET scores match published values: CM3->CT3b = 1527 and CM3->CT0a = 1123.
- Individual interaction scores match published values: CD86/CD28 92 vs 40, CD86/CTLA4 92 vs 14, PDCD1LG2/PDCD1 72 vs 19, CCL22/CCR4 100 vs 39, VEGFA/FLT1 21 vs 0.
- The aggregated profile matrix is in cell-by-gene (cluster-profile) format with one mean column per selected cluster and no unclustered cells included.
- Output CSV icellnet_scores.csv contains both global pair scores and per-interaction scores from the score matrix.
- ICELLNET database version used contains >1600 manually curated human interactions (v2 extends to 1669).

## Limitations

- ICELLNET v2.0.0 requires R version < 4.1; for R > 4.1.x use ICELLNET v2.2.x.
- The curated database includes a relatively small number of interactions compared to other databases, and chemokine and checkpoint interactions have been added over time rather than being initially exhaustive.
- ICELLNET does not integrate downstream effectors, and does not inherently model spatially proximal versus distal signaling sources.
- Scores depend on cluster annotation quality and per-cluster mean expression; rare subpopulations may be masked by averaging.

## Evidence

- [methods] Subset cells belonging to the CM3 (cDC), CT0a and CT3b clusters and compute per-cluster mean normalized expression for every gene, producing a gene x cluster average-expression profile matrix.: [evidence span withheld — anchor status: not_found]
- [readme] install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet"): "install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet")"
- [readme] curated exclusively from human studies, ICELLNET database takes into account the multiple subunits of ligand and receptor complexes (more than 1600 interactions): "curated exclusively from **human** studies, ICELLNET database takes into account the **multiple** subunits of ligand and receptor complexes (more than 1600 interactions)"
- [discussion] Similar to CellPhoneDB, ICELLNET takes into account the multiple subunits of ligands and receptors, by introducing logical rules for co-expression of protein subunits.: [evidence span withheld — anchor status: not_found]
- [methods] ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x.: "ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x."
