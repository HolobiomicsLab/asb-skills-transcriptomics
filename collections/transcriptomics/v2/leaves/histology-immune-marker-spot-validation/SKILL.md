---
name: histology-immune-marker-spot-validation
description: Use when Splane-predicted spatial domains (e.g., D1–D9 across 11 breast
  cancer Visium slices) need orthogonal validation against histology (H&E immune spots)
  or marker staining (CD3+ IF) and copy-number evidence.
when_to_use_negative:
- When spatial domains were not yet identified by Splane — deconvolution (Spoint deconv_spatial)
  and domain identification must precede validation
- When no matched histology (H&E) or marker staining (CD3 IF) data exist for the slices,
  making spot-level immune validation impossible
- When analyzing non-seq-based ST data at single-cell resolution where deconvolution
  with Spoint is unnecessary
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: Splane
  role: Produces the spatial domain assignments (D1–D9) that this skill validates
    via copy-number and immune-staining evidence
  repo: github.com/QuKunLab/SPACEL
- name: Spoint
  role: Upstream deconvolution (SpointModel.deconv_spatial) providing per-spot cell
    type composition that Splane requires before domain identification
  repo: github.com/QuKunLab/SPACEL
- name: SPACEL
  role: Toolkit supplying Spoint and Splane modules and the breast cancer Visium tutorial
    underlying this workflow
  repo: github.com/QuKunLab/SPACEL
- name: Python
  role: Implementation language for the SPACEL pipeline and the validation report
    generation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:38bd4fe4721fe766664691afbf1cb59cd1ee53dcef755f7beaf6e67ddc252e42
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SPACEL
  - Splane
  - Spoint
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# histology-immune-marker-spot-validation

## Summary

Validates computationally predicted spatial domains (from Splane on 10X Visium breast cancer slices) against orthogonal evidence: inferCNV copy-number profiles flag tumor domains (1q/8q gains, 1p losses), while H&E-marked immune spots and CD3+ immunofluorescence quantify immune-domain purity.

## When to use

Use when Splane-predicted spatial domains (e.g., D1–D9 across 11 breast cancer Visium slices) need orthogonal validation against histology (H&E immune spots) or marker staining (CD3+ IF) and copy-number evidence.

## When NOT to use

- When spatial domains were not yet identified by Splane — deconvolution (Spoint deconv_spatial) and domain identification must precede validation
- When no matched histology (H&E) or marker staining (CD3 IF) data exist for the slices, making spot-level immune validation impossible
- When analyzing non-seq-based ST data at single-cell resolution where deconvolution with Spoint is unnecessary

## Inputs

- Splane spatial domain assignments (D1–D9) per spot across the 11 breast cancer 10X Visium slices
- H&E-stained histology images with marked immune regions for slices S1, S2, S5, S6
- CD3 immunofluorescence marker staining for slice S10
- Gene expression matrices per Visium slice for inferCNV copy-number inference

## Outputs

- Per-domain inferCNV copy-number profiles with tumor flags (1q/8q gains >= 0.2, 1p losses <= -0.2)
- Per-domain H&E immune-spot percentages (e.g., D9: 44%, D8: 8%, D7: 6%)
- Per-domain CD3+ IF spot percentages (e.g., ~72% in D9, slice S10)
- validation_report.csv with domain assignments, copy-number calls, and immune-spot percentages

## How to apply

After SplaneModel.identify_spatial_domain assigns uniform domains D1–D9 across the 11 breast cancer Visium slices, compute inferCNV copy-number profiles per domain and flag tumor domains when 1q/8q gains are >= 0.2 and 1p losses are <= -0.2. For candidate immune domains (D7–D9), intersect domain assignments with H&E histology annotations in slices S1, S2, S5, and S6, computing the percentage of H&E-marked immune spots per domain; expect ~44% H&E+ spots in D9 versus <1% in intermediate and tumor domains. Independently, quantify the percentage of CD3+ IF-marked spots per domain in slice S10 (expected ~72% in D9). Write the domain assignments, copy-number calls, and immune-spot percentages to validation_report.csv; a domain is correctly validated when immune-spot fractions are markedly enriched relative to non-immune domains and copy-number signatures are concordant with tumor-domain labels.

## Related tools

- **Splane** (Produces the spatial domain assignments (D1–D9) that this skill validates via copy-number and immune-staining evidence) — github.com/QuKunLab/SPACEL
- **Spoint** (Upstream deconvolution (SpointModel.deconv_spatial) providing per-spot cell type composition that Splane requires before domain identification) — github.com/QuKunLab/SPACEL
- **SPACEL** (Toolkit supplying Spoint and Splane modules and the breast cancer Visium tutorial underlying this workflow) — github.com/QuKunLab/SPACEL
- **Python** (Implementation language for the SPACEL pipeline and the validation report generation)

## Examples

```
from SPACEL import Splane; Splane.utils.add_cell_type_composition(adata_list); SplaneModel.train(adata_list, epochs=50); SplaneModel.identify_spatial_domain(adata_list)
```

## Evaluation signals

- Immune domains D9, D8, and D7 contain 44%, 8%, and 6% H&E-marked immune spots respectively in slices S1/S2/S5/S6, versus <1% in intermediate and tumor domains
- CD3+ IF-marked spots reach ~72% in domain D9 of slice S10
- Tumor domains show 1q/8q copy-number gains >= 0.2 and 1p losses <= -0.2 in inferCNV profiles
- validation_report.csv contains per-domain assignments, copy-number calls, and immune-spot percentages without missing entries
- Domain labels are uniform (D1–D9) and transcriptomically/spatially coherent across all 11 slices

## Limitations

- Validation requires matched H&E histology annotations and CD3 IF staining on specific slices (S1/S2/S5/S6 for H&E, S10 for CD3); absent these, immune-domain verification is impossible
- inferCNV-based tumor flags depend on threshold choices (1q/8q gains >= 0.2, 1p losses <= -0.2) that may not generalize to tumors with different karyotypes

## Evidence

- [full_text] Copy-number validation thresholds: [evidence span withheld — anchor status: not_found]
- [results] H&E immune-spot enrichment finding: [evidence span withheld — anchor status: not_found]
- [abstract] Splane domain identification method: "Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify spatial domains"
- [discussion] Spoint prerequisite for Visium data: "one has to employ a cell type deconvolution algorithm such as Spoint to obtain the cell type composition information before Splane can be applied"
- [readme] SPACEL tutorial for breast cancer Visium data: "Splane tutorial: Identify uniform spatial domain on human breast cancer Visium dataset"
