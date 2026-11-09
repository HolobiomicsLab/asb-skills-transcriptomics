---
name: meta-component-projection-scrna-seq
description: Use when annotating cell types/states in tumor immune microenvironment
  scRNA-seq data at higher resolution and consistency than marker-based references
  (CIBERSORT, Azimuth, Human Primary Cell Atlas) allow.
when_to_use_negative:
- Input is not tumor microenvironment single-cell data (MeCs were learnt from TME
  scRNA-seq and may not transfer to non-TME tissues)
- The goal is joint dataset integration of a new cohort rather than projection/annotation
  — the paper notes Harmony and scArches are the relevant integration algorithms,
  while MetaTiME maps onto a fixed pretrained MeC space
- A bespoke retrained latent space is required for a new domain — use the MetaTiME
  re-training pipeline (MetaTiMEpretrain) rather than direct annotator projection
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: MetaTiME
  role: 'Core toolkit: loaddata normalization, mecmapper/plmapper projection onto
    86 MeCs, and annotator for cell-state labels'
  repo: https://github.com/yi-zhang/MetaTiME
- name: CIBERSORT
  role: Baseline reference-based cell type definition used to benchmark resolution
    of MetaTiME annotations (14 cell types)
- name: SingleR
  role: Alternative automated annotator benchmarked with Human Primary Cell Atlas
    and Blueprint/ENCODE reference panels
- name: scanpy
  role: Dependency for loading/processing anndata objects during normalization and
    projection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:4f989881fdf9879aa2cf466b0c931248a697820216d86d0431881dc1fd21ab17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - CIBERSORT
  - MetaTiME
  - SingleR
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# meta-component-projection-scrna-seq

## Summary

Project new tumor microenvironment scRNA-seq data onto MetaTiME's 86 pretrained, data-driven meta-components (MeCs) to automatically annotate fine-grained cell states and score signature continuums, replacing inconsistent manual marker-based labeling.

## When to use

Use when annotating cell types/states in tumor immune microenvironment scRNA-seq data at higher resolution and consistency than marker-based references (CIBERSORT, Azimuth, Human Primary Cell Atlas) allow.

## When NOT to use

- Input is not tumor microenvironment single-cell data (MeCs were learnt from TME scRNA-seq and may not transfer to non-TME tissues)
- The goal is joint dataset integration of a new cohort rather than projection/annotation — the paper notes Harmony and scArches are the relevant integration algorithms, while MetaTiME maps onto a fixed pretrained MeC space
- A bespoke retrained latent space is required for a new domain — use the MetaTiME re-training pipeline (MetaTiMEpretrain) rather than direct annotator projection

## Inputs

- Tumor microenvironment scRNA-seq raw count matrix (e.g., TISCH-processed Yost et al. BCC count matrix)
- Pretrained MetaTiME MeC model from Zenodo DOI 10.5281/zenodo.7410180
- Optional: baseline annotations (Seurat CIBERSORT-derived labels, SingleR labels against Human Primary Cell Atlas and Blueprint/ENCODE) for comparison

## Outputs

- Per-cell MeC-based cell-state annotation table (e.g., yost_bcc_cell_annotations.csv)
- Cross-tabulation of MetaTiME states vs baseline annotations (comparison table)
- Signature continuum scores per cell

## How to apply

Install the metatime Python package (pip install metatime; tested with pandas==1.1.5, scanpy==1.8.2, anndata==0.8.0, leidenalg==0.8.3, harmonypy). Load and normalize the raw count matrix (library-size normalization with log1p transform) via metatime.loaddata, then project cells onto the 86 data-driven meta-component space using metatime.mecmapper/plmapper with the pretrained MeC model downloaded from Zenodo DOI 10.5281/zenodo.7410180. Run the metatime.annotator module to assign a MeC-derived cell-state label to every cell (e.g., recovering 38 cell states on held-out Yost et al. BCC cells vs 14 types from Seurat CIBERSORT-based annotation). The rationale is that the MeCs were learnt via large-scale pan-cancer ICA integration of millions of TME single cells, so they encode reproducible, independent gene-expression components that transfer across cancer types. Optionally score signature continuums and test differential signature activity using the annotator notebook workflow.

## Related tools

- **MetaTiME** (Core toolkit: loaddata normalization, mecmapper/plmapper projection onto 86 MeCs, and annotator for cell-state labels) — https://github.com/yi-zhang/MetaTiME
- **CIBERSORT** (Baseline reference-based cell type definition used to benchmark resolution of MetaTiME annotations (14 cell types))
- **SingleR** (Alternative automated annotator benchmarked with Human Primary Cell Atlas and Blueprint/ENCODE reference panels)
- **scanpy** (Dependency for loading/processing anndata objects during normalization and projection)

## Examples

```
pip install metatime; then in Python: import metatime; adata = metatime.loaddata.load_and_normalize(counts); mec_scores = metatime.mecmapper.plmapper(adata, model='zenodo.7410180'); labels = metatime.annotator.annotate(mec_scores)
```

## Evaluation signals

- Annotation table contains a MeC-derived state label for every input cell (row count matches input cell count)
- Number of recovered cell states exceeds baseline resolution — e.g., 38 MetaTiME states vs 14 Seurat CIBERSORT-based types on Yost et al. BCC cells
- Cells project successfully into all 86 MeC dimensions with finite scores after log1p-normalized input
- Cross-tabulation of MetaTiME states vs CIBERSORT/SingleR labels shows lineage-consistent concordance with finer granularity
- Package versions match tested dependency set (scanpy==1.8.2, anndata==0.8.0) to avoid API mismatches

## Limitations

- MeCs were learnt from tumor immune microenvironment data; transfer outside the TME domain is untested
- MetaTiME projects onto a fixed pretrained latent space and does not infer joint representations across datasets the way Harmony or scArches do
- Some biological interpretations rest on external artifacts not in the text (e.g., TF annotations in Supplementary Data 2) and on ChIP-seq inferred from other cell lines (e.g., GR ChIP-seq not available in relevant T cell states)

## Evidence

- [readme] MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment.: "MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells"
- [intro] These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities.: "These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities."
- [abstract] By projecting onto the MetaTiME space, we provide a tool to annotate cell states and signature continuums for TME scRNA-seq data.: "By projecting onto the MetaTiME space, we provide a tool to annotate cell states and signature continuums for TME scRNA-seq data."
- [discussion] performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51."
- [other] Use MetaTiME to automatically annotate cell states and map signatures: " - [Use MetaTiME to automatically annotate cell states and map signatures](https://github.com/yi-zhang/MetaTiME/blob/main/docs/notebooks/metatime_annotator.ipynb)"
