---
name: single-cell-subpopulation-selection
description: Use when a processed scRNA-seq expression matrix with cell-level metadata
  (cell type, patient outcome) must be narrowed to one cell type before co-expression
  network inference, e.g. isolating classical monocytes from PBMCs.
when_to_use_negative:
- Input is already bulk or pseudo-bulk expression data with no per-cell labels to
  subset
- The analysis goal is cell-type discovery/clustering itself rather than selecting
  a known annotated sub-population for downstream module detection
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCANet
  role: Python package used to load the processed scRNA-seq data, select the classical
    monocyte sub-population, and create pseudo-bulk profiles via aggregating read
    counts across neighboring cells
  repo: https://github.com/oubounyt/SCANet
- name: Python
  role: Language environment for running SCANet subsetting and pseudo-bulk aggregation
    (install via pip install scanet, Python 3.6+)
  repo: https://pypi.org/project/scanet/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:e212c313d50cb0722d749cddbcf43d653ab73c320cbc49df4b6838bfb73179e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oubounyt/SCANet
  tools:
  - BioNERO
  - Python
  - SCANet
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# single-cell-subpopulation-selection

## Summary

Selects a specific cell sub-population (e.g. classical monocytes) from processed scRNA-seq data and divides it into N sub-clusters, averaging expression per sub-cluster to build pseudo-bulk representatives for downstream network inference.

## When to use

Use when a processed scRNA-seq expression matrix with cell-level metadata (cell type, patient outcome) must be narrowed to one cell type before co-expression network inference, e.g. isolating classical monocytes from PBMCs.

## When NOT to use

- Input is already bulk or pseudo-bulk expression data with no per-cell labels to subset
- The analysis goal is cell-type discovery/clustering itself rather than selecting a known annotated sub-population for downstream module detection

## Inputs

- Processed scRNA-seq expression matrix (e.g. Cillo et al. 2021 GSE180578, processed PBMC data)
- Cell-level metadata: cell type annotations and patient outcome labels (healthy, deceased, surviving)

## Outputs

- Subsetted expression matrix restricted to the selected cell sub-population (e.g. classical monocytes)
- Pseudo-bulk RNA-seq representative profile per sub-cluster (one pseudo-cell per sub-cluster)

## How to apply

Load the preprocessed expression matrix and cell-level metadata (cell type, patient outcome) — for example the Cillo et al. (2021) GSE180578 PBMC data — and subset the target population, such as classical monocytes from healthy, deceased, and surviving patients. Divide the selected cells into N sub-clusters and aggregate read counts across neighboring cells to create one representative pseudo-bulk RNA-seq profile per sub-cluster, reducing sparsity and enabling correlation-based network methods. The pseudo-bulked profiles become the input for co-expression network construction (e.g. WGCNA within BioNERO via SCANet). Rationale: single-cell correlation analysis benefits from aggregating expression across neighboring cells before network inference.

## Related tools

- **SCANet** (Python package used to load the processed scRNA-seq data, select the classical monocyte sub-population, and create pseudo-bulk profiles via aggregating read counts across neighboring cells) — https://github.com/oubounyt/SCANet
- **Python** (Language environment for running SCANet subsetting and pseudo-bulk aggregation (install via pip install scanet, Python 3.6+)) — https://pypi.org/project/scanet/

## Examples

```
pip install scanet; import scanet as sn
```

## Evaluation signals

- Selected sub-population matches expected cell-type annotation (classical monocytes) across all three outcome groups (healthy, deceased, surviving)
- Each sub-cluster yields exactly one representative pseudo-bulk profile, with gene read counts aggregated across the designated number of neighboring cells
- The pseudo-bulk matrix retains enough genes per module for downstream detection (modules of 30–195 genes in the Cillo et al. reference run)
- Downstream WGCNA run on the pseudo-bulk data reproduces 27 co-expression modules with M11 highest-positive for deceased and M21 high for surviving patients

## Limitations

- Subsetting depends on the accuracy of pre-existing cell type annotations; misannotated cells propagate into the pseudo-bulk profiles
- The choice of N sub-clusters and neighbors-per-pseudo-bulk affects network structure; the article's reference result (27 modules of 30–195 genes) is specific to those settings

## Evidence

- [results] Selection of classical monocyte sub-population from PBMCs of patients who died or survived acute respiratory illness as well as healthy controls: "Selection of classical monocyte sub-population from PBMCs of patients who died or survived acute respiratory illness as well as healthy controls"
- [intro] Pre-processing single-cell data by creating pseudo-bulk RNA-seq via aggregating read counts across neighboring cells: "This is achieved by aggregating the read counts for each gene across a designated number of neighboring cells, reducing the data"
- [other] Load preprocessed Cillo GSE180578 and subset classical monocytes: [evidence span withheld — anchor status: not_found]
- [supplementary] SCANet is publicly available as a Python package on Github: "SCANet is publicly available as a Python package on Github: https://github.com/oubounyt/SCANet and pypi: https://pypi.org/project/scanet/"
- [readme] SCANet officially supports Python 3.6+: "`SCANet` officially supports Python 3.6+."
