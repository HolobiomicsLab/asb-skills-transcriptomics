---
name: clustering-concordance-adjusted-rand-index
description: Use when independent cluster assignments of the same cells exist (e.g.,
  CNA-derived and mutation-derived) and an orthogonal-validation metric is needed
  to confirm a caller's subclone structure.
when_to_use_negative:
- The two labelings cover different sets of cells — ARI requires paired assignments
  of the identical cell set (here 179 cells)
- You only have one clustering and no orthogonal genotype-based labels; ARI is undefined
  without a second partition to compare against
- Cluster labels are continuous (e.g., copy-number values) rather than discrete partition
  assignments
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: HiScanner
  role: Produces the CNA-based cell clusters (joint segmentation, '--step cnv' integer
    allele-specific copy numbers at 500 kb bins) whose concordance with SNV clusters
    is quantified
  repo: https://github.com/parklab/HiScanner
- name: SECEDO
  role: Clusters the 179 cells independently by somatic point-mutation genotype from
    Mutect2-called 'PASS' sSNVs, providing the orthogonal labels for the ARI
- name: Mutect2
  role: Calls somatic SNVs used by SECEDO for the mutation-based clustering
- name: SCAN2
  role: Runs on paired bulk and single-cell BAMs to supply the raw variant VCF and
    phased gHETs consumed by HiScanner upstream of clustering
  repo: https://github.com/parklab/SCAN2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:d00fd86772e07687db9ce2c22a0e83acc3fc63d35b1fe1d3e631b0edd7662dbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/parklab/HiScanner
  tools:
  - BICseq2
  - HiScanner
  - Mutect2
  - SCAN2
  - SECEDO
  - SHAPEIT
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# clustering-concordance-adjusted-rand-index

## Summary

Validates single-cell CNA-based clustering against an orthogonal somatic-SNV-based clustering by computing the Adjusted Rand Index (ARI) between the two sets of cell assignments, as done for 179 serial meningioma cells (ARI 0.83).

## When to use

Use when independent cluster assignments of the same cells exist (e.g., CNA-derived and mutation-derived) and an orthogonal-validation metric is needed to confirm a caller's subclone structure.

## When NOT to use

- The two labelings cover different sets of cells — ARI requires paired assignments of the identical cell set (here 179 cells)
- You only have one clustering and no orthogonal genotype-based labels; ARI is undefined without a second partition to compare against
- Cluster labels are continuous (e.g., copy-number values) rather than discrete partition assignments

## Inputs

- Per-cell 500 kb integer allele-specific copy-number matrix from HiScanner ('--step cnv') for 179 meningioma cells
- SECEDO sSNV-based cluster assignments for the same 179 cells (Mutect2 'PASS' variants)
- HiScanner CNA-based hierarchical cluster assignments (complete linkage, four clusters)

## Outputs

- Adjusted Rand Index value quantifying agreement between CNA-based and SNV-based clusters (expected ~0.83)
- Table of per-cell cluster assignments from both methods plus the ARI

## How to apply

First derive cluster labels per cell from copy-number profiles (e.g., assemble the per-cell 500 kb allele-specific copy-number matrix from HiScanner and apply complete-linkage hierarchical clustering, cutting the tree into the expected number of clusters — four for the meningioma data, corresponding to Clusters 8, 11, 13, and 15, with grade II cell BOS-EE06 in Cluster 15). Independently derive clusters from somatic point mutations: run SECEDO on Mutect2-called sSNVs, keeping only 'PASS' variants, to cluster the same cells by SNV genotype. Then compute the Adjusted Rand Index between the two label vectors; values near the reported 0.83 indicate strong agreement and support the validity of the CNA calls. Finally, write both cluster assignments and the ARI to an output table so the concordance result is auditable.

## Related tools

- **HiScanner** (Produces the CNA-based cell clusters (joint segmentation, '--step cnv' integer allele-specific copy numbers at 500 kb bins) whose concordance with SNV clusters is quantified) — https://github.com/parklab/HiScanner
- **SECEDO** (Clusters the 179 cells independently by somatic point-mutation genotype from Mutect2-called 'PASS' sSNVs, providing the orthogonal labels for the ARI)
- **Mutect2** (Calls somatic SNVs used by SECEDO for the mutation-based clustering)
- **SCAN2** (Runs on paired bulk and single-cell BAMs to supply the raw variant VCF and phased gHETs consumed by HiScanner upstream of clustering) — https://github.com/parklab/SCAN2

## Examples

```
from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(cna_clusters, snv_clusters)  # expect ~0.83 for 179 meningioma cells
```

## Evaluation signals

- ARI between CNA-based and SNV-based cluster assignments is high, near the reported 0.83 for the 179 meningioma cells
- CNA-based hierarchical clustering (complete linkage) recovers four clusters corresponding to Clusters 8, 11, 13, and 15
- The grade II cell BOS-EE06 is assigned to Cluster 15 in the CNA-based clustering
- Both clusterings assign the same 179 cells (initial n=88, recurrent n=91) so the label vectors are directly comparable
- Per-cell assignments and ARI are written to the output table as required

## Limitations

- The HiScanner model assumes independence between single-cell genomes, which may be violated for cells sharing clonal origins — relevant since this skill clusters clonally related cells
- Clonal classification may conflate true clonal expansions with independent recurrent mutations sharing identical coordinates, which could inflate or distort the SNV-based partition
- No data accession numbers were provided for the meningioma single-cell data, hindering independent reproduction

## Evidence

- [discussion] CNA–SNV cluster concordance ARI 0.83: [evidence span withheld — anchor status: not_found]
- [intro] Orthogonal clustering rationale: "HiScanner revealed phases and timing of tumor progression by detecting subclone-defining CNAs, which were subsequently corroborated by an orthogonal clustering based on somatic point mutations"
- [discussion] Filter for SNV-based clustering: "Only variants with “PASS” flag were kept"
- [other] CNA matrix construction and clustering: [evidence span withheld — anchor status: not_found]
- [other] Clone structure expectation: [evidence span withheld — anchor status: not_found]
