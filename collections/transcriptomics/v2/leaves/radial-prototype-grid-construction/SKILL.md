---
name: radial-prototype-grid-construction
description: 'Use when building a TCM latent space for temporal scRNA-seq data: before
  fitting the tcm model, you need the R×S radial prototype layout (landscape) that
  encodes temporal ordering as radial distance.'
when_to_use_negative:
- The data lack a complete time index, or the time index is not correlated with the
  underlying dynamic expression pattern
- The analysis goal is a non-temporal dimension reduction with no developmental ordering
  to encode radially (e.g. plain t-SNE or diffusion map visualization)
- You need immediate biological interpretation of clusters on the latent space; TCM
  requires further pathway analysis for that
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3308
tools:
- name: tcm
  role: R package providing the landscape() function that builds the radial prototype
    grid and the tcm() function that fits the model on it
  repo: github.com/gongx030/tcm
- name: devtools
  role: Used to install the tcm package from GitHub via install_github('gongx030/tcm')
  repo: https://github.com/hadley/devtools
- name: TCM
  role: The Topographic Cell Map method whose temporal convolving latent space the
    prototype grid defines
  repo: github.com/gongx030/tcm
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:db3511148d08ae8ef617c965275ad1df5080ec5d999b4f5e66a4b7040e549075
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - TCM
  - devtools
  - tcm
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# radial-prototype-grid-construction

## Summary

Constructs the TCM radial prototype grid (landscape) that defines the 2D latent space onto which time-point-specific latent spaces are convolved, placing early developmental time points at the center and later ones toward the periphery.

## When to use

Use when building a TCM latent space for temporal scRNA-seq data: before fitting the tcm model, you need the R×S radial prototype layout (landscape) that encodes temporal ordering as radial distance.

## When NOT to use

- The data lack a complete time index, or the time index is not correlated with the underlying dynamic expression pattern
- The analysis goal is a non-temporal dimension reduction with no developmental ordering to encode radially (e.g. plain t-SNE or diffusion map visualization)
- You need immediate biological interpretation of clusters on the latent space; TCM requires further pathway analysis for that

## Inputs

- Integer number of time points (ncol of the cell-by-time-point binary time.table, e.g. 5)
- Landscape parameters: type='temporal.convolving', K (e.g. 15), n.prototype (e.g. 15), n.circle (e.g. 10), n.prev (e.g. 2)

## Outputs

- A landscape (prototype grid) object defining H = R×S prototypes arranged in concentric circles for the temporal convolving latent space
- Prototype positions exportable as prototype_layout.csv

## How to apply

Call the tcm package's landscape() function with type='temporal.convolving', setting time.points to the number of columns of the cell-by-time-point binary time.table (e.g. 5), K and n.prototype to the number of prototypes per circle (e.g. 15 each, giving H = R×S prototypes per time-point sub-space), n.circle to the number of concentric rings (e.g. 10), and n.prev to the number of previous time points convolved into each ring (e.g. 2). The rationale is that TCM convolves time-point-specific latent spaces into one global latent space where radial position encodes developmental time: early time points sit at the center, later ones at the periphery. Pass the resulting landscape object to tcm() via the ls argument (as shown for the Trapnell myoblast dataset), or rely on the default landscape created by tcm(X, time.table = time.table). Verify afterwards that each per-time-point latent space contains exactly H = R×S prototypes and that mean time index increases monotonically with radial distance from the center.

## Related tools

- **tcm** (R package providing the landscape() function that builds the radial prototype grid and the tcm() function that fits the model on it) — github.com/gongx030/tcm
- **devtools** (Used to install the tcm package from GitHub via install_github('gongx030/tcm')) — https://github.com/hadley/devtools
- **TCM** (The Topographic Cell Map method whose temporal convolving latent space the prototype grid defines) — github.com/gongx030/tcm

## Examples

```
library(tcm); ls <- landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2); mf <- tcm(X, time.table = time.table, ls = ls)
```

## Evaluation signals

- Each per-time-point sub-space of the latent space contains exactly H = R×S prototypes (e.g. 15×15 = 225 with the documented parameters)
- Mean time index increases monotonically with radial distance from the center of the latent space
- The landscape object is accepted by tcm() via the ls argument without error, as demonstrated for the Trapnell dataset
- Cells from early time points lie near the center and cells from later time points near the periphery of the fitted map
- Exported prototype_layout.csv exists and is valid CSV with prototype coordinates

## Limitations

- TCM requires scRNA-seq datasets with a complete time index, and the time index needs to be correlated with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis is needed

## Evidence

- [methods] Landscape specification for temporal convolving: "ls <- landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2)"
- [abstract] TCM convolves latent spaces radially: [evidence span withheld — anchor status: not_found]
- [readme] Radial layout passed to tcm for Trapnell dataset: "ls <- landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2)"
- [discussion] Requirement of complete time index: "TCM requires the scRNA-seq datasets with complete time index and the time index needs to be correlated with the underlying dynamic expression pattern."
