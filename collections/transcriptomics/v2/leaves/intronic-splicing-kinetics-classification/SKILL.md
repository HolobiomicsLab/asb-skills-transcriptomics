---
name: intronic-splicing-kinetics-classification
description: Use when you have lead sQTL variants mapped with sQTLseekeR2 plus an
  intron annotation, and need to test whether splicing QTLs localize preferentially
  in post-transcriptionally vs co-transcriptionally spliced introns.
when_to_use_negative:
- Intron set or cell lines with no matched nuclear/cytosolic ENCODE RNA-seq, since
  the ps/cs classification depends on that splicing-kinetics data
- Questions about exon-centric splicing events or junction-level LeafCutter clusters
  rather than intron-centered sQTL localization
- Variant-level effect annotation (e.g. splice-site disruption prediction with LOFTEE)
  rather than enrichment testing of mapped sQTL lead variants
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: sQTLseekeR2
  role: Detects sQTLs whose lead variants are intersected with the ps/cs intron sets
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: Nextflow
  role: Executes the sqtlseeker2-nf pipeline that produces the lead sQTL variants
    used in the enrichment test
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:9436b0d9ea64e3050322ed87420cc905d390952fef17b0e0e48061048b5739f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sqtlseeker2-nf
  tools:
  - Docker
  - Nextflow
  - sQTLseekeR2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# intronic-splicing-kinetics-classification

## Summary

Classify annotated introns as co-transcriptionally spliced (cs) versus post-transcriptionally spliced (ps) using ENCODE nuclear/cytosolic RNA-seq splicing kinetics across 13 cell lines, then test whether sQTLs are enriched in ps introns by Fisher's exact test. This reproduces the paper's finding that splicing-regulatory variants concentrate in introns spliced after transcript release.

## When to use

Use when you have lead sQTL variants mapped with sQTLseekeR2 plus an intron annotation, and need to test whether splicing QTLs localize preferentially in post-transcriptionally vs co-transcriptionally spliced introns.

## When NOT to use

- Intron set or cell lines with no matched nuclear/cytosolic ENCODE RNA-seq, since the ps/cs classification depends on that splicing-kinetics data
- Questions about exon-centric splicing events or junction-level LeafCutter clusters rather than intron-centered sQTL localization
- Variant-level effect annotation (e.g. splice-site disruption prediction with LOFTEE) rather than enrichment testing of mapped sQTL lead variants

## Inputs

- Genotype file with 0/1/2/-1 genotype coding sorted by coordinate
- Transcript expression file with trId and geneId columns
- Metadata file with indId/sampleId/group columns
- Gene location file with chr/start/end/geneId columns
- Intron annotation with coordinates for each tested intron
- ENCODE nuclear and cytosolic RNA-seq data for the 13 cell lines (splicing kinetics)
- eQTL catalog (e.g. GTEx v8) to partition sQTLs into eQTL and non-eQTL subsets

## Outputs

- Per-intron table labeling each tested intron as cs or ps
- 2x2 contingency table of lead sQTL variant counts in ps vs cs introns
- Fisher's exact odds ratio and P-value for all sQTLs (expected OR ~1.47)
- Fisher's exact odds ratio and P-value for sQTLs that are not eQTLs (expected OR ~1.67)

## How to apply

First obtain lead sQTL variants by running the sQTLseekeR2 pipeline (sqtlseeker2-nf via Nextflow with Docker) in nominal mode (win=5000 bp cis window, fdr=0.05, min_md=0.05) and optionally in permuted mode (max_perm=1000) for empirical beta-distribution P-values. Then classify each intron from the annotation as co- vs post-transcriptionally spliced using splicing kinetics derived from ENCODE nuclear and cytosolic RNA-seq across 13 cell lines: introns whose splicing completes before cytosolic appearance are 'cs', those spliced after are 'ps'. Intersect lead sQTL variants with the ps and cs intron sets and compute a 2x2 Fisher's exact test of sQTL counts in ps vs cs introns, repeating the test for the subset of sQTLs that are not eQTLs. The rationale is that introns spliced post-transcriptionally cannot be spliced co-transcriptionally by the polymerase and thus represent hotspots where splicing regulation by trans-acting variation is detectable.

## Related tools

- **sQTLseekeR2** (Detects sQTLs whose lead variants are intersected with the ps/cs intron sets) — https://github.com/guigolab/sqtlseeker2-nf
- **Nextflow** (Executes the sqtlseeker2-nf pipeline that produces the lead sQTL variants used in the enrichment test) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
R> fisher.test(matrix(c(n_ps_sqtl, n_ps_nonsqtl, n_cs_sqtl, n_cs_nonsqtl), nrow=2))  # after NXF_VER=22.04.0 ./nextflow run guigolab/sqtlseeker2-nf -with-docker
```

## Evaluation signals

- Enrichment odds ratio exceeds 1 for ps introns for all sQTLs (paper value: OR = 1.47)
- Non-eQTL sQTL subset shows a stronger enrichment (paper value: OR = 1.67)
- Fisher's exact test reaches genome-wide significance (paper: p < 10-16)
- Percentages of ps vs cs introns harboring sQTLs match the reported 9.2% vs 6.6%
- Every tested intron from the annotation receives exactly one cs/ps label from the ENCODE 13-cell-line kinetics

## Limitations

- The ps/cs classification relies on ENCODE nuclear/cytosolic RNA-seq limited to 13 cell lines, so introns outside those cell types cannot be confidently classified
- A substantial fraction of sQTLs also affects gene expression, so all-sQTL enrichment is partially confounded by eQTLs; the non-eQTL subset controls for this but precludes fine-mapping whether the paired sQTL/eQTL effects share one causal variant
- The multivariate, non-parametric sQTL framework means co-localization and fine-mapping methods are not directly applicable when interpreting overlapping sQTLs and eQTLs

## Evidence

- [results] sQTLs enriched in post-transcriptionally spliced introns: 9.2% vs 6.6%: [evidence span withheld — anchor status: not_found]
- [methods] ps/cs classification method from ENCODE 13 cell lines: [evidence span withheld — anchor status: not_found]
- [abstract] Biological rationale: ps introns as splicing-regulation hotspots: "sQTLs tend to be located in post-transcriptionally spliced introns, which would function as hotspots for splicing regulation."
- [readme] sQTLseekeR2 as the mapping engine feeding the enrichment analysis: [evidence span withheld — anchor status: not_found]
