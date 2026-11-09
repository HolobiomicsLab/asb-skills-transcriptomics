---
name: tumor-keratinocyte-colocalization-analysis
description: Use when you have paired SC and ST data from human squamous cell carcinoma
  (e.g. Figshare DOI 10.6084/m9.figshare.24452812) and need to test spatial proximity
  of rare cell types such as TSKs and immune cells at single-cell resolution.
when_to_use_negative:
- SC and ST data come from different donors in the same disease state — the article
  lists this as untested future work
- The goal is spot-level deconvolution into a fixed number of cell types rather than
  single-cell-level spatial mapping; deconvolution treats a spot as a mixture and
  cannot link SC and ST at the single-cell level
- No spot spatial coordinates are available for the ST data, since pseudo-spatial
  assignment via all_coord depends on them
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: STEM
  role: Trains the SOmodel transfer-learning model (PyTorch/CUDA) to produce SC and
    ST embeddings and the SC-ST mapping matrix netsc2st used to assign pseudo-spatial
    coordinates
  repo: https://github.com/WhirlFirst/STEM
- name: scSTEM
  role: PyPI installable package providing the STEM.model and STEM.utils modules (SOmodel,
    seed_all, all_coord) used in the workflow
  repo: https://github.com/WhirlFirst/STEM
- name: PyTorch
  role: Backend for the CUDA-trained SOmodel; embeddings from netE are used to build
    the softmax mapping matrices
- name: scanpy
  role: Normalization (sc.pp.normalize_total, sc.pp.log1p) and QC metrics (sc.pp.calculate_qc_metrics)
    for computing the dropout rate dp
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:dc4795789c2a62975b272c992db179cbfa7ad06b9afec60eee0ded1915d7894a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - PyTorch
  - STEM
  - scSTEM
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4090909090909091
n_claims: 22
---

# tumor-keratinocyte-colocalization-analysis

## Summary

Maps single cells onto spatial transcriptomics spots with STEM to test whether tumor-specific keratinocytes (TSKs) colocalize with endothelial cells at the tumor leading edge and whether rare immune populations (pDCs) concentrate at specific spots, reproducing signature-based hSCC results without manual gene selection.

## When to use

Use when you have paired SC and ST data from human squamous cell carcinoma (e.g. Figshare DOI 10.6084/m9.figshare.24452812) and need to test spatial proximity of rare cell types such as TSKs and immune cells at single-cell resolution.

## When NOT to use

- SC and ST data come from different donors in the same disease state — the article lists this as untested future work
- The goal is spot-level deconvolution into a fixed number of cell types rather than single-cell-level spatial mapping; deconvolution treats a spot as a mixture and cannot link SC and ST at the single-cell level
- No spot spatial coordinates are available for the ST data, since pseudo-spatial assignment via all_coord depends on them

## Inputs

- hSCC donor 2 and 10 single-cell (SC) count matrices with cell-type labels (CSV, Figshare DOI 10.6084/m9.figshare.24452812)
- hSCC spatial transcriptomics (ST) count matrices with spot spatial coordinates (CSV)
- Keratinocyte subtype labels including tumor-specific keratinocytes (TSKs) and tumor cycling KCs
- Immune/pDC spot assignments for marker-gene comparison

## Outputs

- Mapped spatial cell-type table with pseudo-spatial coordinates per single cell (adata.obsm['spatialDA'] from all_coord)
- Row-normalized neighborhood-enrichment matrix among keratinocyte subtypes saved as CSV
- TSK-endothelial colocalization result at the top tumor leading edge (both donors)
- Marker-gene comparison table for BST2, NRP1, JCHAIN between pDC-localized and other spots (P < 0.01)

## How to apply

