---
name: spark-x-spatial-statistics-execution
description: Use when you need per-gene spatially variable gene (SVG) rankings from
  spatial transcriptomics data — e.g., a Slide-seqV2 gene-by-bead puck matrix with
  2D coordinates — to benchmark against singleCellHaystack's ranks.
when_to_use_negative:
- The dataset lacks physical spatial coordinates (e.g., pbmc3k scRNA-seq, PCA latent
  spaces, or 1D trajectories) — SPARK-X targets spatial inputs, while singleCellHaystack
  handles 1D trajectories and higher-dimensional latent spaces.
- 'Maximum sensitivity to strongly patterned genes matters more than runtime: the
  hippocampus benchmark showed SPARK-X ranking Plp1 1139th and Mal 1543rd where singleCellHaystack
  ranked them 20th and 55th.'
- The goal is multi-condition comparison (e.g., wild-type vs knockout) rather than
  ranking spatial variability within a single tissue section.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SPARK-X
  role: Runs the spatial variability test on the all-gene puck matrix with identical
    2D coordinates, returning the fast per-gene SVG ranking baseline
- name: singleCellHaystack
  role: Comparator method whose per-gene differential-activity ranks (run on 2D spatial
    coordinates, R version 1.0.0) are extracted and diffed against SPARK-X ranks
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: scanpy
  role: Loads the puck expression matrix into an AnnData object and attaches the 2D
    puck coordinates as the cells' spatial input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:8152d0905ee4b0fe8fcdb06553eb89af3e170922a020dd4ce9a7957d4a8966b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - Python
  - R
  - SPARK-X
  - scanpy
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# spark-x-spatial-statistics-execution

## Summary

Executes SPARK-X to produce per-gene spatially variable gene (SVG) ranks from a gene-by-bead expression matrix with 2D puck coordinates (e.g., Slide-seqV2 mouse hippocampus), serving as the fast spatial ranking baseline that the article benchmarks against singleCellHaystack's more sensitive differential-activity approach.

## When to use

Use when you need per-gene spatially variable gene (SVG) rankings from spatial transcriptomics data — e.g., a Slide-seqV2 gene-by-bead puck matrix with 2D coordinates — to benchmark against singleCellHaystack's ranks.

## When NOT to use

- The dataset lacks physical spatial coordinates (e.g., pbmc3k scRNA-seq, PCA latent spaces, or 1D trajectories) — SPARK-X targets spatial inputs, while singleCellHaystack handles 1D trajectories and higher-dimensional latent spaces.
- Maximum sensitivity to strongly patterned genes matters more than runtime: the hippocampus benchmark showed SPARK-X ranking Plp1 1139th and Mal 1543rd where singleCellHaystack ranked them 20th and 55th.
- The goal is multi-condition comparison (e.g., wild-type vs knockout) rather than ranking spatial variability within a single tissue section.

## Inputs

- Gene-by-bead expression matrix from a Slide-seqV2 puck (e.g., Stickels_hippo, Broad Single Cell Portal accession SCP815)
- 2D puck coordinates (x, y) per bead, used as spatial input coordinates
- AnnData object with spatial coordinates attached (scanpy)

## Outputs

- Per-gene SPARK-X SVG ranks over the full gene set
- svg_rank_comparison.csv with per-gene ranks under SPARK-X and singleCellHaystack, rank differences, and agreement flags

## How to apply

Download the Stickels_hippo Slide-seqV2 puck gene-by-bead matrix and 2D puck coordinates (Broad Single Cell Portal, accession SCP815), load the matrix into an AnnData object with scanpy, and attach the coordinates as the spatial input. Run SPARK-X on the full all-gene expression matrix using the identical 2D coordinate input used for singleCellHaystack (R package, version 1.0.0), so per-gene SVG ranks are computed on exactly the same data basis. Extract each gene's rank under both methods and compute rank differences for the target genes Plp1, Mal, Zfhx3, and Cox4i1, where the reported expectation is SPARK-X 1139/1543/64/183 versus singleCellHaystack 20/55/378/1123. Write svg_rank_comparison.csv containing per-gene ranks and agreement flags, flagging any row whose computed ranks deviate from the reported values — accept only exact matches for the four target genes. Rationale: the article identifies SPARK-X as the fastest spatial-transcriptomics method, but its lower sensitivity means strongly patterned genes can be under-ranked, so exact input parity is the critical decision point for a fair comparison.

## Related tools

- **SPARK-X** (Runs the spatial variability test on the all-gene puck matrix with identical 2D coordinates, returning the fast per-gene SVG ranking baseline)
- **singleCellHaystack** (Comparator method whose per-gene differential-activity ranks (run on 2D spatial coordinates, R version 1.0.0) are extracted and diffed against SPARK-X ranks) — https://github.com/alexisvdb/singleCellHaystack
- **scanpy** (Loads the puck expression matrix into an AnnData object and attaches the 2D puck coordinates as the cells' spatial input)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad("data.h5ad")
```

## Evaluation signals

- Computed SPARK-X ranks for the four target genes exactly match the reported values: Plp1 1139th, Mal 1543rd, Zfhx3 64th, Cox4i1 183rd; mismatched rows are flagged, not silently accepted.
- Input parity holds: the SPARK-X run uses the same all-gene expression matrix and identical 2D coordinate input as the singleCellHaystack run.
- Every gene in the matrix receives a rank under both methods, so a full rank-difference column is computable for svg_rank_comparison.csv.
- Expected discrepancy direction reproduces: genes top-ranked by singleCellHaystack (Plp1 20th, Mal 55th) fall far lower under SPARK-X (1139th, 1543rd), while Zfhx3 and Cox4i1 show the reverse pattern (SPARK-X 64th/183rd vs 378th/1123rd).
- Output CSV contains per-gene ranks plus agreement flags, and validation fails closed if any of the four target-gene ranks mismatch.

## Limitations

- SPARK-X can strongly under-rank genes with clear spatial expression patterns: in the Slide-seqV2 mouse hippocampus comparison, Plp1 ranked 1139th (vs 20th for singleCellHaystack) and Mal 1543rd (vs 55th).
- The sensitivity/speed trade-off is explicit in the discussion: singleCellHaystack's better sensitivity 'makes it the best alternative, despite relatively longer runtimes', so SPARK-X's speed comes at reduced sensitivity.
- Cross-method rank comparisons are only valid with exact matrix and coordinate parity; changing the coordinate basis or gene subset invalidates the rank-difference table.

## Evidence

- [discussion] For spatial transcriptomics the fastest method available is SPARK-X.: "For spatial transcriptomics the fastest method available is SPARK-X."
- [discussion] The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes.: "The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes."
- [readme] DEGs along a trajectory, spatial DEGs, or any other features with non-random levels of activity inside any input space (1D, 2D, or \>2D): [evidence span withheld — anchor status: external_doc]
- [abstract] it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates: "it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates"
