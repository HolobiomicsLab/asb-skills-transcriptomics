---
name: mutual-nearest-neighbor-pair-identification
description: Use when combining scRNA-seq matrices from different batches (e.g., GSE70630
  MGH53 & MGH54) and you need to detect which of the ~50 PCA subspaces carry batch
  effect before removal.
when_to_use_negative:
- Data has not been quality-controlled (low-quality cells/genes not removed) — the
  README requires basic QC before BEER
- Input is gene-centric scaled data rather than raw counts (UMI, FPKM, RPKM, TPM,
  PKM), which BEER explicitly does not accept
- Cells of a given type exist in only one batch with no cross-batch analogues, so
  mutual nearest pairs cannot anchor correlation estimates
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BEER
  role: Implements MN pair identification and PC subspace scoring via the BEER() call
    with GNUM/PCNUM/ROUND/GN/SEED parameters
  repo: https://github.com/jumphone/BEER
- name: Seurat
  role: Runs PCA (RunPCA) over 2000 variable features whose 50 subspaces are scored;
    supplies the Seurat object in mybeer$seurat
  repo: https://github.com/jumphone/BEER
- name: ComBat
  role: Optional expression adjustment of PCs (COMBAT=TRUE / BEER.combat) after subspace
    selection
  repo: https://github.com/jumphone/BEER
- name: .simple_combine
  role: Merges the per-batch matrices into a single combined DATA matrix with matched
    genes before pair identification
  repo: https://github.com/jumphone/BEER
- name: .selectUSE
  role: Refines mybeer$select with cutoffs CUTR=0.7, CUTL=0.7, RR=0.5, RL=0.5
  repo: https://github.com/jumphone/BEER
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41421-019-0114-x
    title: A novel approach to remove the batch effect of single-cell data
  source_content_hash: sha256:493c194ed7bbfeedd3fa213482e5dcb1d68504d3034da471746229bcc6156a33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jumphone/BEER
  tools:
  - .selectUSE
  - .simple_combine
  - BEER
  - ComBat
  - R >=3.5
  - Seurat
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# mutual-nearest-neighbor-pair-identification

## Summary

Identifies mutual nearest (MN) cell pairs across batches within grouped cells, then scores each PCA subspace by rank and linear correlation of MN pair positions to flag sub-spaces with latent high batch effect, as implemented in BEER.

## When to use

Use when combining scRNA-seq matrices from different batches (e.g., GSE70630 MGH53 & MGH54) and you need to detect which of the ~50 PCA subspaces carry batch effect before removal.

## When NOT to use

- Data has not been quality-controlled (low-quality cells/genes not removed) — the README requires basic QC before BEER
- Input is gene-centric scaled data rather than raw counts (UMI, FPKM, RPKM, TPM, PKM), which BEER explicitly does not accept
- Cells of a given type exist in only one batch with no cross-batch analogues, so mutual nearest pairs cannot anchor correlation estimates

## Inputs

- Gene-by-cell UMI/TPM expression matrices per batch (e.g., DATA1_MAT.txt, DATA2_MAT.txt from demodata.zip) with gene rownames and deduplicated cell colnames
- BATCH character vector labeling each cell's batch of origin, length = ncol(DATA)
- BEER parameters: GNUM=30, PCNUM=50, ROUND=1, GN=2000, SEED=1, RMG=NULL

## Outputs

- mybeer$select: indices (within 1-50) of batch-effect-free PCA subspaces to retain (PCUSE)
- Per-PC rank correlation and linear correlation values (mybeer$cor, mybeer$lcor) for the correlation scatter plot
- pcuse.txt containing only selected PC indices within 1-50

## How to apply

