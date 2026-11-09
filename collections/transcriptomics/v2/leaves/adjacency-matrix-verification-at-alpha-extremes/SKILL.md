---
name: adjacency-matrix-verification-at-alpha-extremes
description: Use when implementing or re-implementing the ClonoCluster α/β edge-weight
  combination and you need to prove the integration is anchored correctly before clustering
  at intermediate α values.
when_to_use_negative:
- Input is a clustering assignment or marker table rather than a cell-by-cell weighted
  adjacency matrix — there is nothing to verify at α extremes
- The barcode table and count matrix are from different cells or lack matching cell
  IDs in 'rn', since the clone and transcriptome graphs cannot be aligned edge-wise
- You need to validate hybrid cluster quality or marker fidelity at intermediate α;
  the paper recommends marker fidelity analysis or ground-truth comparison instead
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: ClonoCluster
  role: Provides dt2m, irlba_wrap, build_barcode_matrix, and clonocluster functions
    used to construct and combine the transcriptome and clone edge-weight matrices
  repo: https://github.com/leeprichman/ClonoCluster
- name: data.table
  role: Loads the barcode table and count matrix TSV files via fread
- name: irlba
  role: Computes the 25-PC transcriptome representation wrapped by irlba_wrap that
    feeds the SNN graph
- name: Seurat
  role: Builds the shared-nearest-neighbor Jaccard-weighted transcriptome graph whose
    equality with W at α = 0 is verified
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:b33db6a4fd44f9ad887f03a609bb133de443ce4db4e94682164a9ccc33b589f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - R (version >=)
  - Seurat
  - data.table
  - irlba
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Adjacency matrix verification at alpha extremes

## Summary

Sanity-check that the ClonoClone-weighted transcriptome-clone edge integration W = αβ(lin − J) + J reproduces the pure modality graphs at its boundary values: the transcriptome Jaccard SNN graph at α = 0 and the size-normalized clone-weight graph at α = 1, with all combined weights confined to [0, 1].

## When to use

Use when implementing or re-implementing the ClonoCluster α/β edge-weight combination and you need to prove the integration is anchored correctly before clustering at intermediate α values.

## When NOT to use

- Input is a clustering assignment or marker table rather than a cell-by-cell weighted adjacency matrix — there is nothing to verify at α extremes
- The barcode table and count matrix are from different cells or lack matching cell IDs in 'rn', since the clone and transcriptome graphs cannot be aligned edge-wise
- You need to validate hybrid cluster quality or marker fidelity at intermediate α; the paper recommends marker fidelity analysis or ground-truth comparison instead

## Inputs

- Two-column barcode table (columns 'rn' and 'Barcode') in TSV format read with data.table::fread
- Normalized/scaled count matrix in gene-by-cell TSV format, converted to cell-by-gene via ClonoCluster's dt2m
- 25-principal-component transcriptome representation from irlba_wrap(cm, npc = 25)
- Seurat shared-nearest-neighbor Jaccard-weighted transcriptome graph (edge weight matrix)
- Clone-weight matrix from build_barcode_matrix (1/0 barcode-sharing edges, normalized by cells-per-barcode)

## Outputs

- Combined weighted adjacency matrix W = αβ(lin − J) + J for each α in seq(0, 1, by = 0.1), written to clonocluster_weighted_adjacency.tsv
- Verification report confirming W(α = 0) equals the transcriptome Jaccard graph and W(α = 1) equals the normalized clone-weight graph

## How to apply

After loading the two-column barcode table ('rn', 'Barcode') and the count matrix with data.table::fread and converting to a cell-by-gene matrix via dt2m, compute a 25-PC transcriptome representation with irlba_wrap(cm, npc = 25) and build the Seurat shared-nearest-neighbor Jaccard graph J. Build the clone-weight matrix with build_barcode_matrix, assigning edges weight 1 (same barcode) or 0 (different barcode) and normalizing by the number of cells carrying each barcode. Form the combined matrix W = αβ(lin − J) + J with β = 0.1 across α ∈ seq(0, 1, by = 0.1), as in the clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1) call. Then assert the two boundary identities: at α = 0 the α-scaled clone term vanishes so W equals the transcriptome Jaccard graph exactly, and at α = 1 W equals the size-normalized clone graph; finally check that every edge weight of every combined matrix lies in [0, 1]. Rationale: these extremes verify the α parameterization returns 'purely transcriptome weights for α = 0 and purely clone weights for α = 1' before any downstream hybrid clustering is trusted.

## Related tools

- **ClonoCluster** (Provides dt2m, irlba_wrap, build_barcode_matrix, and clonocluster functions used to construct and combine the transcriptome and clone edge-weight matrices) — https://github.com/leeprichman/ClonoCluster
- **data.table** (Loads the barcode table and count matrix TSV files via fread)
- **irlba** (Computes the 25-PC transcriptome representation wrapped by irlba_wrap that feeds the SNN graph)
- **Seurat** (Builds the shared-nearest-neighbor Jaccard-weighted transcriptome graph whose equality with W at α = 0 is verified)

## Examples

```
clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)
```

## Evaluation signals

- Element-wise equality check: the combined matrix at α = 0 is identical to the transcriptome-only Jaccard SNN graph
- Element-wise equality check: the combined matrix at α = 1 is identical to the size-normalized clone-weight graph from build_barcode_matrix
- Range invariant: all edge weights of every combined matrix lie in [0, 1] for every α in seq(0, 1, by = 0.1)
- Output file clonocluster_weighted_adjacency.tsv is a square cell-by-cell matrix with rows/columns matching the input barcode table's 'rn' cell IDs

## Limitations

- Verification only anchors the boundary behavior; whether intermediate α values give biologically meaningful hybrid clusters requires separate marker-fidelity or ground-truth analysis
- The paper notes it did not reveal a reliable method to identify which experimental systems have intrinsic determinants of cell fate, so a passing verification does not guarantee clone information is informative
- Further testing of the homogeneity of functional properties within clusters is required to determine which clustering methods best match biological distinctions

## Evidence

- [intro] Linear combination of transcriptome and clone weights: "We linearly combined transcriptome and clone weights using α such that it returned purely transcriptome weights for α = 0 and purely clone weights for α = 1"
- [intro] Clone weight definition and normalization: "For each edge between cells, we also created a “clone weight” of 1 or 0 based on whether the cells have the same or different barcodes. We then normalized the “clone weight” by the number of cells"
- [methods] Concrete R invocation with α sweep and β: "clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)"
- [methods] PC computation used to build the transcriptome graph: "pca <- irlba_wrap(cm, npc = 25)"
- [readme] Install path for the software implementing the functions: "devtools::install_github("leeprichman/ClonoCluster")"
