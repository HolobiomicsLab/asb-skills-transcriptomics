---
name: cross-method-annotation-comparison
description: Use when a pretrained annotation model (e.g., MetaTiME MeCs) must be
  validated against baseline annotators (Seurat CIBERSORT-based labeling, SingleR
  with Human Primary Cell Atlas / Blueprint-ENCODE) on the same held-out scRNA-seq
  cells.
when_to_use_negative:
- Input data are not tumor microenvironment scRNA-seq — the MetaTiME MeC space was
  learnt from millions of TME single cells across cancer types and may not transfer
  outside that domain.
- You only need one annotation method's output, not a head-to-head resolution comparison
  across methods on identical cells.
- Cells have already been manually annotated with curated marker genes and the goal
  is label merging, not benchmarking automated annotators.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MetaTiME
  role: Pretrained meta-component (MeC) model; projects normalized BCC cells onto
    the 86-component MeC space and assigns fine-grained cell-state labels via metatime.annotator
  repo: https://github.com/yi-zhang/MetaTiME
- name: Seurat
  role: Automated reference-labeling baseline using CIBERSORT-derived signatures,
    producing the 14 cell-type comparison labels
- name: CIBERSORT
  role: Reference signature database whose marker definitions drive the Seurat baseline
    annotation
- name: SingleR
  role: Independent reference-based annotator using the Human Primary Cell Atlas and
    Blueprint/ENCODE panels for cross-method comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:a4529549d996594883846fcf69a296c1925b7b303fe728c2d67925f61540fa68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - CIBERSORT
  - MetaTiME
  - Seurat
  - SingleR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cross-method-annotation-comparison

## Summary

Benchmark a learned cell-state annotator (MetaTiME's 86 meta-components) against marker/reference-based automated annotators (Seurat CIBERSORT signatures, SingleR) on identical held-out single cells, quantifying the resolution gap in cell-state labels.

## When to use

Use when a pretrained annotation model (e.g., MetaTiME MeCs) must be validated against baseline annotators (Seurat CIBERSORT-based labeling, SingleR with Human Primary Cell Atlas / Blueprint-ENCODE) on the same held-out scRNA-seq cells.

## When NOT to use

- Input data are not tumor microenvironment scRNA-seq — the MetaTiME MeC space was learnt from millions of TME single cells across cancer types and may not transfer outside that domain.
- You only need one annotation method's output, not a head-to-head resolution comparison across methods on identical cells.
- Cells have already been manually annotated with curated marker genes and the goal is label merging, not benchmarking automated annotators.

## Inputs

- Held-out tumor microenvironment scRNA-seq count matrix (e.g., Yost et al. basal cell carcinoma cells from the TISCH portal)
- Pretrained MetaTiME MeC model from Zenodo DOI 10.5281/zenodo.7410180
- CIBERSORT-derived cell-type signatures for Seurat automated reference labeling
- Human Primary Cell Atlas and Blueprint/ENCODE reference panels for SingleR

## Outputs

- MeC-based cell-state annotation for every cell (38 cell states on the BCC benchmark)
- Baseline annotations: 14 CIBERSORT-marker-based Seurat cell types and SingleR labels per cell
- Cross-tabulation of MetaTiME states vs. CIBERSORT- and SingleR-based labels (comparison table)
- Cell annotation table yost_bcc_cell_annotations.csv

## How to apply

Download a held-out tumor scRNA-seq count matrix (e.g., the Yost et al. BCC data from TISCH) and the pretrained MetaTiME MeC model from Zenodo (DOI 10.5281/zenodo.7410180). Normalize counts with library-size normalization and log1p transform via the metatime.loaddata utilities, then project all cells onto the 86-component MeC space with metatime.mecmapper/plmapper and assign MeC-based cell-state labels with metatime.annotator. On the exact same cells, run Seurat's automated reference labeling using CIBERSORT-derived signatures and SingleR using the Human Primary Cell Atlas and Blueprint/ENCODE reference panels. Cross-tabulate MetaTiME states against each baseline's labels and save the annotation table (yost_bcc_cell_annotations.csv) and comparison tables; the rationale is that marker-gene definitions chosen differently across prior studies limit resolution, so a data-learnt MeC space should recover finer-grained states (38 vs 14 cell types in the benchmark).

## Related tools

- **MetaTiME** (Pretrained meta-component (MeC) model; projects normalized BCC cells onto the 86-component MeC space and assigns fine-grained cell-state labels via metatime.annotator) — https://github.com/yi-zhang/MetaTiME
- **Seurat** (Automated reference-labeling baseline using CIBERSORT-derived signatures, producing the 14 cell-type comparison labels)
- **CIBERSORT** (Reference signature database whose marker definitions drive the Seurat baseline annotation)
- **SingleR** (Independent reference-based annotator using the Human Primary Cell Atlas and Blueprint/ENCODE panels for cross-method comparison)

## Examples

```
pip install metatime && jupyter nbconvert --to notebook --execute docs/notebooks/metatime_annotator.ipynb
```

## Evaluation signals

- MetaTiME annotator recovers 38 distinct cell states on the held-out Yost et al. BCC cells, versus 14 cell types from Seurat's CIBERSORT-marker-based labeling on the identical cells
- Every cell receives a label from each method; cross-tabulation tables of MetaTiME states vs. CIBERSORT and SingleR labels are non-empty and cover the same cell set
- Counts were normalized with library-size normalization plus log1p before MeC projection (consistent preprocessing across compared methods)
- Projection succeeded onto the 86 meta-component space (cells embedded via metatime.mecmapper/plmapper)
- Output files yost_bcc_cell_annotations.csv and the comparison table exist with per-cell label columns from all three annotators

## Limitations

- Prior automated annotators rely on reference databases whose cell-type definitions were 'chosen differently in the respective studies', so baseline labels are study-dependent and may be inconsistently defined.
- MetaTiME's MeC space is trained on tumor microenvironment scRNA-seq; annotation quality outside TME data is not established.
- Resolution counts (e.g., 38 vs 14) are dataset-dependent — running on other tumor types may yield different numbers of states.

## Evidence

- [readme] MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment.: "MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment."
- [abstract] By projecting onto the MetaTiME space, we provide a tool to annotate cell states and signature continuums for TME scRNA-seq data.: "By projecting onto the MetaTiME space, we provide a tool to annotate cell states and signature continuums for TME scRNA-seq data."
- [intro] cell type definitions in reference databases such as CIBERSORT, Azimuth, and Human Primary Cell Atlas: "cell type definitions in reference databases such as CIBERSORT, Azimuth, and Human Primary Cell Atlas"
- [discussion] Cell type definitions in the previous studies were based on representative genes, which were chosen differently in the respective studies.: "Cell type definitions in the previous studies were based on representative genes, which were chosen differently in the respective studies."
