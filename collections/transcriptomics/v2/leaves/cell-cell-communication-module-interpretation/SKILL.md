---
name: cell-cell-communication-module-interpretation
description: Use when you have pre-computed ligand-receptor CCI predictions (e.g.,
  from CellChat on a scRNA-seq dataset such as GSE72056 melanoma) and need to biologically
  interpret which interaction groups/signals characterize a specific sender or receiver
  cell type.
when_to_use_negative:
- The CCI results have not yet been computed — InterCellar builds on pre-computed,
  inferred ligand−receptor interactions and does not itself predict interactions.
- You need to run the CCC prediction itself or compare prediction methods; InterCellar
  is a downstream interactive analysis platform, and only a limited set of input tools
  (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR) is currently supported.
- You require programmatic, scripted analysis rather than interactive exploration
  — InterCellar's module workflow is designed for the Shiny GUI without programming
  skills.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: InterCellar
  role: R/Shiny app used to subset CCI data in the cluster-verse, annotate int-pairs
    in the function-verse, and define/export int-pair modules via UMAP and hierarchical
    clustering
  repo: https://github.com/martaint/InterCellar
- name: CellChat
  role: Upstream tool that generated the predicted CCI data analyzed (melanoma malignant-cell
    outgoing interactions)
  repo: https://github.com/sqjin/CellChat
- name: biomaRt
  role: R package used in the function-verse to query Gene Ontology through Ensembl
    for GO term-gene annotations
- name: graphite
  role: R package used in the function-verse to retrieve pathway term-gene annotations
    from KEGG, Reactome, Biocarta, PID:NCI-Nature, Panther, and PharmGKB
- name: R/Shiny
  role: Implementation platform for the interactive module-definition workflow
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:e0d0debd6847abd87151603406a39e11dfd29889167ede21c329b3d4ed299581
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - CellChat
  - InterCellar
  - R
  - R/Shiny
  - biomaRt
  - graphite
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# cell-cell-communication-module-interpretation

## Summary

Interpret predicted ligand-receptor CCI results by interactively subsetting them to a cell-type/direction viewpoint in InterCellar's cluster-verse, functionally annotating int-pairs in the function-verse, and grouping interactions into int-pair modules (UMAP embedding + hierarchical clustering) that share functional patterns — as demonstrated for malignant-cell outgoing interactions in Tirosh metastatic melanoma.

## When to use

Use when you have pre-computed ligand-receptor CCI predictions (e.g., from CellChat on a scRNA-seq dataset such as GSE72056 melanoma) and need to biologically interpret which interaction groups/signals characterize a specific sender or receiver cell type.

## When NOT to use

- The CCI results have not yet been computed — InterCellar builds on pre-computed, inferred ligand−receptor interactions and does not itself predict interactions.
- You need to run the CCC prediction itself or compare prediction methods; InterCellar is a downstream interactive analysis platform, and only a limited set of input tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR) is currently supported.
- You require programmatic, scripted analysis rather than interactive exploration — InterCellar's module workflow is designed for the Shiny GUI without programming skills.

## Inputs

- Pre-computed ligand-receptor interaction table (CCI dataset) from an InterCellar-supported tool (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR)
- Cell cluster/cell-type annotations for the scRNA-seq dataset (e.g., malignant cells, macrophages, T cells in the GSE72056 melanoma data)
- Enriched int-pair table loaded into the function-verse

## Outputs

- Int-pair module assignment table (interactions grouped into modules by functional similarity, e.g., nine modules for melanoma malignant-cell outgoing interactions)
- Ranked functional term-to-int-pair annotation table (functional_annotation_ranked_terms.csv)
- UMAP embedding and hierarchical-clustering dendrogram of int-pairs
- Downloaded tables and figures via InterCellar's download options

## How to apply

Load the pre-computed CCI dataset (e.g., CellChat output for the Tirosh metastatic melanoma dataset GSE72056) into the InterCellar Shiny app. In the cluster-verse, interactively filter and subset the interactions to the desired viewpoint, e.g., directed outgoing interactions of malignant cells. In the function-verse, annotate enriched interaction pairs using GO (queried via biomaRt through Ensembl) and pathway resources (KEGG, Reactome, Biocarta, PID:NCI-Nature, Panther, PharmGKB via graphite); a functional term is assigned to an int-pair only when all partner genes/complex members are enriched by that term. Run the int-pair module definition, which embeds the annotated interaction pairs via UMAP and groups them by hierarchical clustering into modules of interactions with similar functional patterns. Inspect the dendrogram and modules (in the melanoma case, nine modules were recovered, with module #6 grouping ECM-organization/integrin-signaling interactions from collagen/laminin genes and module #8 containing ANXA1 from malignant cells with FPR exclusively expressed by macrophages). Export the module assignment table and UMAP/hierarchy figures with InterCellar's download options; run with reproducible = TRUE for session-to-session reproducibility.

## Related tools

- **InterCellar** (R/Shiny app used to subset CCI data in the cluster-verse, annotate int-pairs in the function-verse, and define/export int-pair modules via UMAP and hierarchical clustering) — https://github.com/martaint/InterCellar
- **CellChat** (Upstream tool that generated the predicted CCI data analyzed (melanoma malignant-cell outgoing interactions)) — https://github.com/sqjin/CellChat
- **biomaRt** (R package used in the function-verse to query Gene Ontology through Ensembl for GO term-gene annotations)
- **graphite** (R package used in the function-verse to retrieve pathway term-gene annotations from KEGG, Reactome, Biocarta, PID:NCI-Nature, Panther, and PharmGKB)
- **R/Shiny** (Implementation platform for the interactive module-definition workflow)

## Examples

```
library(InterCellar); InterCellar::run_app(reproducible = TRUE)
```

## Evaluation signals

- The module run recovers the expected number of modules (nine for the melanoma malignant-cell directed-outgoing subset).
- Module #6 groups collagen- and laminin-encoding gene interactions with integrin complexes, consistent with ECM organization/integrin signaling.
- Module #8 contains the ANXA1 (malignant cell) & FPR (macrophage) interaction, with FPR exclusively expressed by macrophages.
- Every functional term-to-int-pair assignment satisfies the all-components-enriched rule (term enriched in both partners, including all subunits of multi-subunit complexes).
- The functional_annotation_ranked_terms.csv output exists and contains ranked terms with their annotated int-pair counts; results are reproducible across R sessions when run_app(reproducible = TRUE).

## Limitations

- InterCellar currently supports only a limited set of input tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); extension to other published tools is planned future work.
- Predicted interactions await experimental validation, e.g., by single-molecule fluorescence in situ hybridization or flow cytometry co-occurrence.
- Prediction methods rely on different reference databases and statistical assumptions, so the input CCI predictions shape the resulting modules.

## Evidence

- [intro] int-pair modules defined as groups of interactions with similar functional patterns: "The first enables a focused inspection of cell−cell communication through the analysis of the so-called int-pair modules, which are defined as groups of interactions with similar functional patterns"
- [intro] three universes: cell clusters, genes, biological functions: [evidence span withheld — anchor status: not_found]
- [intro] functional annotation of enriched interactions in the function-verse: "in the so-called function-verse, InterCellar provides multiple resources to perform functional annotation of enriched interactions."
- [readme] reproducible app launch: "InterCellar::run_app( reproducible = TRUE )"
- [readme] interactive analysis without programming: "Every step of the analysis can be performed interactively, thus not requiring any programming skills."
- [discussion] predicted interactions need experimental validation: "we want to stress the importance of further experimental validation of the predicted interactions"
