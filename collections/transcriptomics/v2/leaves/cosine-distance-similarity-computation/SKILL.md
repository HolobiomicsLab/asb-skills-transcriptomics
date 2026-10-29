---
name: cosine-distance-similarity-computation
description: Use when you have cohort-shared ICA gene z-weight profiles for independent
  components and need a component-to-component similarity matrix to support reproducibility
  filtering and graph-based meta-component construction.
when_to_use_negative:
- Input is raw single-cell expression data rather than an ICA-derived gene z-weight
  matrix — first run ICA decomposition and reproducibility pre-processing
- Goal is annotating cell states on a new tumor scRNA-seq dataset — use the pretrained
  MetaTiME MeC space (MetaTiME-Annotator) instead of recomputing component similarity
- Integrating single cells (rather than learned components) across cohorts — Harmony,
  scVI, or scArches-style joint-representation methods are the appropriate alternative
  named in the article
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MetaTiME
  role: Framework in which IC cosine-distance similarity feeds reproducibility filtering
    and Louvain clustering into consensus meta-components (MeCs)
  repo: github.com/yi-zhang/MetaTiME
- name: leidenalg
  role: Community detection dependency used downstream on the IC similarity graph
    (resolution 1.25) to group reproducible ICs into MeCs
  repo: github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:b204d5a51a174ec2849adbcc8a7c5ee00cbb860ea4e6e5d2c8c7bb4aa19080ba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaTiME
  - leidenalg
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cosine-distance-similarity-computation

## Summary

Computes pairwise cosine-distance similarity between independent-component (IC) gene z-weight vectors across tumor single-cell cohorts, as used in MetaTiME's MeC discovery stage to define component-to-component similarity before reproducibility filtering and Louvain clustering.

## When to use

Use when you have cohort-shared ICA gene z-weight profiles for independent components and need a component-to-component similarity matrix to support reproducibility filtering and graph-based meta-component construction.

## When NOT to use

- Input is raw single-cell expression data rather than an ICA-derived gene z-weight matrix — first run ICA decomposition and reproducibility pre-processing
- Goal is annotating cell states on a new tumor scRNA-seq dataset — use the pretrained MetaTiME MeC space (MetaTiME-Annotator) instead of recomputing component similarity
- Integrating single cells (rather than learned components) across cohorts — Harmony, scVI, or scArches-style joint-representation methods are the appropriate alternative named in the article

## Inputs

- Cohort-shared ICA gene z-weight matrix (rows = independent components, columns = gene z-weight features)
- Cohort labels mapping each IC to its source tumor single-cell dataset

## Outputs

- Pairwise cosine-distance similarity matrix over independent components (component-to-component similarity)
- Filtered/reproducible IC set (minimum Pearson correlation with matching components in ≥3 datasets), feeding Louvain clustering into consensus MeCs

## How to apply

Load the cohort-shared ICA gene z-weight matrix (task_inputs port 'ica_zweight_matrix'), where each row is an independent component (IC) and columns are gene z-weights. Compute the pairwise cosine-distance similarity between the gene z-weight vectors of ICs across cohorts to obtain a component-to-component similarity measure. Use this similarity to retain only ICs whose z-weight profiles match components in at least 3 datasets (minimum Pearson correlation ≥3 cohorts), ensuring downstream MeCs reflect shared rather than cohort-specific programs. The similarity matrix then serves as edge weights for a graph over retained ICs on which Louvain community detection (resolution 1.25, discarding clusters smaller than 5 components) is run; gene z-weights of ICs in each cluster are averaged to define each meta-component (MeC) profile.

## Related tools

- **MetaTiME** (Framework in which IC cosine-distance similarity feeds reproducibility filtering and Louvain clustering into consensus meta-components (MeCs)) — github.com/yi-zhang/MetaTiME
- **leidenalg** (Community detection dependency used downstream on the IC similarity graph (resolution 1.25) to group reproducible ICs into MeCs) — github.com/yi-zhang/MetaTiME

## Examples

```
from scipy.spatial.distance import pdist; S = pdist(ica_zweight_matrix, metric='cosine')
```

## Evaluation signals

- Each retained IC has a minimum Pearson correlation with matching components in at least 3 datasets (reproducibility threshold met)
- Number of derived MeCs reproduces the scale of the published meta-component space (~86 interpretable meta-components reported for MetaTiME)
- No Louvain cluster smaller than 5 components is emitted as an MeC (resolution 1.25 clustering)
- Each output MeC profile in mec_profiles.csv contains a defined gene z-weight vector over the full gene feature set (no missing values per row)

## Limitations

- Similarity is computed on ICA gene z-weights, so errors or batch artifacts in the upstream ICA decomposition propagate into the MeC space
- The pipeline is trained on tumor microenvironment (TME) scRNA-seq; transfer to non-TME tissues is outside the article's stated scope

## Evidence

- [intro] MetaTiME identifies reproducible low-dimensional meta-components that reflect independent components of gene expression variation across cohorts and cancer types.: [evidence span withheld — anchor status: not_found]
- [discussion] performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51."
- [intro] These MeCs represent the TME landscape along 86 data-driven transcriptional directions: "These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities."
- [readme] MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells: "MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data."