Merge batch matrices with .simple_combine (optionally FILL=TRUE to keep one-condition genes), build a BATCH character vector, and run PCA (Seurat) over GN=2000 variable genes to compute PCNUM=50 subspaces. Group cells into GNUM=30 groups per batch (SEED=1, ROUND=1), then within each group identify mutual nearest cell pairs across batches. Evaluate each PCA subspace using the rank correlation and linear correlation of MN pair positions — poor correlation indicates a latent high batch-effect subspace. Select PCUSE from mybeer$select (optionally refined with .selectUSE using CUTR=0.7, CUTL=0.7, RR=0.5, RL=0.5), keeping PCA embeddings unchanged so results remain trackable and explainable.

## Related tools

- **BEER** (Implements MN pair identification and PC subspace scoring via the BEER() call with GNUM/PCNUM/ROUND/GN/SEED parameters) — https://github.com/jumphone/BEER
- **Seurat** (Runs PCA (RunPCA) over 2000 variable features whose 50 subspaces are scored; supplies the Seurat object in mybeer$seurat) — https://github.com/jumphone/BEER
- **ComBat** (Optional expression adjustment of PCs (COMBAT=TRUE / BEER.combat) after subspace selection) — https://github.com/jumphone/BEER
- **.simple_combine** (Merges the per-batch matrices into a single combined DATA matrix with matched genes before pair identification) — https://github.com/jumphone/BEER
- **.selectUSE** (Refines mybeer$select with cutoffs CUTR=0.7, CUTL=0.7, RR=0.5, RL=0.5) — https://github.com/jumphone/BEER

## Examples

```
mybeer=BEER(DATA, BATCH, GNUM=30, PCNUM=50, ROUND=1, GN=2000, SEED=1, COMBAT=TRUE, RMG=NULL); PCUSE=mybeer$select
```

## Evaluation signals

- The emitted PCUSE index set must exactly equal mybeer$select from BEER(DATA, BATCH, GNUM=30, PCNUM=50, ROUND=1, GN=2000, SEED=1, COMBAT=TRUE, RMG=NULL) on the demo data
- pcuse.txt exists and contains only indices within 1-50
- Plot of mybeer$cor vs mybeer$lcor shows selected PCs (red) in the high-correlation region and removed PCs with poor correlation (xlim 0-1, ylim 0-1)
- PCA embedding values are unchanged by selection — no PC values are modified, only which dims are passed downstream (e.g., dims = PCUSE in RunUMAP)
- BATCH vector length equals ncol(DATA) and each cell carries exactly one D1/D2 label

## Limitations

- MN-pair correlation flags subspaces with poor cross-batch correlation, but sub-spaces carrying real biology shared by only one batch may be confounded; users should inspect the cor/lcor scatter distribution before trusting defaults
- BEER automatically adjusts GNUM when cell number is small in some batch (v0.1.6), which can change group structure for small datasets
- .simple_combine default FILL=FALSE drops genes expressed in only one condition, affecting the PCA space being scored
- Compatibility differs across Seurat versions: V0.1.9 is the first version for Seurat 4.0.0; V0.1.8 is the last for Seurat 3.0.0

## Evidence

- [abstract] MN pair identification rationale: [evidence span withheld — anchor status: not_found]
- [abstract] No value modification finding: "By using a cell-type imbalanced benchmark, we show that BEER has a clear advantage over four representative batch-effect removal tools: Combat, Seurat (CCA alignment), fastMNN, and BBKNN"
- [methods] BEER invocation: "mybeer=BEER(DATA, BATCH, GNUM=30, PCNUM=50, ROUND=1, GN=2000, SEED=1, COMBAT=TRUE, RMG=NULL)"
- [methods] .selectUSE cutoffs: "# PCUSE=.selectUSE(mybeer, CUTR=0.7, CUTL=0.7, RR=0.5, RL=0.5)"
- [readme] Scatter plot of correlations: "plot(mybeer$cor,mybeer$lcor,pch=16,col=COL,
        xlab='Rank Correlation',ylab='Linear Correlation',xlim=c(0,1),ylim=c(0,1))"
