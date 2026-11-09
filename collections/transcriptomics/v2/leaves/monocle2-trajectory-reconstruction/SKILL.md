---
name: monocle2-trajectory-reconstruction
description: Use when a time-course or stage-ordered scRNA-seq matrix (e.g. 262 pre-implantation
  embryo cells from oocyte to late blastocyst) needs to be ordered along a pseudotime
  development trajectory with monocle2 DDRTree.
when_to_use_negative:
- The goal is ligand-receptor communication inference rather than trajectory reconstruction
  (use TimeTalk's higher-level workflow instead)
- Cell identities/stages are already definitively resolved and no temporal ordering
  is needed — tools like CellPhoneDB/CellChat for given cell types would be more appropriate
- Input is raw unnormalized counts without the merged quantile-normalized matrix the
  pipeline expects
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: monocle2
  role: 'Core tool: differentialGeneTest (qval = 0.01), reduceDimension with DDRTree,
    and orderCells to build and order the pseudotime trajectory'
- name: R
  role: Environment for loading the normalized matrix, building the CellDataSet object,
    and running monocle2 (statistical analyses performed in R 3.6.3)
- name: preprocessCore
  role: Provides 'normalize.quantiles' (version 1.48.0) used to produce the quantile-normalized
    expression matrix that feeds the CellDataSet
- name: monocle3
  role: Related trajectory tool installed in the TimeTalk environment (RunMonocle3
    wrapper); the article's trajectory step itself uses monocle2 DDRTree
  repo: https://github.com/cole-trapnell-lab/monocle3
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:9d784cc7413a9832459e75b8108e55a11757667c9391ef8f555dad7e7eb1ae49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cole-trapnell-lab/monocle3
  tools:
  - R
  - TimeTalk
  - monocle2
  - monocle3
  - preprocessCore
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# monocle2-trajectory-reconstruction

## Summary

Reconstruct a developmental pseudotime trajectory from single-cell RNA-seq using monocle2: differential gene testing followed by DDRTree dimension reduction and orderCells to place all cells along a pseudotime axis (e.g. oocyte to late blastocyst).

## When to use

Use when a time-course or stage-ordered scRNA-seq matrix (e.g. 262 pre-implantation embryo cells from oocyte to late blastocyst) needs to be ordered along a pseudotime development trajectory with monocle2 DDRTree.

## When NOT to use

- The goal is ligand-receptor communication inference rather than trajectory reconstruction (use TimeTalk's higher-level workflow instead)
- Cell identities/stages are already definitively resolved and no temporal ordering is needed — tools like CellPhoneDB/CellChat for given cell types would be more appropriate
- Input is raw unnormalized counts without the merged quantile-normalized matrix the pipeline expects

## Inputs

- Merged, quantile-normalized scRNA-seq expression matrix (preprocessCore 'normalize.quantiles' output)
- Cell-stage annotation for each cell (oocyte through late blastocyst, 262 pre-implantation cells in the article)

## Outputs

- monocle2 CellDataSet object with DDRTree-reduced dimensions
- Per-cell pseudotime ordering covering all cells from oocyte to late blastocyst
- Trajectory plot serialized/visualized to reproduce the developmental-stage arrangement (Fig. 1e layout)

## How to apply

Load the merged, quantile-normalized expression matrix (quantile normalization via preprocessCore 'normalize.quantiles', version 1.48.0) into R and build a monocle2 CellDataSet object for all cells (e.g. 262 pre-implantation cells). Run 'differentialGeneTest' and retain differentially expressed genes at cutoff qval = 0.01. Apply 'reduceDimension' with DDRTree on the selected genes to reconstruct the development trajectory, then call 'orderCells' to arrange all cells along pseudotime from the earliest stage (oocyte) to the latest (late blastocyst). Serialize/visualize the ordered trajectory so it can be compared against a reference figure layout (here Fig. 1e).

## Related tools

- **monocle2** (Core tool: differentialGeneTest (qval = 0.01), reduceDimension with DDRTree, and orderCells to build and order the pseudotime trajectory)
- **R** (Environment for loading the normalized matrix, building the CellDataSet object, and running monocle2 (statistical analyses performed in R 3.6.3))
- **preprocessCore** (Provides 'normalize.quantiles' (version 1.48.0) used to produce the quantile-normalized expression matrix that feeds the CellDataSet)
- **monocle3** (Related trajectory tool installed in the TimeTalk environment (RunMonocle3 wrapper); the article's trajectory step itself uses monocle2 DDRTree) — https://github.com/cole-trapnell-lab/monocle3

## Examples

```
cds <- reduceDimension(cds, method = 'DDRTree'); cds <- orderCells(cds)
```

## Evaluation signals

- The output trajectory must contain all 262 cells with no cells dropped
- Cells must be ordered along pseudotime from oocyte to late blastocyst, matching the developmental-stage arrangement shown in Fig. 1e
- Only genes passing differentialGeneTest with qval = 0.01 are used for the DDRTree dimension reduction
- The input expression matrix must be the quantile-normalized matrix produced with preprocessCore 'normalize.quantiles' (version 1.48.0)

## Limitations

- Monocle2 is a dependency installed alongside the TimeTalk environment (monocle3 and cellAlign via devtools); the DDRTree trajectory was performed in the discussion/methods workflow rather than via the shipped TimeTalk demo, which uses monocle3
- Pseudotime ordering is validated visually against Fig. 1e; no quantitative accuracy metric beyond containing all 262 cells in stage order is given
- The article notes Granger/trajectory downstream conclusions are parameter-sensitive (e.g. winsz) and advise testing multiple parameter combinations for dependable results

## Evidence

- [discussion] the differential expressed genes were used to perform DDRTree dimension reduction to reconstruct the development trajectory by the “reduceDimension” function.: "the differential expressed genes were used to perform DDRTree dimension reduction to reconstruct the development trajectory by the “reduceDimension” function."
- [discussion] we used the “differentialGeneTest” function to get differentially expressed genes (cutoff: qval = 0.01): "we used the “differentialGeneTest” function to get differentially expressed genes (cutoff: qval = 0.01)"
- [discussion] Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0).: "Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0)."
- [readme] devtools::install_github('cole-trapnell-lab/monocle3'): "devtools::install_github('cole-trapnell-lab/monocle3')"
