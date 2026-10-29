---
name: multimodal-annadata-concatenation
description: Use when an AnnData object must jointly carry a normalized expression
  matrix and one or more conflicting annotation columns (ADT-derived clusters, label-transfer
  results, or GoT clonal genotypes) before running scTriangulate.
when_to_use_negative:
- Data are raw unnormalized counts — CLR (ADT) or log CPT (RNA) normalization must
  be applied first, otherwise concatenated values are not comparable
- Only a single modality with a single non-conflicting annotation exists and no decision-level
  integration of conflicting cluster labels is intended
- The AnnData has an adata.raw attribute and it has not been removed (del adata.raw)
  — this will interfere with how scanpy calculates marker genes downstream
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scTriangulate
  role: Provides scTriangulate.preprocessing.concat_rna_and_other, Normalization.CLR_normalization,
    and add_umap used to build the combined AnnData; version 0.9.0 cited for the methods
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy
  role: Reads and normalizes the RNA/ADT count matrices (scanpy_recipe) prior to concatenation;
    version 1.7.2 used in the paper
- name: cellHarmony
  role: Supplies one of the conflicting annotation sets (Hay et al. CD34+ reference
    clusters) that is placed into the combined AnnData alongside clonal-genotype labels
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:e4e3002dc0bfff3cb43d85469bdfbea780b527e8e8fc4d9def2099118efb1e2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - cellHarmony
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# multimodal-annadata-concatenation

## Summary

Combine properly normalized RNA expression values (e.g. log CPT) with another modality or annotation layer — ADT CLR-normalized counts, or clonal genotype (GoT) assignments — into a single AnnData object using scTriangulate.preprocessing.concat_rna_and_other, so conflicting annotation sets can be supplied to scTriangulate for decision-level integration.

## When to use

Use when an AnnData object must jointly carry a normalized expression matrix and one or more conflicting annotation columns (ADT-derived clusters, label-transfer results, or GoT clonal genotypes) before running scTriangulate.

## When NOT to use

- Data are raw unnormalized counts — CLR (ADT) or log CPT (RNA) normalization must be applied first, otherwise concatenated values are not comparable
- Only a single modality with a single non-conflicting annotation exists and no decision-level integration of conflicting cluster labels is intended
- The AnnData has an adata.raw attribute and it has not been removed (del adata.raw) — this will interfere with how scanpy calculates marker genes downstream

## Inputs

- Normalized single-cell RNA count matrix (log CPT) from scTriangulate.preprocessing.scanpy_recipe
- CLR-normalized ADT counts or gene-mutation (GoT) clonal genotype assignments per cell
- Existing AnnData object with cells barcoded consistently across modalities (e.g. from GEO GSE117825 pre-processed matrices)

## Outputs

- Combined AnnData object containing normalized RNA values plus the other modality/annotation columns in .obs (e.g. cellHarmony clusters, clonal-genotype subclusters)
- adata.obsm['X_umap'] coordinates for downstream scTriangulate visualization

## How to apply

First ensure adata.X holds properly normalized values — log CPT for RNA or center log-ratio (CLR) for ADT, using scTriangulate.preprocessing.Normalization.CLR_normalization where needed. Then call scTriangulate.preprocessing.concat_rna_and_other to concatenate the normalized RNA matrix with the other modality/annotation layer into one AnnData. In the MPN GoT use case, the concatenated object carries cellHarmony cluster labels and clonal-genotype subclusters (WT, SF3B1-mutant, SF3B1-CALR, SF3B1-CALR-NFE2) as the conflicting annotation sets supplied to the query argument. The concatenated AnnData should also include adata.obsm['X_umap'] (via scanpy_recipe or add_umap) and must NOT have an adata.raw attribute, which interferes with scanpy marker-gene calculation. This prepares inputs for ScTriangulate.lazy_run with win_fraction_cutoff = 0.25 and default pruning (abs_thresh = 10).

## Related tools

- **scTriangulate** (Provides scTriangulate.preprocessing.concat_rna_and_other, Normalization.CLR_normalization, and add_umap used to build the combined AnnData; version 0.9.0 cited for the methods) — https://github.com/frankligy/scTriangulate
- **scanpy** (Reads and normalizes the RNA/ADT count matrices (scanpy_recipe) prior to concatenation; version 1.7.2 used in the paper)
- **cellHarmony** (Supplies one of the conflicting annotation sets (Hay et al. CD34+ reference clusters) that is placed into the combined AnnData alongside clonal-genotype labels)

## Examples

```
from sctriangulate.preprocessing import concat_rna_and_other, Normalization; Normalization.CLR_normalization(adt); adata = concat_rna_and_other(rna_adata, adt_adata)
```

## Evaluation signals

- The combined AnnData contains one observation per cell barcode with both normalized RNA values in .X and the conflicting annotation columns (e.g. cellHarmony cluster, clonal genotype) in .obs
- No adata.raw attribute remains on the concatenated object (README warns this 'will interfere how scanpy calculate marker genes')
- adata.obsm['X_umap'] is populated so scTriangulate can automatically generate visualizations
- ADT values, if included, were CLR-normalized via scTriangulate.preprocessing.Normalization.CLR_normalization rather than log-normalized
- Downstream lazy_run with win_fraction_cutoff = 0.25 produces pruned, reclassified annotations, confirming the concatenated object was accepted as valid query input

## Limitations

- Concatenation assumes cells are matched by barcode across modalities; shallow GoT sequencing produces false-negative genotype calls and ambient RNA produces false positives, which propagate into the concatenated annotation columns
- The README notes support for integrating multiple samples of the same modality and across different multimodal/spatial technologies is planned but not yet available
- Programmatic default analytical settings may not be optimal for all study designs

## Evidence

- [methods] we concatenate both properly normalized RNA and ADT values to a combined AnnData through scTriangulate.preprocessing.concat_rna_and_other function.: "we concatenate both properly normalized RNA and ADT values to a combined AnnData through scTriangulate.preprocessing.concat_rna_and_other function."
- [readme] (b) at least two columns representing conflicting annotations in adata.obs, passed to query argument: [evidence span withheld — anchor status: external_doc]
- [methods] we conducted Center log-ratio (CLR) normalization using scTriangulate.preprocessing.Normalization.CLR_normalization function.: "we conducted Center log-ratio (CLR) normalization using scTriangulate.preprocessing.Normalization.CLR_normalization function."
- [methods] The obtained annotations and UMAP coordinates24 were added to the existing AnnData by the scTriangulate.preprocessing.add_annotation and scTriangulate.preprocessing.add_umap functions: "The obtained annotations and UMAP coordinates24 were added to the existing AnnData by the scTriangulate.preprocessing.add_annotation and scTriangulate.preprocessing.add_umap functions"
- [readme] don't have adata.raw attribute, if have, del adata.raw, this will interfere how scanpy calculate marker genes: [evidence span withheld — anchor status: external_doc]
