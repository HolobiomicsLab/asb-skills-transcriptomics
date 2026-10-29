---
name: gene-functional-annotation
description: Use when a ranked disease-vs-control gene list (e.g. top-100 or top-10
  biomarkers from scaLR's eval_and_analysis_pipeline) must be checked for enrichment
  of known disease or neurological-disorder genes to validate biomarker discovery.
when_to_use_negative:
- Input is raw scRNA-seq counts or an unranked gene set — annotation should follow
  scaLR's feature scoring and top-K selection, not precede it
- No curated disease reference gene set exists for the phenotype of interest (e.g.
  CellMarker2.0 lacks relevant biomarker sets), making overlap counting uninformative
- The goal is cell type annotation rather than disease biomarker validation — scaLR's
  cellMeSH/CellAssign/scCATCH/SCINA/SCSA/scSorter/scType annotation tools are the
  appropriate route
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: scaLR
  role: Platform that ranks disease-associated genes via DNN training, linear_scorer/shap_scorer
    feature scoring, and abs_mean/classwise_abs top-K selection to produce the biomarker
    gene lists that are then functionally annotated
  repo: https://github.com/infocusp/scaLR
- name: CellMarker2.0
  role: Source of cell-specific biomarker reference gene sets (derived from scRNA-seq
    studies) used as the annotation target for top-ranked genes
- name: Python
  role: Implementation language of the scaLR pipeline and the annotation workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:5ebf9380d2b0df920e704d0e015fa12ae2e2084e30dd21a389b0044d7e972bd7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/infocusp/scaLR
  repo_url: https://github.com/infocusp/scaLR
  tools:
  - CellMarker2.0
  - Python
  - scaLR
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gene-functional-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Functionally annotate scaLR's top-ranked disease biomarker genes against curated reference gene sets (CellMarker2.0-derived biomarker sets) to determine how many of the highest-ranked genes are linked to a disease category, as demonstrated for Alzheimer's biomarkers in the HBSFG and HBPFC brain datasets.

## When to use

Use when a ranked disease-vs-control gene list (e.g. top-100 or top-10 biomarkers from scaLR's eval_and_analysis_pipeline) must be checked for enrichment of known disease or neurological-disorder genes to validate biomarker discovery.

## When NOT to use

- Input is raw scRNA-seq counts or an unranked gene set — annotation should follow scaLR's feature scoring and top-K selection, not precede it
- No curated disease reference gene set exists for the phenotype of interest (e.g. CellMarker2.0 lacks relevant biomarker sets), making overlap counting uninformative
- The goal is cell type annotation rather than disease biomarker validation — scaLR's cellMeSH/CellAssign/scCATCH/SCINA/SCSA/scSorter/scType annotation tools are the appropriate route

## Inputs

- Ranked biomarker gene list (top-100 genes exported per disease-vs-control class by scaLR's eval_and_analysis_pipeline)
- Top 10 highest-ranked genes per dataset (e.g. HBSFG, HBPFC)
- CellMarker2.0-derived cell/disease-specific biomarker reference gene sets (scRNA-seq-based)

## Outputs

- Per-dataset annotation table mapping top-ranked genes to disease/neurological-disorder gene references
- Count of top-10 genes linked to neurological disorders per dataset (e.g. 7/10 for HBSFG, 2/10 for HBPFC)
- Functional annotation summary supporting disease-specific biomarker claims

## How to apply

First run scaLR's data ingestion, sample_norm/standard_scale preprocessing, stratified group splitting, DNN training with linear_scorer/shap_scorer scoring, and abs_mean/classwise_abs top-K selection to export a top-100 biomarker gene list for each disease-vs-control class. Then take the 10 highest-ranked genes per dataset and functionally annotate them against neurological-disorder gene references derived from CellMarker2.0 cell-specific biomarkers identified by scRNA-seq studies. Count how many of the top 10 genes are linked to the disease category per dataset and compare across datasets/biomarker sets to judge whether the discovered biomarkers are biologically coherent. The validation pattern from the source task is 7/10 neurological-disorder-linked genes for HBSFG versus 2/10 for HBPFC, showing annotation outcome can differ substantially between brain regions.

## Related tools

- **scaLR** (Platform that ranks disease-associated genes via DNN training, linear_scorer/shap_scorer feature scoring, and abs_mean/classwise_abs top-K selection to produce the biomarker gene lists that are then functionally annotated) — https://github.com/infocusp/scaLR
- **CellMarker2.0** (Source of cell-specific biomarker reference gene sets (derived from scRNA-seq studies) used as the annotation target for top-ranked genes)
- **Python** (Implementation language of the scaLR pipeline and the annotation workflow)

## Examples

```
python pipeline.py --config /path/to/config_clinical.yaml -l -m  # then annotate the top-100 ClasswisePromoters gene list against CellMarker2.0 neurological-disorder biomarkers
```

## Evaluation signals

- Top-100 biomarker gene lists are successfully exported for both datasets (e.g. HBSFG and HBPFC) via the eval_and_analysis_pipeline
- Annotation of the top-10 genes reproduces the reported pattern: 7/10 neurological-disorder-linked genes for HBSFG versus 2/10 for HBPFC
- Each top-ranked gene can be traced to its importance score from linear_scorer/shap_scorer with abs_mean or classwise_abs selection
- Reference gene sets are verifiably CellMarker2.0-derived (scRNA-seq-based cell-specific biomarkers), not ad hoc gene lists

## Limitations

- Annotation outcome depends on prior biomarker knowledge in CellMarker2.0; genes absent from the reference will count as unlinked even if biologically relevant
- Feature importance cannot be extracted from several comparison tools (scBalance, scvi, devCellPy, CellPLM), limiting comparability of the ranked gene lists being annotated
- scaLR's performance may vary based on the input data format, requiring users to ensure proper data preprocessing for optimal results

## Evidence
