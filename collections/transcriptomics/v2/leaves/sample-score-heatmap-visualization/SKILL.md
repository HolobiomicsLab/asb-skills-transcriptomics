---
name: sample-score-heatmap-visualization
description: Use when rRoma.R has produced a ModuleMatrix of sample scores for tested
  gene sets and you need to display per-sample module activity as a clustered heatmap
  annotated with sample-group labels to confirm which modules are shifted or overdispersed.
when_to_use_negative:
- The rRoma.R analysis has not yet been run — there are no ModuleMatrix sample scores
  to plot
- You need per-gene contribution rankings rather than per-sample module activity —
  use GetTopContrib instead
- Input is raw (unnormalized) counts — rRoma requires normalized data (e.g., edgeR
  cpm or DESeq2 vst) before scores are meaningful
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: rROMA
  role: Provides the Plot.Genesets.Samples function used to draw clustered heatmaps
    of module sample scores with group annotations, plus Plot.Genesets.vs.Sampled
    for PC1Median module ranking
  repo: https://github.com/sysbio-curie/rROMA
- name: R
  role: Runtime environment in which rRoma visualization functions are executed
- name: edgeR
  role: cpm normalization of the GSE176121 count matrix upstream of rRoma, ensuring
    scores plotted in the heatmap come from normalized data
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:6f906d5ff12da11ca2ec260af674b1538124b01c6a42fe9c6b537f3ba4f20d65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/rROMA
  tools:
  - DESeq2
  - R
  - edgeR
  - irlba
  - rROMA
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# sample-score-heatmap-visualization

## Summary

Visualize rROMA module (gene set) activity scores per sample as a clustered heatmap using the rRoma Plot.Genesets.Samples function, with sample-group labels (e.g., CF vs non-CF, or culture medium UNC vs SC) passed via GroupInfo to reveal group-level activity shifts.

## When to use

Use when rRoma.R has produced a ModuleMatrix of sample scores for tested gene sets and you need to display per-sample module activity as a clustered heatmap annotated with sample-group labels to confirm which modules are shifted or overdispersed.

## When NOT to use

- The rRoma.R analysis has not yet been run — there are no ModuleMatrix sample scores to plot
- You need per-gene contribution rankings rather than per-sample module activity — use GetTopContrib instead
- Input is raw (unnormalized) counts — rRoma requires normalized data (e.g., edgeR cpm or DESeq2 vst) before scores are meaningful

## Inputs

- rRoma.R output object (rRoma.output with ModuleMatrix of sample scores per module)
- Named sample-group vector (e.g., Type from sample_labels$sample_id; CF vs non-CF, or UNC vs SC medium)
- List of selected modules (shifted: 'ppv Median Exp' <= 0.05; overdispersed: 'ppv L1' <= 0.05 and 'ppv Median Exp' > 0.05)

## Outputs

- Clustered heatmap of module sample scores annotated with GroupInfo labels (saved as PNG)
- Optional PC1Median scatter from Plot.Genesets.vs.Sampled (Plot = 'PC1Median') ranking shifted modules

## How to apply

After running rRoma.R on the normalized expression matrix (e.g., edgeR cpm) and the module list (HALLMARK sets from SelectFromMSIGdb or cell-type signatures from ReadGMTFile), select the modules of interest using significance criteria such as 'ppv Median Exp' <= 0.05 for shifted modules and 'ppv L1' <= 0.05 with 'ppv Median Exp' > 0.05 for overdispersed-but-not-shifted modules. Build a named grouping vector (e.g., Type from sample_labels$Type, names from sample_labels$sample_id) and pass it as GroupInfo to Plot.Genesets.Samples with cluster_cols = TRUE so samples cluster within and across groups. The resulting heatmap of sample scores confirms group-level activity differences, e.g., higher APICAL_SURFACE in healthy controls and higher FATTY_ACID_METABOLISM in CF patients, or ciliated scores higher in UNC medium and secretory scores higher in SC medium. Save heatmaps as PNG files and pair with Plot.Genesets.vs.Sampled (Plot = 'PC1Median') when ranking shifted modules. Optionally follow with GetTopContrib (nGenes = 0.1, OrderType = 'Abs', Mode = 'Wei') to identify leading genes such as GSN in COAGULATION.

## Related tools

- **rROMA** (Provides the Plot.Genesets.Samples function used to draw clustered heatmaps of module sample scores with group annotations, plus Plot.Genesets.vs.Sampled for PC1Median module ranking) — https://github.com/sysbio-curie/rROMA
- **R** (Runtime environment in which rRoma visualization functions are executed)
- **edgeR** (cpm normalization of the GSE176121 count matrix upstream of rRoma, ensuring scores plotted in the heatmap come from normalized data)

## Examples

```
Plot.Genesets.Samples(rRoma.output, Genesets = shiftedModules, GroupInfo = Type, cluster_cols = TRUE)
```

## Evaluation signals

- Heatmap columns (samples) cluster consistently with the GroupInfo labels — e.g., CF and non-CF samples, or UNC and SC medium samples, form coherent blocks
- Shifted modules highlighted in the heatmap match the selection criteria: 3 shifted genesets including FATTY_ACID_METABOLISM and APICAL_SURFACE for the GSE176121 HALLMARK analysis
- APICAL_SURFACE sample scores are visibly higher in healthy controls and FATTY_ACID_METABOLISM scores higher in CF patients, reproducing the article's finding
- For cell-type signatures, ciliated module scores are higher in UNC medium and secretory module scores higher in SC medium, matching expected abundances
- Output files are saved as PNG heatmaps and accompany the ModuleMatrix CSV export

## Limitations

- The article notes the CF case study lacks statistical power (small sample and module count), so q-values were ignored in module selection and the heatmap interpretation should be treated as exploratory
- Default hyperparameter values used by rRoma internals (e.g., null-distribution ApproxSample settings, nSamples = 100) are not fully documented in the article, so heatmap patterns can vary with the random seed unless the seed is fixed
- rRoma works only with normalized expression data; heatmaps built on unnormalized matrices are invalid

## Evidence

- [other] Visualize sample scores of the shifted and overdispersed modules with Plot.Genesets.Samples (GroupInfo = Type, cluster_cols = TRUE) and Plot.Genesets.vs.Sampled (Plot = 'PC1Median'): [evidence span withheld — anchor status: not_found]
- [readme] rRoma works with normalized data: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation"
- [methods] we can then load sample labels and pass them as group info: "We can then load sample labels"
- [intro] distinguishing shifted and over-dispersed gene sets: "estimate the statistical significance of the distribution of samples along the first component for a gene set in two ways: it distinguishes between shifted and over-dispersed gene sets"
- [other] heatmaps saved as PNG confirming cell-type abundance by culture medium: [evidence span withheld — anchor status: not_found]
