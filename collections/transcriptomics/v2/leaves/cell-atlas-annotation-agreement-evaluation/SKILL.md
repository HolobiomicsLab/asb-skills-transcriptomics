---
name: cell-atlas-annotation-agreement-evaluation
description: Use when a cell-atlas scRNA-Seq dataset carries multiple conflicting
  annotation columns (e.g., ICGS2, Seurat3, Monocle3, curated labels) in adata.obs
  and you need to quantify whether an integrated solution reproduces or refines the
  reference atlas clusters.
when_to_use_negative:
- The dataset has only a single clustering annotation and no reference label set to
  compare against — there is nothing to mix-and-match or evaluate agreement on
- You need a modality-specific joint embedding or geometric graph integration (e.g.,
  Seurat-WNN, TotalVI) rather than decision-level reconciliation of existing label
  assignments
- The AnnData already holds a final, trusted curated annotation and the goal is only
  label transfer, not integration or agreement benchmarking
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scTriangulate
  role: Produces the reconciled mix-and-match cluster assignment (lazy_run with win_fraction_cutoff
    and add_metrics settings) whose agreement with the reference atlas is evaluated
  repo: https://github.com/frankligy/scTriangulate
- name: scTriangulate.preprocessing.add_annotation
  role: Adds the retrieved alternative annotations (ICGS2, Monocle3, Seurat3, curated)
    as obs columns to the AnnData before integration
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy
  role: Hosts the AnnData object, log-CPTT normalization, and UMAP generation (scTriangulate.preprocessing.scanpy_recipe)
    feeding the integration
- name: ClusterEnsembles
  role: Python package used with default options in the methods for consensus-based
    comparison against the scTriangulate decision-level solution
  repo: https://github.com/tsano430/ClusterEnsembles
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:fa9956dedcf4ef47d20bbbd1124a8f90db8f582954a126bdfe7d1f62ca8c30a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - ClusterEnsembles
  - scTriangulate
  - scTriangulate.preprocessing.add_annotation
  - scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cell-atlas-annotation-agreement-evaluation

## Summary

Evaluate how well an integrated single-cell clustering solution (e.g., from scTriangulate) agrees with a curated reference cell-atlas annotation, such as the Hay et al. human bone marrow atlas, by tabulating cluster-level agreement between the mix-and-match assignment and each contributing or reference label set.

## When to use

Use when a cell-atlas scRNA-Seq dataset carries multiple conflicting annotation columns (e.g., ICGS2, Seurat3, Monocle3, curated labels) in adata.obs and you need to quantify whether an integrated solution reproduces or refines the reference atlas clusters.

## When NOT to use

- The dataset has only a single clustering annotation and no reference label set to compare against — there is nothing to mix-and-match or evaluate agreement on
- You need a modality-specific joint embedding or geometric graph integration (e.g., Seurat-WNN, TotalVI) rather than decision-level reconciliation of existing label assignments
- The AnnData already holds a final, trusted curated annotation and the goal is only label transfer, not integration or agreement benchmarking

## Inputs

- AnnData object (adata.X log-CPTT normalized, adata.obs columns of conflicting annotations, adata.obsm['X_umap']; no adata.raw attribute)
- Alternative clustering annotations (ICGS2, Monocle3, Seurat3) and curated labels from Synapse syn26320732
- Curated reference atlas cluster labels (Hay et al. bone marrow reference clusters)

## Outputs

- Mix-and-match cluster assignment per cell (final scTriangulate annotation column)
- Agreement table of scTriangulate solution versus Hay et al. reference clusters (Supplementary Fig. 4d pattern)
- UMAP visualization of reconciled clusters showing resolved transitional states (HSC-cycle, MEP, LMPP)

## How to apply