Load the hSCC donor SC and ST count matrices, cell-type labels, and spot coordinates, then normalize and log-transform both with scanpy (sc.pp.normalize_total, sc.pp.log1p). Compute the dropout rate dp as 1 minus the ratio of median n_genes_by_counts between SC and ST data via sc.pp.calculate_qc_metrics, and configure the STEM SOmodel with sigma equal to half the ST spot adjacent distance, MMD weight alpha=0.8, lr=0.002, training 400 epochs on the full data with seed_all(2022). Extract SC and ST embeddings with netE, build row-normalized softmax mapping matrices netst2sc and netsc2st, and assign each single cell pseudo-spatial coordinates with all_coord applied to netsc2st and the ST spot coordinates. Then compute the row-normalized neighborhood-enrichment matrix among keratinocyte subtypes (TSKs self-aggregating away from tumor cycling KCs), test TSK colocalization with endothelial cells at the top tumor leading edge, and compare pDC marker genes BST2, NRP1, and JCHAIN between pDC-localized and other spots with significance P < 0.01. Rationale: STEM's transfer learning eliminates the domain gap between SC and ST data so rare-cell localization emerges from the learned embeddings rather than manual signature gene selection.

## Related tools

- **STEM** (Trains the SOmodel transfer-learning model (PyTorch/CUDA) to produce SC and ST embeddings and the SC-ST mapping matrix netsc2st used to assign pseudo-spatial coordinates) — https://github.com/WhirlFirst/STEM
- **scSTEM** (PyPI installable package providing the STEM.model and STEM.utils modules (SOmodel, seed_all, all_coord) used in the workflow) — https://github.com/WhirlFirst/STEM
- **PyTorch** (Backend for the CUDA-trained SOmodel; embeddings from netE are used to build the softmax mapping matrices)
- **scanpy** (Normalization (sc.pp.normalize_total, sc.pp.log1p) and QC metrics (sc.pp.calculate_qc_metrics) for computing the dropout rate dp)

## Examples

```
from STEM.model import *; from STEM.utils import *; seed_all(2022); testmodel = SOmodel(opt); testmodel.togpu(); testmodel.train_wholedata(400, torch.tensor(scdata.values).float(), torch.tensor(stdata.values).float(), torch.tensor(spcoor.values).float())
```

## Evaluation signals

- Loss curve from testmodel.train_wholedata(400, ...) converges as shown in the README's expected loss.png figure
- Row-normalized mapping matrices: in netst2sc each spot's probabilities sum to 1, and in netsc2st each cell's probabilities sum to 1
- TSK colocalization with endothelial cells at the top tumor leading edge reproduces in both hSCC donors (2 and 10), matching the original signature-based result without manual gene selection
- Neighborhood-enrichment matrix shows TSKs self-aggregating away from tumor cycling KCs
- pDC marker genes BST2, NRP1, and JCHAIN show significantly higher expression in pDC-localized vs other spots (P < 0.01)

## Limitations

- Applying STEM to SC and ST data from different donors in the same disease state remains untested future work
- Interpretability of which genes drive spatial location requires the separate spatial adjacency-embedding-gene pathfinding for spatially dominant genes (SDGs); using SDGs as drug-target insights is proposed but not demonstrated
- Sigma must be set empirically to half the ST spot adjacent distance; mis-set sigma degrades performance

## Evidence

- [results] Applying STEM to hSCC data verified that tumor-specific keratinocytes (TSKs) were colocalized with endothelial cells at the top tumor leading edge in both donors, reproducing the original study's signature-based result without manual gene selection.: [evidence span withheld — anchor status: not_found]
- [readme] sigma half adjacent distance rationale: "Empirically we found by setting the sigma as half of the ST spot adjacent distance, STEM achieves the best performance."
- [readme] dropout rate calculation: "Next we calculate the ratio between the median total counts value of SC and ST data as the dropout rate."
- [readme] mapping matrix semantics: "In the SC-ST mapping matrix, the probability of one cell to other spots summarizes to 1."
- [abstract] rare cell type localization abstract claim: "We apply STEM to human squamous cell carcinoma and hepatic lobule datasets to uncover the localization of rare cell types"
- [readme] figshare deposit: "The processed data and trained models used for reproducing the results are deposited in [Figshare](https://doi.org/10.6084/m9.figshare.24452812)."
