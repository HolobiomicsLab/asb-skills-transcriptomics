---
name: highly-variable-gene-selection-comparison
description: Use when building homologous-gene feature spaces for cross-species scRNA-seq
  integration and you need to know which HVGs each homology strategy (one-to-one orthologs
  vs in-paralog-including) selects and how this affects integration scores.
when_to_use_negative:
- The task involves only a single species or within-species batch integration — homology
  mapping and ortholog/paralog strategy choice is irrelevant there
- The species pair has no reliable curated gene orthology annotation, so in-paralog
  inclusion cannot be built on confident sequence-based homologs
- Only whole-body atlas integration with challenging gene homology annotation is needed,
  where SAMap rather than HVG-set comparison is the recommended approach
edam_operation: http://edamontology.org/operation_3934
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: BENGAL
  role: Benchmarking pipeline that builds homology feature spaces, records HVG selections
    per strategy, runs integration algorithms and computes the 9 integration metrics
    plus biology conservation
- name: scVI
  role: Deep generative integration algorithm run on each homology-derived HVG set
    to embed species in a joint latent space
- name: scANVI
  role: Semi-supervised integration algorithm run on each homology-derived HVG set
    to embed species in a joint latent space
- name: SeuratV4
  role: Anchor-based integration algorithm run on each homology-derived HVG set to
    embed species in a joint latent space
  repo: https://github.com/satijalab/seurat-wrappers
- name: SAMap
  role: Alternative cross-species mapping method highlighted for whole-body atlas
    integration with challenging gene homology annotation
  repo: https://github.com/atarashansky/SAMap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:482f2e520a9afcac553df46a5d09c0d268e1c725e80af3091097f6a2990d5872
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satijalab/seurat-wrappers
  tools:
  - BENGAL
  - SAMap
  - SeuratV4
  - scANVI
  - scVI
  edam_operation: http://edamontology.org/operation_3934
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# highly-variable-gene-selection-comparison

## Summary

Compares the sets of highly variable genes (HVGs) selected under different cross-species homology mapping strategies (O2O one-to-one orthologs vs HE/SH strategies retaining in-paralogs) within the BENGAL benchmarking pipeline, to explain how homology choice shapes integration feature spaces.

## When to use

Use when building homologous-gene feature spaces for cross-species scRNA-seq integration and you need to know which HVGs each homology strategy (one-to-one orthologs vs in-paralog-including) selects and how this affects integration scores.

## When NOT to use

- The task involves only a single species or within-species batch integration — homology mapping and ortholog/paralog strategy choice is irrelevant there
- The species pair has no reliable curated gene orthology annotation, so in-paralog inclusion cannot be built on confident sequence-based homologs
- Only whole-body atlas integration with challenging gene homology annotation is needed, where SAMap rather than HVG-set comparison is the recommended approach

## Inputs

- Cross-species single-cell RNA-seq dataset pairs (e.g. Xenopus tropicalis vs Danio rerio embryo atlases) as Seurat RDS objects
- Curated sequence-based gene orthology annotations (one-to-one, one-to-many, many-to-many homologs)
- Cell type ontology annotations (.obo files, e.g. Cell Ontology) processed with scOntoMatch

## Outputs

- homology_strategy_comparison.csv aggregating integrated and biology-conservation scores by homology method
- Per-strategy HVG sets recording which genes each homology strategy selects
- Joint latent embeddings from scVI, scANVI and SeuratV4 per homology-method x algorithm combination

## How to apply

Within BENGAL, load the cross-species task (e.g. Embryo_xt_dr: Xenopus tropicalis vs Danio rerio scRNA-seq) via getSeuratRds, then construct the homologous-gene feature space under each homology strategy — O2O (one-to-one orthologs only) versus HE/SH (strategies retaining one-to-many and many-to-many in-paralogs) — using curated sequence-based orthology annotations. Record which HVG set each strategy selects so that feature-space differences are traceable. Run integration algorithms such as scVI, scANVI and SeuratV4 on each gene set, and score every homology-method × algorithm combination with the 9 established integration metrics plus the new biology conservation metric quantifying maintenance of cell type distinguishability. Aggregate integrated and biology-conservation scores by homology method into homology_strategy_comparison.csv and check the direction of the difference: for evolutionarily distant species pairs, in-paralog-including strategies are expected to outperform O2O.

## Related tools

- **BENGAL** (Benchmarking pipeline that builds homology feature spaces, records HVG selections per strategy, runs integration algorithms and computes the 9 integration metrics plus biology conservation)
- **scVI** (Deep generative integration algorithm run on each homology-derived HVG set to embed species in a joint latent space)
- **scANVI** (Semi-supervised integration algorithm run on each homology-derived HVG set to embed species in a joint latent space)
- **SeuratV4** (Anchor-based integration algorithm run on each homology-derived HVG set to embed species in a joint latent space) — https://github.com/satijalab/seurat-wrappers
- **SAMap** (Alternative cross-species mapping method highlighted for whole-body atlas integration with challenging gene homology annotation) — https://github.com/atarashansky/SAMap

## Examples

```
obj_list = getSeuratRds(metadata = metadata, sep = "\t"); adatas_minimal = ontoMultiMinimal(adatas, ont = ont, anno_col = "cell_ontology_type", onto_id_col = "cell_ontology_id")
```

## Evaluation signals

- homology_strategy_comparison.csv reproduces the reported direction: for the distant Embryo_xt_dr task, in-paralog-including strategies (HE/SH) outperform O2O on the biology-conservation score
- Each homology-method x algorithm combination receives all 9 established integration metrics plus the biology conservation metric
- The recorded HVG set for O2O contains only one-to-one orthologs, while HE/SH sets include one-to-many and many-to-many homologs
- Successful integrations show a balance between species-mixing and biology conservation, consistent with scANVI/scVI/SeuratV4 behaviour reported in the abstract

## Limitations

- Quantitative scoring assumes accurate cell type annotations and confident known cell type homology from literature
- The benchmark is limited to one-to-one cell type mappings; valid non-one-to-one cell type mappings across species are not assessed
- Cross-species integration relies on curated sequence-based gene orthology, ignoring species-specific genes; functional diversification between orthologs may exceed that between paralogs
- Limited by availability of ground-truth datasets, sophisticated computational methods, and specialised assessment metrics for non-one-to-one mappings

## Evidence

- [abstract] For evolutionarily distant species, including in-paralogs is beneficial.: "For evolutionarily distant species, including in-paralogs is beneficial."
- [abstract] 9 metrics plus new biology conservation metric: "using 9 established metrics. We also develop a new biology conservation metric to address the maintenance of cell type distinguishability."
- [intro] BENGAL pipeline development: "we develop the BENchmarking strateGies for cross-species integrAtion of singLe-cell RNA sequencing data (BENGAL) pipeline"
- [other] HVG sets per strategy recorded in workflow: [evidence span withheld — anchor status: not_found]
- [discussion] Scoring assumptions: "Our quantitative scoring system of species mixing and biology conservation is supported by two assumptions: (1) cell type annotations are accurate;"
