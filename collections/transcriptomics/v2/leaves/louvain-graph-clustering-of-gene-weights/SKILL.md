---
name: louvain-graph-clustering-of-gene-weights
description: Use when you have cohort-shared ICA gene z-weight vectors for independent
  components and need to collapse them into consensus, reproducible gene programs
  (meta-components) for downstream TME scRNA-seq annotation.
when_to_use_negative:
- Input is raw single-cell expression data rather than already-computed cohort-shared
  ICA z-weight vectors — the upstream ICA/reproducibility-filter stage must run first.
- Goal is annotating new TME scRNA-seq data with the pretrained MeC space — use MetaTiME-Annotator
  projection instead of re-clustering.
- Components are already known to be cohort-specific (fewer than 3 cohorts with matching
  components) — reproducibility filtering would discard them and clustering would
  be invalid.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MetaTiME
  role: Framework within which the reproducibility filtering, similarity graph construction,
    Louvain clustering, and z-weight averaging are performed to derive MeCs
  repo: github.com/yi-zhang/MetaTiME
- name: leidenalg
  role: Graph community-detection dependency of the MetaTiME package used for clustering
    components on the similarity graph
  repo: github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:00fe578fbaa2f74997717fe9c66d2237d0242cc30550fe4d514e5ca1f645b180
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaTiME
  - leidenalg
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# louvain-graph-clustering-of-gene-weights

## Summary

Clusters cohort-shared ICA independent components (ICs) into consensus meta-components (MeCs) by building an IC-similarity graph over gene z-weight vectors and running Louvain community detection at resolution 1.25, then averaging z-weights within each community. This is how MetaTiME derives its 86 reproducible, interpretable tumor microenvironment transcriptional directions.

## When to use

Use when you have cohort-shared ICA gene z-weight vectors for independent components and need to collapse them into consensus, reproducible gene programs (meta-components) for downstream TME scRNA-seq annotation.

## When NOT to use

- Input is raw single-cell expression data rather than already-computed cohort-shared ICA z-weight vectors — the upstream ICA/reproducibility-filter stage must run first.
- Goal is annotating new TME scRNA-seq data with the pretrained MeC space — use MetaTiME-Annotator projection instead of re-clustering.
- Components are already known to be cohort-specific (fewer than 3 cohorts with matching components) — reproducibility filtering would discard them and clustering would be invalid.

## Inputs

- Cohort-shared ICA gene z-weight matrix (task_inputs port 'ica_zweight_matrix'), one vector of gene z-weights per independent component (IC)

## Outputs

- mec_profiles.csv: MeC gene-weight profile table (one row per MeC, gene z-weight columns)
- MeC-to-community assignment from Louvain clustering at resolution 1.25

## How to apply

First filter ICs for reproducibility, retaining only those whose gene z-weight profiles match components in at least 3 datasets (minimum Pearson correlation criterion across cohorts). Then compute pairwise cosine distances between the gene z-weight vectors of the retained ICs to define component-to-component similarity, and build a similarity graph over them. Run Louvain community detection on this graph at resolution 1.25, discarding communities smaller than 5 components. Average the gene z-weights of all ICs within each Louvain cluster to define each meta-component (MeC) profile, and write one row per MeC (gene z-weight columns) to mec_profiles.csv. The rationale is that communities of similar ICs represent shared transcriptional programs across cohorts rather than cohort-specific variation.

## Related tools

- **MetaTiME** (Framework within which the reproducibility filtering, similarity graph construction, Louvain clustering, and z-weight averaging are performed to derive MeCs) — github.com/yi-zhang/MetaTiME
- **leidenalg** (Graph community-detection dependency of the MetaTiME package used for clustering components on the similarity graph) — github.com/yi-zhang/MetaTiME

## Examples

```
import networkx as nx, scipy.spatial.distance as d; G = nx.from_numpy_array(1 - d.cdist(Z, Z, metric='cosine')); comms = nx.community.louvain_communities(G, resolution=1.25)
```

## Evaluation signals

- Number of derived MeCs reproduces the scale of the published meta-component space (86 interpretable meta-components reported for MetaTiME).
- Each output row in mec_profiles.csv contains a defined MeC profile over the gene z-weight features (no missing/empty profiles).
- All ICs feeding clustering passed the reproducibility filter: minimum Pearson correlation with matching components in at least 3 datasets.
- No Louvain cluster smaller than 5 components is retained in the final MeC set.
- Community detection parameters match the specification: similarity graph from cosine distance between z-weight vectors, Louvain resolution 1.25.

## Limitations

- The reproducibility filter depends on matching components across a sufficient number of datasets (at least 3); cohorts without matching components contribute nothing to the MeC space.
- Louvain resolution (1.25) and minimum cluster size (5 components) are fixed choices; other resolutions may partition the graph differently.
- GR/other TF ChIP-seq support for downstream MeC interpretation is only available from other cell lines, not in the relevant T cell state contexts.

## Evidence

- [other] Build a similarity graph over the retained ICs and run Louvain community detection at resolution 1.25, discarding clusters smaller than 5 components.: [evidence span withheld — anchor status: not_found]
- [intro] These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities.: "These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities."
- [discussion] performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components"
- [readme] MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data.: "MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data."
