---
name: pseudotime-cell-ordering
description: Use when you have a merged, quantile-normalized scRNA-seq matrix spanning
  sequential developmental stages (e.g. pre-implantation mouse embryos) and need cells
  ordered along pseudotime for trajectory-based or temporal-communication analyses.
when_to_use_negative:
- Analysis targets communication between discrete, well-defined cell types — tools
  like CellPhoneDB or CellChat are designed for that setting and do not need pseudotime
  ordering.
- Input is not a single integrated expression matrix across a temporal/developmental
  series (e.g. it is already a per-stage pseudo-bulk table or a cell-type-level feature
  matrix).
- Cells lack sequential stage labels or the biological process is not developmental/temporal,
  so pseudotime ordering would not reflect a meaningful progression.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: monocle2
  role: Builds the CellDataSet, runs differentialGeneTest (qval = 0.01), DDRTree reduceDimension,
    and orderCells to produce the pseudotime ordering
  repo: https://github.com/cole-trapnell-lab/monocle3
- name: R
  role: Analysis environment; quantile normalization of the merged matrix via preprocessCore
    'normalize.quantiles' precedes trajectory construction
- name: TimeTalk
  role: Downstream framework that consumes the trajectory-ordered cells to infer temporally-resolved
    ligand-receptor communication
  repo: https://github.com/ChengLiLab/TimeTalk
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:920df0745cb11bfc64ca5b29100e7c8c0cabe5f28bc99aedc86bbc39b95dfd59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cole-trapnell-lab/monocle3
  tools:
  - R
  - TimeTalk
  - monocle2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pseudotime-cell-ordering

## Summary

Orders single cells along a developmental pseudotime trajectory by building a monocle2 CellDataSet from a merged, quantile-normalized scRNA-seq expression matrix, selecting differentially expressed genes (qval = 0.01), running DDRTree dimension reduction, and calling orderCells. Used to recapitulate early embryo development progression (oocyte to late blastocyst) as a scaffold for downstream temporal analyses such as TimeTalk.

## When to use

Use when you have a merged, quantile-normalized scRNA-seq matrix spanning sequential developmental stages (e.g. pre-implantation mouse embryos) and need cells ordered along pseudotime for trajectory-based or temporal-communication analyses.

## When NOT to use

- Analysis targets communication between discrete, well-defined cell types — tools like CellPhoneDB or CellChat are designed for that setting and do not need pseudotime ordering.
- Input is not a single integrated expression matrix across a temporal/developmental series (e.g. it is already a per-stage pseudo-bulk table or a cell-type-level feature matrix).
- Cells lack sequential stage labels or the biological process is not developmental/temporal, so pseudotime ordering would not reflect a meaningful progression.

## Inputs

- Merged, quantile-normalized expression matrix (preprocessCore 'normalize.quantiles', version 1.48.0)
- Cell-level stage/annotation metadata for all cells (e.g. oocyte, 2-cell, ... late blastocyst)
- monocle2 CellDataSet object constructed from the matrix for the 262 pre-implantation cells

## Outputs

- monocle2 trajectory with all cells ordered along pseudotime from oocyte to late blastocyst
- DDRTree reduced-dimension trajectory layout matching the developmental-stage arrangement (Fig. 1e)
- Set of differentially expressed genes retained at qval = 0.01 used for trajectory reconstruction

## How to apply

Load the merged expression matrix — quantile-normalized with preprocessCore 'normalize.quantiles' (version 1.48.0) — into R and build a monocle2 CellDataSet for all cells (262 pre-implantation cells in the source study). Run monocle2 'differentialGeneTest' and retain differentially expressed genes at cutoff qval = 0.01. Perform DDRTree dimension reduction on the selected genes via 'reduceDimension' to reconstruct the development trajectory, then call 'orderCells' to arrange all cells along pseudotime from oocyte to late blastocyst. Validate that the trajectory contains every cell and that the ordering reproduces the expected developmental-stage arrangement (Fig. 1e layout in the source paper); the ordered cells then feed downstream steps such as co-varying LR pair identification by PCC in TimeTalk.

## Related tools

- **monocle2** (Builds the CellDataSet, runs differentialGeneTest (qval = 0.01), DDRTree reduceDimension, and orderCells to produce the pseudotime ordering) — https://github.com/cole-trapnell-lab/monocle3
- **R** (Analysis environment; quantile normalization of the merged matrix via preprocessCore 'normalize.quantiles' precedes trajectory construction)
- **TimeTalk** (Downstream framework that consumes the trajectory-ordered cells to infer temporally-resolved ligand-receptor communication) — https://github.com/ChengLiLab/TimeTalk

## Examples

```
cds <- reduceDimension(cds, reduction_method = "DDRTree"); cds <- orderCells(cds)
```

## Evaluation signals

- Output trajectory contains all 262 pre-implantation cells, none dropped or duplicated.
- Pseudotime ordering runs from oocyte to late blastocyst, consistent with the developmental-stage arrangement shown in Fig. 1e.
- Genes used for DDRTree dimension reduction are exactly those passing differentialGeneTest at qval = 0.01.
- Input matrix was quantile-normalized (preprocessCore 'normalize.quantiles', version 1.48.0) before CellDataSet construction.
- Ordered trajectory is a valid input to downstream steps (e.g. PCC calculation on ordered cells with |PCC| = 0.1 cutoff).

## Limitations

- Trajectory quality depends on the differentially expressed gene selection cutoff (qval = 0.01); different cutoffs yield different DDRTree layouts.
- Pseudotime reconstruction requires a merged, batch-assessed matrix; batch effects should be checked (e.g. kBET in PCA space) before ordering.
- TimeTalk's downstream causal conclusions are parameter-sensitive (e.g. interpolation winsz), so the pseudotime scaffold alone does not guarantee dependable downstream results; users are advised to test multiple parameter combinations.

## Evidence

- [discussion] DDRTree reduction: "the differential expressed genes were used to perform DDRTree dimension reduction to reconstruct the development trajectory by the “reduceDimension” function."
- [discussion] DEG cutoff: "we used the “differentialGeneTest” function to get differentially expressed genes (cutoff: qval = 0.01)"
- [discussion] normalization: "Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0)."
- [readme] monocle install: "devtools::install_github('cole-trapnell-lab/monocle3')"
