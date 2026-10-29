---
name: per-spot-module-expression-scoring
description: Use when you have a merged, edge-pruned SCING GRN divided into leiden
  subnetworks and need quantitative module activity scores per spatial spot or cell
  to support regional-specificity tests, clustering, or phenotype/plaque correlation
  analyses.
when_to_use_negative:
- When no gene modules/subnetworks have been defined yet — AUCell scoring requires
  a merged SCING GRN already partitioned with leiden; network construction and edge
  pruning must come first.
- When the analysis needs the GRN itself to use spatial coordinates during inference
  — SCING does not explicitly utilize spatial information in network construction,
  so this skill scores modules post hoc rather than building spatially aware networks.
- When the goal is causal/directional regulatory inference — SCING links are not necessarily
  causal or directional, so module scores should not be interpreted as causal pathway
  activity.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: SCING
  role: Provides the merged, pruned GRN whose leiden subnetworks are the gene modules
    scored per spot; the ModuleBasedDimensionalityReduction.ipynb tutorial performs
    the leiden partitioning and AUCell scoring step.
  repo: github.com/XiaYangLabOrg/SCING
- name: SCENIC
  role: Supplies the AUCell method used to compute module-specific expression for
    each cell/spatial spot; installed via pyscenic.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:da35fae5d1166a2a54d2cc4609f0256263dc5f21c500063a716933b02c7ead7a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCENIC
  - SCING
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# per-spot-module-expression-scoring

## Summary

Score module-specific expression for every spatial transcriptomics spot (or single cell) by combining SCING's leiden-partitioned GRN subnetworks with the AUCell method from SCENIC, producing per-spot module activity scores that can be spatially mapped and correlated with phenotypes such as amyloid beta plaque burden.

## When to use

Use when you have a merged, edge-pruned SCING GRN divided into leiden subnetworks and need quantitative module activity scores per spatial spot or cell to support regional-specificity tests, clustering, or phenotype/plaque correlation analyses.

## When NOT to use

- When no gene modules/subnetworks have been defined yet — AUCell scoring requires a merged SCING GRN already partitioned with leiden; network construction and edge pruning must come first.
- When the analysis needs the GRN itself to use spatial coordinates during inference — SCING does not explicitly utilize spatial information in network construction, so this skill scores modules post hoc rather than building spatially aware networks.
- When the goal is causal/directional regulatory inference — SCING links are not necessarily causal or directional, so module scores should not be interpreted as causal pathway activity.

## Inputs

- Merged, edge-pruned SCING GRN partitioned into leiden subnetworks/modules
- Visium spatial transcriptomics count matrix with per-spot metadata (e.g., GSE152506 mouse AD vs WT, ages 3-18 months, beta-amyloid plaque staining annotations)
- pyscenic installed in the scing conda environment (for AUCell)

## Outputs

- Per-spot (or per-cell) AUCell module-specific expression score matrix (spots x modules)
- Spatial maps of module expression (e.g., Figure 6 visualizations of module spatial maps and correlation significance)
- Per-module statistics: regional specificity t-test/ANOVA results and Pearson correlations with plaque burden and age vs 1,000 random gene-set nulls (Bonferroni corrected)

## How to apply

First build supercells, infer ~100 GRNs from subsamples with SCING's BuildNetwork workflow, merge them into a final GRN, prune edges, and partition it into subnetworks (33 in the mouse AD Visium case) using the leiden graph partitioning algorithm. Then combine these gene-module subnetworks with the AUCell method from SCENIC to compute, for each spatial spot (or cell), a module-specific expression score per subnetwork. These per-spot AUCell module scores become the values used for downstream analyses: t-tests/ANOVA across annotated brain subregions for regional specificity, and Pearson correlation of each module's expression with plaque burden and age against a null of 1,000 random gene sets with Bonferroni correction. The scores are also usable for clustering and phenotypic association, and modules can be biologically annotated via enrichr pathway enrichment on the genes in each subnetwork.

## Related tools

- **SCING** (Provides the merged, pruned GRN whose leiden subnetworks are the gene modules scored per spot; the ModuleBasedDimensionalityReduction.ipynb tutorial performs the leiden partitioning and AUCell scoring step.) — github.com/XiaYangLabOrg/SCING
- **SCENIC** (Supplies the AUCell method used to compute module-specific expression for each cell/spatial spot; installed via pyscenic.)

## Examples

```
conda activate scing; pip install pyscenic; jupyter notebook tutorials/ModuleBasedDimensionalityReduction.ipynb  # leiden partition merged GRN, then AUCell module scores per spot
```

## Evaluation signals

- The AUCell score matrix has one row per spatial spot and one column per leiden subnetwork (e.g., 33 modules in the mouse AD Visium analysis) with scores bounded in AUCell's expected [0,1]-style ranking range.
- Module scores reproduce the expected spatial pattern when plotted per spot — e.g., plaque-associated modules (such as module 25, enriched for microglial activation, lysosome, and cell migration) show high scores in plaque-rich regions.
- At least some modules show significant Pearson correlation with plaque burden against a null of 1,000 random gene sets after Bonferroni correction (six significant modules in the source task).
- Regional specificity: module expression differs significantly across annotated brain subregions under t-tests/ANOVA.
- Each scored module's gene membership matches its leiden subnetwork and is biologically interpretable via enrichr pathway enrichment.

## Limitations

- SCING does not explicitly utilize the spatial information of spatial transcriptomics during GRN construction, even though resulting GRNs show spatial patterns; leveraging spatial information is deferred to future work.
- Many hyperparameters require selection and selection can be dataset dependent, despite shown robustness across a range of hyperparameters.
- Inferred GRN links are not necessarily causal or directional, so module scores are correlational activity measures, not causal pathway states.

## Evidence

- [methods] AUCell method combined with leiden subnetworks for per-spot module scores: "We combine these subnetworks with the AUCell method to get module specific expression for each cell"
- [methods] Leiden partitioning identifies GRN subnetworks (modules): "We apply the leiden graph partitioning algorithm to identify GRN subnetworks"
- [intro] Modules enable clustering, phenotypic association, and biological annotation: "SCING GRNs are then partitioned into modules, to compute module specific expression for each cell. These modules can be used for clustering, phenotypic association, and biological annotation through"
- [readme] README tutorial covering leiden partitioning plus AUCell module scoring: "ModuleBasedDimensionalityReduction.ipynb: Take the merged network, and divide it into subnetworks with the leiden graph partitioning algorithm, finally run the AUCell approach"
- [discussion] Spatial information not used in network construction: "the spatial information of spatial transcriptomics is not explicitly utilized in SCING GRN construction despite the fact that the resulting GRNs show spatial patterns"