Retrieve the alternative clustering results (ICGS2, Monocle3, Seurat3) and curated annotations from the Synapse deposit (syn26320732), then attach them to the existing AnnData object with scTriangulate.preprocessing.add_annotation and add_umap, along with the reference UMAP coordinates. Normalize raw UMI counts to log CPTT (log counts per ten thousand) before running scTriangulate on the processed AnnData with win_fraction_cutoff = 0.3 and add_metrics = 'tfidf5' so TF-IDF stability scoring resolves transitional states (e.g., HSC-cycle, MEP, LMPP) that a single clustering view misses. After the mix-and-match assignment is produced, build an agreement table cross-tabulating the scTriangulate clusters against the curated Hay et al. reference clusters to reproduce the Supplementary Fig. 4d comparison pattern. Rationale: scTriangulate integrates at the decision level using Shapley values from coalitional game theory, so agreement with a curated atlas serves as the biological validation of the stability-pruned, reconciled solution.

## Related tools

- **scTriangulate** (Produces the reconciled mix-and-match cluster assignment (lazy_run with win_fraction_cutoff and add_metrics settings) whose agreement with the reference atlas is evaluated) — https://github.com/frankligy/scTriangulate
- **scTriangulate.preprocessing.add_annotation** (Adds the retrieved alternative annotations (ICGS2, Monocle3, Seurat3, curated) as obs columns to the AnnData before integration) — https://github.com/frankligy/scTriangulate
- **scanpy** (Hosts the AnnData object, log-CPTT normalization, and UMAP generation (scTriangulate.preprocessing.scanpy_recipe) feeding the integration)
- **ClusterEnsembles** (Python package used with default options in the methods for consensus-based comparison against the scTriangulate decision-level solution) — https://github.com/tsano430/ClusterEnsembles

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['ICGS2','Monocle3','Seurat3','curated']); sctri.lazy_run(win_fraction_cutoff=0.3, add_metrics='tfidf5')
```

## Evaluation signals

- The scTriangulate run completes on all input annotation sets (ICGS2, Monocle3, Seurat3, curated) without dropping any query column
- The agreement table against Hay et al. reference clusters reproduces the pattern reported in Supplementary Fig. 4d
- Transitional states HSC-cycle, MEP, and LMPP are resolved as distinct populations (unlike a single TF-IDF score analysis)
- Cells with winning fraction below win_fraction_cutoff (0.3 here; default 0.25 in the paper) are flagged as unstable, and final clusters fall below the minimum cluster size threshold (abs_thresh default 10) are pruned
- Input AnnData is log-CPTT normalized and has no adata.raw attribute, since raw data interferes with scanpy marker-gene calculation

## Limitations

- Integration of cluster predictions from dozens of sources significantly increases runtime; Shapley selection is automatically replaced with rank-based selection when more than 15 annotation sets are provided
- Programmatic default analytical settings (e.g., win_fraction_cutoff, add_metrics choices) may not be optimal for all study designs and must be justified per dataset
- Population naming of the reconciled clusters remains largely non-standardized, so agreement evaluation depends on a curated reference mapping

## Evidence

- [methods] The obtained annotations and UMAP coordinates24 were added to the existing AnnData by the scTriangulate.preprocessing.add_annotation and scTriangulate.preprocessing.add_umap functions: "The obtained annotations and UMAP coordinates24 were added to the existing AnnData by the scTriangulate.preprocessing.add_annotation and scTriangulate.preprocessing.add_umap functions"
- [methods] Four embedded stability scores: "scTriangulate uses four embedded scores: the reassign score, the TFIDF10 score, the TFIDF5 score, and the SCCAF score."
- [methods] Synapse deposit of inputs: "using the Python package ClusterEnsembles with default options (https://github.com/tsano430/ClusterEnsembles)."
- [readme] Decision-level reconciliation: "scTriangulate is a Python package to mix-and-match conflicting clustering results in single cell analysis, and generate reconciled clustering solutions."
- [abstract] Coalitional stability rationale: "Rather than ensemble approaches which select the "consensus", scTriangulate picks the most stable solution through coalitional iteration."
- [discussion] Reference-classification integration scope: "the integration of diverse cellular annotations from independent studies (i.e., reference-classification), multimodal measurements, distinct clustering algorithms, and software settings"
