---
name: hierarchical-clustering-of-somatic-cna-profiles
description: Use when you have per-cell allele-specific integer copy-number calls
  across single cells and need to define clone structure, especially to cross-validate
  CNV clones against SNV-genotype clusters.
when_to_use_negative:
- Cells are normal/diploid controls with no somatic CNAs, so clustering would split
  noise rather than clones (e.g., the neurotypical brain cells where aneuploid neurons
  were excluded from downstream analyses)
- No paired bulk data is available for phasing gHETs and VAF filtering, so HiScanner's
  RDR+BAF copy numbers cannot be generated and only RDR-only profiles would be clusterable
- The goal is calling CNAs themselves rather than grouping cells — use the HiScanner
  calling steps instead of clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3673
tools:
- name: HiScanner
  role: Produces the per-cell integer allele-specific copy-number matrix (snp, phase,
    ado, normalize, segment, cnv steps) that is clustered
  repo: https://github.com/parklab/HiScanner
- name: SCAN2
  role: Genotypes somatic SNVs and produces the raw variant VCF and phased gHETs consumed
    by HiScanner before clustering
  repo: https://github.com/parklab/SCAN2
- name: SECEDO
  role: Independently clusters the cells by somatic point-mutation genotype to validate
    the CNA-based clusters via ARI
- name: Mutect2
  role: Calls somatic SNVs from which SECEDO clusters are derived ('PASS' variants
    only)
- name: BICseq2
  role: Provides the GC-corrected binning and the BIC framework extended by HiScanner
    for joint multi-sample segmentation feeding the copy-number matrix
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:45662f05a2543b8722a3b428c27b4f98f624b4594151989df1d2413d8d1ea3bf
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# hierarchical-clustering-of-somatic-cna-profiles

## Summary

Clusters single tumor cells by their per-cell integer allele-specific copy-number profiles (e.g., a 500 kb-bin matrix from HiScanner) using complete-linkage hierarchical clustering, and validates the recovered clone structure against an independent somatic-SNV-based clustering via Adjusted Rand Index.

## When to use

Use when you have per-cell allele-specific integer copy-number calls across single cells and need to define clone structure, especially to cross-validate CNV clones against SNV-genotype clusters.

## When NOT to use

- Cells are normal/diploid controls with no somatic CNAs, so clustering would split noise rather than clones (e.g., the neurotypical brain cells where aneuploid neurons were excluded from downstream analyses)
- No paired bulk data is available for phasing gHETs and VAF filtering, so HiScanner's RDR+BAF copy numbers cannot be generated and only RDR-only profiles would be clusterable
- The goal is calling CNAs themselves rather than grouping cells — use the HiScanner calling steps instead of clustering

## Inputs

- Per-cell integer allele-specific copy-number matrix (cells × bins, e.g., 500 kb bins from HiScanner '--step cnv' output)
- HiScanner intermediate outputs (gatk/hc_raw.mmq60.vcf.gz, phased gHETs with genotype field 'phasedgt', normalized/segmented per-cell profiles)
- Independent per-cell somatic SNV genotype data (Mutect2 'PASS' variants) for cross-validation clustering

## Outputs

- Dendrogram and cut of complete-linkage hierarchical clustering assigning each single cell to a CNA-based clone cluster
- Cluster assignment table for all cells (e.g., 179 meningioma cells in four clusters)
- Adjusted Rand Index quantifying agreement between CNA-based and SNV-based cluster assignments (expected ~0.83)

## How to apply

First obtain integer allele-specific copy numbers for every cell by running the HiScanner pipeline: run SCAN2 up to the phasing step (--until shapeit/phased_hets.vcf.gz), process with 'hiscanner run --step snp' and '--step phase' (retaining gHETs phasable by SHAPEIT with bulk VAF 0.1–0.9 and reads with MAPQ ≥ 30 on the hg19 100mer mappability track), then '--step ado' to pick the bin size from allelic dropout patterns (500 kb for ~0.5x data), followed by '--step normalize' (optionally --use-cluster), '--step segment' (joint BIC-based multi-sample segmentation) and '--step cnv'. Assemble the per-cell copy-number matrix at the chosen bin size (179 meningioma cells × 500 kb bins in the source study) and apply complete-linkage hierarchical clustering, cutting the tree into the number of clusters supported by the data (four in the meningioma case, corresponding to Clusters 8, 11, 13, and 15, with the grade II cell BOS-EE06 in Cluster 15). Independently cluster the same cells by somatic point-mutation genotype (SECEDO on Mutect2 'PASS' SNVs) and compute the Adjusted Rand Index between the two cluster assignments; an ARI near the reported 0.83 confirms the CNA-based clone structure.

## Related tools

- **HiScanner** (Produces the per-cell integer allele-specific copy-number matrix (snp, phase, ado, normalize, segment, cnv steps) that is clustered) — https://github.com/parklab/HiScanner
- **SCAN2** (Genotypes somatic SNVs and produces the raw variant VCF and phased gHETs consumed by HiScanner before clustering) — https://github.com/parklab/SCAN2
- **SECEDO** (Independently clusters the cells by somatic point-mutation genotype to validate the CNA-based clusters via ARI)
- **Mutect2** (Calls somatic SNVs from which SECEDO clusters are derived ('PASS' variants only))
- **BICseq2** (Provides the GC-corrected binning and the BIC framework extended by HiScanner for joint multi-sample segmentation feeding the copy-number matrix)

## Examples

```
hiscanner run --step cnv && Rscript -e 'hc <- hclust(dist(t(cn_matrix)), method="complete"); cl <- cutree(hc, k=4); ari <- mclust::adjustedRandIndex(cl, snv_clusters)'
```

## Evaluation signals

- Adjusted Rand Index between CNA-based and SNV-based cluster assignments is near 0.83 (as reported for the 179 meningioma cells)
- Recovered clone count matches the structure seen in SNV clustering (four clusters, corresponding to Clusters 8, 11, 13, and 15)
- Known lineage assignment recovered: the grade II cell BOS-EE06 falls in Cluster 15
- Bin size used for the copy-number matrix matches the ADO-derived optimum (500 kb for ~0.5x coverage data)
- Only cells with valid integer allele-specific copy numbers across the 500 kb bin matrix are included in the tree

## Limitations

- The ADO-based bin size is assay-specific; for the ~0.5x meningioma dataset it was 500 kb, and other coverages/amplification methods require re-running '--step ado'
- HiScanner's model assumes independence between single-cell genomes, which may be violated for cells sharing clonal origins
- Low-coverage (~0.5x) data limits resolution to the 500 kb bin scale; smaller CNAs require high-coverage scWGS
- SNV-based validation clustering may conflate true clonal expansions with independent recurrent mutations at identical coordinates

## Evidence

- [methods] workflow step 4: [evidence span withheld — anchor status: not_found]
- [results] finding: [evidence span withheld — anchor status: not_found]
- [discussion] filter: "Only the gHETs that can be phased by SHAPEIT2 and have a variant allele frequency between 0.1 and 0.9 in bulk data were retained"
- [intro] clustering rationale: "HiScanner revealed phases and timing of tumor progression by detecting subclone-defining CNAs, which were subsequently corroborated by an orthogonal clustering based on somatic point mutations"
- [readme] pipeline invocation: "hiscanner run --step ado      # ADO analysis to identify optimal bin size"
