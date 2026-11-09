---
name: marker-centroid-reference-mapping
description: Use when aligning independent lung atlas annotations (GSE161383, GSE171524,
  GSE136831, EGAS00001004344) onto a base scRNA/snRNA dataset to feed competing reference
  classifications into scTriangulate.
when_to_use_negative:
- When the goal is a data-level integration via a joint latent space or geometric
  graph (e.g. Seurat-WNN, TotalVI) rather than decision-level label reconciliation
  — scTriangulate deliberately makes no assumption about the distributions of raw
  underlying values.
- When annotations are already concordant with no conflicting cluster-label assignments,
  as mix-and-match reconciliation of competing references adds no value.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: cellHarmony
  role: 'Two-step label transfer: MarkerFinder top-60 marker centroids plus self-reclassification
    of the base dataset'
- name: scTriangulate
  role: Decision-level integration of transferred reference annotations via Shapley
    value; add_annotation/add_umap attach inputs, lazy_run-like ScTriangulate produces
    the final solution
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy
  role: Normalization of raw UMI counts to log CPTT and preprocessing of the base
    AnnData
- name: scikit-learn
  role: Homogeneity, Completeness, and V-Measure computation of the triangulated solution
    against HLCA and LungMAP references
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:0911c716dc51ba70e711f7dec22acfeb39f9e5b168af081b37585b259a1e52bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - cellHarmony
  - scTriangulate
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# marker-centroid-reference-mapping

## Summary

A two-step label-transfer strategy for mapping lung cell-atlas annotations (Melms, Adams, Travaglini, Wang; HLCA; LungMAP CellRef) onto a base snRNA-Seq dataset using MarkerFinder top-60 marker centroids with Pearson correlation, as preparation for scTriangulate decision-level integration.

## When to use

Use when aligning independent lung atlas annotations (GSE161383, GSE171524, GSE136831, EGAS00001004344) onto a base scRNA/snRNA dataset to feed competing reference classifications into scTriangulate.

## When NOT to use

- When the goal is a data-level integration via a joint latent space or geometric graph (e.g. Seurat-WNN, TotalVI) rather than decision-level label reconciliation — scTriangulate deliberately makes no assumption about the distributions of raw underlying values.
- When annotations are already concordant with no conflicting cluster-label assignments, as mix-and-match reconciliation of competing references adds no value.

## Inputs

- Cell-population assignment and raw UMI count data from four independent lung studies (GSE161383 Melms, GSE171524 Adams, GSE136831 Travaglini, EGAS00001004344 Wang, 9-donor snRNA-Seq as base)
- HLCA reference annotations (Azimuth human-lung-v2) and LungMAP CellRef 1.0 reference assignments
- MNP-Verse mononuclear phagocyte centroids from healthy controls for myeloid comparison
- Base AnnData object with raw UMI counts (to be normalized to log CPTT)

## Outputs

- Per-cell transferred annotations in the base AnnData from each of the four studies plus HLCA and CellRef references
- UMAP coordinates attached via add_umap
- Triangulated (scTriangulate) consensus cell-population assignment
- lung_atlas_agreement_metrics.csv with Homogeneity, Completeness, and V-Measure against HLCA and LungMAP references

## How to apply

Select the Wang et al. 9-donor snRNA-Seq dataset (EGAS00001004344) as the base and run cellHarmony's two-step strategy: first compute MarkerFinder top-60 marker centroids from non-diseased controls of each reference atlas, transferring labels to the base cells only where Pearson correlation rho > 0.2; second, self-reclassify the base dataset to provide its own annotation solution. Apply the identical two-step mapping to the HLCA (Azimuth human-lung-v2) and LungMAP CellRef 1.0 references, and compute MNP-Verse mononuclear-phagocyte centroids from healthy controls for the myeloid comparison. Attach all transferred annotations and UMAP coordinates to the base AnnData via scTriangulate.preprocessing.add_annotation and add_umap, then run scTriangulate (win_fraction_cutoff=0.4, add_metrics='tfidf5') so these reference-derived clusters are mix-and-matched with the base self-annotation. Rationale: scTriangulate integrates clustering results at the decision level, so prior reference-classification labels from supervised tools like cellHarmony can be jointly considered without a joint latent space; agreement of the final triangulated solution with independent references is then quantified by Homogeneity, Completeness, and V-Measure against HLCA and LungMAP assignments (sklearn conventions), including subdivided dendritic-cell and macrophage populations, and written to lung_atlas_agreement_metrics.csv.

## Related tools

- **cellHarmony** (Two-step label transfer: MarkerFinder top-60 marker centroids plus self-reclassification of the base dataset)
- **scTriangulate** (Decision-level integration of transferred reference annotations via Shapley value; add_annotation/add_umap attach inputs, lazy_run-like ScTriangulate produces the final solution) — https://github.com/frankligy/scTriangulate
- **scanpy** (Normalization of raw UMI counts to log CPTT and preprocessing of the base AnnData)
- **scikit-learn** (Homogeneity, Completeness, and V-Measure computation of the triangulated solution against HLCA and LungMAP references)

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['wang_self_reclass','melms_mapped','adams_mapped','travaglini_mapped','hlca_mapped','cellref_mapped']); sctri.lazy_run(win_fraction_cutoff=0.4, add_metrics='tfidf5')
```

## Evaluation signals

- V-Measure values in lung_atlas_agreement_metrics.csv must be internally consistent with their harmonic-mean definition: V = 2*h*c/(h+c)
- All three agreement metrics (Homogeneity, Completeness, V-Measure) must lie in [0,1]
- scTriangulate's agreement scores must exceed the ensemble-clustering baseline, confirming the reported performance relationship
- Only label transfers with Pearson correlation rho > 0.2 against MarkerFinder top-60 centroids should be accepted
- win_fraction_cutoff = 0.4 and add_metrics = 'tfidf5' must be set (scTriangulate 0.9.0, scanpy 1.7.2), matching the reported Fig. 5g configuration

## Limitations

- Dendritic-cell and macrophage (myeloid) populations require the subdivided MNP-Verse centroids from healthy controls for reliable comparison.
- Programmatic default analytical settings (e.g. win_fraction_cutoff) may not be optimal for all study designs.
- Integrating cluster predictions from dozens of sources significantly increases runtime.

## Evidence

- [methods] two-step marker centroid mapping: [evidence span withheld — anchor status: not_found]
- [methods] annotation attachment to AnnData: "added to the existing AnnData by the scTriangulate.preprocessing.add_annotation and scTriangulate.preprocessing.add_umap functions"
- [intro] decision-level integration: [evidence span withheld — anchor status: not_found]
- [readme] reference atlases as inputs: "Integrate results from different reference atlases."
- [results] agreement metrics: [evidence span withheld — anchor status: not_found]
