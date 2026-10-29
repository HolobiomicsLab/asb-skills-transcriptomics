---
name: wgcna-co-expression-module-detection
description: Use when input is a processed single-cell expression matrix with cell-level
  metadata and the goal is to infer co-expression modules and identify which modules
  track a trait such as patient survival status.
when_to_use_negative:
- Input is bulk RNA-seq or an already-aggregated bulk expression matrix rather than
  single-cell data requiring pseudo-bulk aggregation across neighboring cells.
- The goal is transcription-factor regulatory network (GRN) reconstruction or drug–gene
  interaction repurposing rather than co-expression module detection and module–trait
  correlation.
- The network is already inferred and modules already detected — use downstream skills
  for hub detection or GRN/drug analysis instead of re-running module detection.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCANet
  role: All-in-one Python package that implements pseudo-bulk aggregation, GCN inference,
    module detection, and module–trait association from single-cell data
  repo: https://github.com/oubounyt/SCANet
- name: WGCNA
  role: Modified version used by SCANet for network inference and module detection
    from the gene–gene correlation matrix
- name: BioNERO
  role: Framework used to construct the pairwise gene–gene correlation matrix (Pearson/Spearman/biweight
    midcorrelation) feeding the WGCNA GCN inference
- name: Python
  role: Implementation language; SCANet is distributed as a Python package on PyPI
    (supports Python 3.6+)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad644
    title: Inference of differential key regulatory networks and mechanistic drug
      repurposing candidates from scRNA-seq data with SCANet
  source_content_hash: sha256:62461048f5fb16782cf9c6d1774cb25140d7722a04bd91242908980cb7108173
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oubounyt/SCANet
  tools:
  - BioNERO
  - Python
  - SCANet
  - WGCNA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# wgcna-co-expression-module-detection

## Summary

Detects trait- and cell-type-specific co-expression gene modules from scRNA-seq data by building a signed gene co-expression network (GCN) with a modified WGCNA implementation inside SCANet, then correlating detected modules against sample-level trait metadata (e.g. patient outcome).

## When to use

Use when input is a processed single-cell expression matrix with cell-level metadata and the goal is to infer co-expression modules and identify which modules track a trait such as patient survival status.

## When NOT to use

- Input is bulk RNA-seq or an already-aggregated bulk expression matrix rather than single-cell data requiring pseudo-bulk aggregation across neighboring cells.
- The goal is transcription-factor regulatory network (GRN) reconstruction or drug–gene interaction repurposing rather than co-expression module detection and module–trait correlation.
- The network is already inferred and modules already detected — use downstream skills for hub detection or GRN/drug analysis instead of re-running module detection.

## Inputs

- Processed scRNA-seq expression matrix with cell-level metadata (cell type, patient outcome), e.g. the Cillo et al. (2021) GSE180578 processed PBMC data
- Cell-type annotation enabling subsetting of a target population (e.g. classical monocytes)
- Sample/trait metadata for module–trait association (healthy, deceased, surviving)

## Outputs

- Signed gene co-expression network (GCN) adjacency matrix
- Co-expression gene modules with gene counts per module (expected: 27 modules, 30–195 genes each for the Cillo classical-monocyte case)
- Module–trait correlation table linking each module to patient-outcome classes (e.g. M11 vs deceased, M21 vs surviving)
- Hub gene list (top 10% most connected genes per module)

## How to apply

Subset the annotated cell type of interest (e.g. classical monocytes from the Cillo et al. 2021 PBMC dataset, GSE180578) and create pseudo-bulk profiles by aggregating read counts for each gene across a designated number of neighboring cells, yielding representative per-sub-cluster cells that reduce single-cell sparsity. Within the BioNERO framework, compute the pairwise gene–gene correlation matrix (Pearson's r, Spearman's, or biweight midcorrelation) and apply SCANet's modified WGCNA (Langfelder and Horvath 2008) to convert correlations into an adjacency matrix and infer a signed GCN. Check that the module graph fits the scale-free topology, a characteristic of real-world biological networks. Detect co-expression modules; for the Cillo classical-monocyte case the expected result is 27 modules of 30–195 genes per module. Correlate the detected modules against the trait metadata (healthy, deceased, surviving) to produce a module–trait correlation table, and optionally identify network hubs as the intersection of the top 10% most connected genes in each module.

## Related tools

- **SCANet** (All-in-one Python package that implements pseudo-bulk aggregation, GCN inference, module detection, and module–trait association from single-cell data) — https://github.com/oubounyt/SCANet
- **WGCNA** (Modified version used by SCANet for network inference and module detection from the gene–gene correlation matrix)
- **BioNERO** (Framework used to construct the pairwise gene–gene correlation matrix (Pearson/Spearman/biweight midcorrelation) feeding the WGCNA GCN inference)
- **Python** (Implementation language; SCANet is distributed as a Python package on PyPI (supports Python 3.6+))

## Examples

```
pip install scanet && python -c "import scanet as sn"
```

## Evaluation signals

- The run yields 27 co-expression modules of classical monocytes with 30–195 genes per module.
- Module–trait correlation table shows M11 with the highest positive correlation to deceased patients and M21 highly correlated to surviving patients.
- The module graph passes the scale-free topology check, a characteristic of real-world biological networks.
- Within M11, 16 out of 90 genes exhibit correlation ≥ 0.9, and over half of genes and hubs code for heat shock proteins (e.g. HESPB1, DNAJA4, CRYAB).
- Hub identification returns the intersection of the top 10% of the most connected genes in the module.

## Limitations

- GCNs rely on correlation, not causation, potentially leading to false positives in co-expression analysis.
- Correlation-based module detection may miss subtle relationships; converting GCNs into GRNs may additionally increase the risk of false negatives.
- SCANet officially supports Python 3.6+; the full worked example in the repository notebook uses the 3k PBMCs from 10x Genomics rather than the Cillo dataset.

## Evidence

- [intro] SCANet builds on a modified version of weighted gene co-expression network analysis (WGCNA) (Langfelder and Horvath 2008) for network inference and module detection: "SCANet builds on a modified version of weighted gene co-expression network analysis (WGCNA) (Langfelder and Horvath 2008) for network inference and module detection"
- [results] We detected 27 co-expression gene modules of classical monocytes among healthy, deceased, and surviving patients (Fig. 2B) with 30–195 genes per module (M) (Fig. 2C): "We detected 27 co-expression gene modules of classical monocytes among healthy, deceased, and surviving patients (Fig. 2B) with 30–195 genes per module (M) (Fig. 2C)"
- [results] In particular, M11 had the highest positive correlation with deceased patients, while M21 had a high correlation with surviving patients: "In particular, M11 had the highest positive correlation with deceased patients, while M21 had a high correlation with surviving patients"
- [methods] They can also check whether the module graph fits the scale-free topology, a characteristic of real-world biological networks: "They can also check whether the module graph fits the scale-free topology, a characteristic of real-world biological networks"
- [intro] This is achieved by aggregating the read counts for each gene across a designated number of neighboring cells, reducing the data: "This is achieved by aggregating the read counts for each gene across a designated number of neighboring cells, reducing the data"
- [readme] SCANet, a python package that incorporates the inference of gene co-expression networks from single-cell gene expression data: "`SCANet`, a python package that incorporates the inference of gene co-expression networks from single-cell gene expression data"
