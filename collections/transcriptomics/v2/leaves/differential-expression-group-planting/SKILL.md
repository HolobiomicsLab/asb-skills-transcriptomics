---
name: differential-expression-group-planting
description: Use when you need a controlled synthetic scRNA-seq dataset with known
  ground-truth cell groups defined by distinct DE gene combinations, e.g. to benchmark
  cluster-merging or cluster-significance tools against planted truth.
when_to_use_negative:
- You need biologically interpretable cell types rather than synthetic groups with
  artificially planted DE signatures — use a real annotated dataset instead
- Your clustering method requires continuous trajectory or lineage structure; this
  skill only plants discrete DE-defined groups
- You are testing tools that depend on realistic dropout/covariate structure not captured
  by the simple Splatter splat model
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Splatter
  role: Estimates generative count-model parameters from the PBMC 3K matrix (splatEstimate)
    and simulates the 10,000 x 3,000 synthetic count matrix (splatSimulate)
- name: scanpy
  role: Provides the reference pbmc3k tutorial dataset and processing workflow used
    as the source for parameter estimation
  repo: https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- name: AnnData
  role: Container for storing simulated counts, gene metadata, and group labels in
    the output .h5ad file
- name: Cytocipher
  role: Downstream consumer of the simulated benchmark; its cluster-significance calls
    are evaluated against the planted groups
  repo: https://github.com/BradBalderson/Cytocipher/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:1ae7b68fe94f4a64aba82e9a02cbc96ea722087a81741844c188ffece29dbfd7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - Cytocipher
  - Splatter
  - scanpy
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# differential-expression-group-planting

## Summary

Generates a synthetic scRNA-seq benchmark by simulating counts from Splatter parameters estimated on PBMC 3K, then planting differential expression by assigning each of 8 cell groups a unique random combination of 3–7 genes from a 15-gene DE pool with N(6,1) fold-change scaling. Used to create ground-truth clusters for evaluating cluster-significance methods like Cytocipher.

## When to use

Use when you need a controlled synthetic scRNA-seq dataset with known ground-truth cell groups defined by distinct DE gene combinations, e.g. to benchmark cluster-merging or cluster-significance tools against planted truth.

## When NOT to use

- You need biologically interpretable cell types rather than synthetic groups with artificially planted DE signatures — use a real annotated dataset instead
- Your clustering method requires continuous trajectory or lineage structure; this skill only plants discrete DE-defined groups
- You are testing tools that depend on realistic dropout/covariate structure not captured by the simple Splatter splat model

## Inputs

- Raw count matrix from a real scRNA-seq dataset (PBMC 3K via scanpy.datasets.pbmc3k) for Splatter parameter estimation
- Splatter v1.20 (splatEstimate / splatSimulate)
- Simulation parameters: 10,000 genes, 3,000 cells, 15-gene DE pool, 8 groups, 3–7 DE genes per group, fold-change ~ N(6,1)

## Outputs

- AnnData object (simulated_pbmc3k_splat.h5ad) containing the simulated count matrix, gene metadata, and ground-truth group labels

## How to apply

First estimate Splatter's generative count-model parameters with splatEstimate from a real count matrix — the Cytocipher benchmark used the scanpy pbmc3k tutorial dataset as the reference. Simulate a count matrix of 10,000 genes x 3,000 cells with splatSimulate using the estimated parameters (Splatter v1.20). Randomly designate 15 genes as the differential-expression pool, partition cells into 8 groups, and assign each group a unique random set of 3–7 pool genes so that groups are defined by gene-combination signatures rather than single markers. Scale each group's assigned genes by a multiplicative fold-change drawn from a normal distribution N(6,1). Store the simulated counts, gene metadata, and group labels in an AnnData object (e.g. simulated_pbmc3k_splat.h5ad) so the ground-truth labels can be compared against clusters recovered by the tool under test.

## Related tools

- **Splatter** (Estimates generative count-model parameters from the PBMC 3K matrix (splatEstimate) and simulates the 10,000 x 3,000 synthetic count matrix (splatSimulate))
- **scanpy** (Provides the reference pbmc3k tutorial dataset and processing workflow used as the source for parameter estimation) — https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- **AnnData** (Container for storing simulated counts, gene metadata, and group labels in the output .h5ad file)
- **Cytocipher** (Downstream consumer of the simulated benchmark; its cluster-significance calls are evaluated against the planted groups) — https://github.com/BradBalderson/Cytocipher/

## Examples

```
import scanpy as sc; from cytocipher_bench import simulate; adata = simulate(splat_params=splatEstimate(sc.datasets.pbmc3k().X), n_genes=10000, n_cells=3000, n_groups=8, de_pool=15, de_per_group=(3,7), fc_mean=6, fc_sd=1); adata.write('simulated_pbmc3k_splat.h5ad')
```

## Evaluation signals

- Output .h5ad contains exactly 10,000 genes and 3,000 cells
- Exactly 8 groups are present in the labels, each with a unique combination of 3–7 genes drawn from the 15-gene DE pool (no duplicated group signatures)
- Only genes assigned to a group show the planted fold-change scaling; non-pool genes remain at simulated baseline expression
- Simulated counts are consistent with Splatter parameters estimated from the PBMC 3K input (splatEstimate output reused for splatSimulate)
- Downstream clustering/significance testing on the simulated data can be compared cell-by-cell against the stored ground-truth group labels

## Limitations

- Ground-truth structure is artificial: groups differ only by multiplicative fold-changes on a small gene set, so performance on the benchmark may not transfer to real biological heterogeneity
- The fold-change distribution N(6,1) and the 3–7 gene-per-group design are fixed benchmark choices; other effect sizes or group structures require re-deriving the simulation
- Parameter realism is bounded by the single PBMC 3K reference dataset used for splatEstimate

## Evidence

- [methods] Simulated dataset composition: [evidence span withheld — anchor status: not_found]
- [methods] Fold-change scaling rule: [evidence span withheld — anchor status: not_found]
- [methods] Parameter estimation reference workflow: [evidence span withheld — anchor status: not_found]
- [methods] Output container and file: [evidence span withheld — anchor status: not_found]
- [readme] Expected input for downstream evaluation: "An AnnData object, *data*, that has been processed similarly to the 
[scanpy standard workflow](https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html)
to produce log-cpm normalised data"
