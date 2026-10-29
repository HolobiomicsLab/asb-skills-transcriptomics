---
name: genomic-variant-interval-overlap
description: Use when you have a list of genetic variants (e.g. lead sQTLs from sQTLseekeR2)
  and two disjoint genomic interval sets, and need to test whether variants are preferentially
  located in one set (e.g. ps introns) versus the other (cs introns).
when_to_use_negative:
- When the goal is fine-mapping or colocalization of sQTL/eQTL effects — the article
  notes available methods are 'not directly applicable within our multivariate, non-parametric
  framework' and LD confounds paired-effect interpretation
- When testing enrichment of variant properties (e.g. RBP binding site disruption)
  rather than overlap with interval sets — this skill only handles counts of variants
  per interval class
- When the variant-to-interval statistic should account for LD structure between lead
  variants — the default pipeline runs with LD-based variant clustering disabled (ld
  threshold 0)
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: sQTLseekeR2
  role: R package that maps the sQTL lead variants subsequently intersected with ps/cs
    intron sets; provides improved P-value precision and permutation-based multiple
    testing correction
  repo: https://github.com/guigolab/sQTLseekeR2
- name: Nextflow / sqtlseeker2-nf
  role: Reproducible workflow engine wrapping the sQTL mapping steps (indexing, preprocessing,
    cis nominal test, permutation pass) whose lead variant output feeds the interval
    overlap analysis
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: Docker
  role: Container runtime used to execute the sqtlseeker2-nf pipeline reproducibly
    (-with-docker)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:2f9f6aa5685b68b622859b9dd9b0ddf2071974cc49d4537fa0b9ba77cf8797df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - Docker
  - Nextflow / sqtlseeker2-nf
  - sQTLseekeR2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Genomic variant interval overlap

## Summary

Intersects mapped sQTL lead variants with curated genomic interval sets — here co- vs post-transcriptionally spliced (cs/ps) introns defined from ENCODE nuclear/cytosolic RNA-seq splicing kinetics — and uses Fisher's exact test to quantify enrichment of variants in one interval class.

## When to use

Use when you have a list of genetic variants (e.g. lead sQTLs from sQTLseekeR2) and two disjoint genomic interval sets, and need to test whether variants are preferentially located in one set (e.g. ps introns) versus the other (cs introns).

## When NOT to use

- When the goal is fine-mapping or colocalization of sQTL/eQTL effects — the article notes available methods are 'not directly applicable within our multivariate, non-parametric framework' and LD confounds paired-effect interpretation
- When testing enrichment of variant properties (e.g. RBP binding site disruption) rather than overlap with interval sets — this skill only handles counts of variants per interval class
- When the variant-to-interval statistic should account for LD structure between lead variants — the default pipeline runs with LD-based variant clustering disabled (ld threshold 0)

## Inputs

- Genotype file with 0/1/2/-1 genotype coding, sorted by coordinate (tabix-indexed)
- Transcript expression file with trId and geneId columns
- Metadata file with indId/sampleId/group columns
- Gene location file with chr/start/end/geneId columns
- Intron annotation classified into cs and ps intron sets using ENCODE nuclear/cytosolic RNA-seq splicing kinetics
- Lead sQTL variant list with genomic coordinates (from nominal and permuted sQTLseekeR2 runs)

## Outputs

- Contingency table of sQTL counts in ps vs cs introns
- Fisher's exact test odds ratio and P-value for all sQTLs
- Fisher's exact test odds ratio and P-value for the sQTLs that are not eQTLs
- Per-intron splicing kinetics classification (ps vs cs)

## How to apply

First obtain lead sQTL variants by running the sqtlseeker2-nf pipeline (Nextflow + Docker) in nominal mode (cis window win=5000 bp, fdr=0.05, min_md=0.05) and optionally in permuted mode (max_perm=1000) to derive empirical P-values from the fitted beta distribution. Independently classify each tested intron from the annotation as co-transcriptionally (cs) or post-transcriptionally (ps) spliced using ENCODE nuclear/cytosolic RNA-seq splicing kinetics across the 13 cell lines. Intersect the lead sQTL variant coordinates with the ps and cs intron sets, count sQTLs falling in each class, and run Fisher's exact test of sQTL counts in ps vs cs introns — separately for all sQTLs and for the subset of sQTLs that are not eQTLs. Correct interpretation requires reporting an odds ratio: the paper reports 9.2% of variants in ps introns are sQTLs vs 6.6% in cs introns (OR = 1.47, p < 10-16), rising to OR = 1.67 for sQTLs that are not eQTLs.

## Related tools

- **sQTLseekeR2** (R package that maps the sQTL lead variants subsequently intersected with ps/cs intron sets; provides improved P-value precision and permutation-based multiple testing correction) — https://github.com/guigolab/sQTLseekeR2
- **Nextflow / sqtlseeker2-nf** (Reproducible workflow engine wrapping the sQTL mapping steps (indexing, preprocessing, cis nominal test, permutation pass) whose lead variant output feeds the interval overlap analysis) — https://github.com/guigolab/sqtlseeker2-nf
- **Docker** (Container runtime used to execute the sqtlseeker2-nf pipeline reproducibly (-with-docker))

## Examples

```
NXF_VER=22.04.0 ./nextflow run guigolab/sqtlseeker2-nf -with-docker
```

## Evaluation signals

- Odds ratio for sQTL enrichment in ps introns exceeds 1, reproducing the reported OR = 1.47 for all sQTLs
- Odds ratio for the sQTL-not-eQTL subset exceeds that of all sQTLs, approaching the reported OR = 1.67
- Fisher's exact test achieves p < 10-16 for the all-sQTLs contrast (9.2% sQTLs in ps introns vs 6.6% in cs introns)
- Every lead sQTL variant in the intersection output has valid chromosome/coordinate assignments against both the ps and cs intron interval files
- Permuted run returns empirical P-values for each phenotype based on the fitted beta distribution, confirming the permutation pass completed before variant selection

## Limitations

- Co- vs post-transcriptional splicing classification depends on ENCODE nuclear/cytosolic RNA-seq kinetics in only 13 cell lines, limiting generalizability of the intron sets
- The enrichment is an overlap-based association: it cannot establish that a variant's effect is caused by its position within a ps intron
- Inability to distinguish whether paired sQTL/eQTL effects come from the same variant or two variants in LD affects the non-eQTL sQTL subset definition
- Pipeline defaults (min_md=0.05 effect size filter, LD clustering off) shape the lead-variant list and thus the overlap counts

## Evidence

- [abstract] sQTLs tend to be located in post-transcriptionally spliced introns, which would function as hotspots for splicing regulation.: "sQTLs tend to be located in post-transcriptionally spliced introns, which would function as hotspots for splicing regulation."
- [methods] minimum effect size filter default: "--min_md MIN_MD             minimum effect size reported (default: 0.05)"
- [readme] permuted pass in pipeline README: "Obtain an empirical P-value for each phenotype (permutation pass, optional)"
- [readme] permutation schema in sQTLseekeR2 README: [evidence span withheld — anchor status: external_doc]
- [abstract] many variants affect RBP binding sites: "While many variants affect splicing patterns by altering the sequence of splice sites, many more modify the binding sites of RNA-binding proteins."
