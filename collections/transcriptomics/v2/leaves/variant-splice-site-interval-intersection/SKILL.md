---
name: variant-splice-site-interval-intersection
description: Use when sQTL mapping results (e.g. all-tests.nominal.tsv from sqtlseeker2-nf)
  must be restricted to variants overlapping annotated donor/acceptor splice-site
  intervals so their effect on splice-site strength can be quantified and compared.
when_to_use_negative:
- The analysis targets RNA-binding protein (RBP) motif disruption rather than splice-site
  sequences — the paper notes 'many more modify the binding sites of RNA-binding proteins'
  than splice-site sequences
- Variants are intronic/exonic but far from annotated donor/acceptor motifs (e.g.
  deep intronic or branchpoint effects), which this interval intersection will drop
- No transcript annotation with splice-junction coordinates is available for the genome
  build of the variants
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0798
tools:
- name: sQTLseekeR2
  role: Produces the per-variant nominal sQTL test results (coordinates, snpId, md,
    pv) and FDR-based sQTL labels whose variants are intersected with splice-site
    intervals
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow pipeline that runs sQTLseekeR2 and emits the output folder (all-tests.nominal.tsv,
    sqtls-${level}fdr.nominal.tsv) consumed by the intersection step
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:489b696a527dba006eee2609540cb6071950dd72d78ced671e14beeeb3eafff9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0798
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# variant-splice-site-interval-intersection

## Summary

A method step from the GTEx sQTL catalog analysis (Garrido-Martín et al., Nat Commun 2021) that intersects tested sQTL variant coordinates from sqtlseeker2-nf output with annotated splice-site (donor/acceptor) intervals to retain variants overlapping splice junction motifs, enabling downstream PWM-based splice-strength comparison of sQTLs vs non-sQTLs.

## When to use

Use when sQTL mapping results (e.g. all-tests.nominal.tsv from sqtlseeker2-nf) must be restricted to variants overlapping annotated donor/acceptor splice-site intervals so their effect on splice-site strength can be quantified and compared.

## When NOT to use

- The analysis targets RNA-binding protein (RBP) motif disruption rather than splice-site sequences — the paper notes 'many more modify the binding sites of RNA-binding proteins' than splice-site sequences
- Variants are intronic/exonic but far from annotated donor/acceptor motifs (e.g. deep intronic or branchpoint effects), which this interval intersection will drop
- No transcript annotation with splice-junction coordinates is available for the genome build of the variants

## Inputs

- sQTLseekeR2 nominal test results (all-tests.nominal.tsv: chr, start, end, snpId, md, pv)
- sQTL labels file (sqtls-${level}fdr.nominal.tsv, default FDR 0.05)
- Gene annotation with splice donor and acceptor site coordinates (e.g. Gencode, as used for GTEx v8)

## Outputs

- Subset of tested variants overlapping annotated splice-site intervals (donor/acceptor)
- Variant table annotated with sQTL/non-sQTL status and splice-site type
- Per-variant reference and alternative allele splice-site sequence windows ready for PWM scoring

## How to apply

Load per-variant nominal tests (chr, start, end, snpId, md effect size, pv) from the sqtlseeker2-nf output folder and the sQTL labels from sqtls-${level}fdr.nominal.tsv (FDR threshold 0.05). Build donor and acceptor splice-site intervals from the gene annotation, keeping strand orientation so motifs are extracted in the correct direction. Intersect the tested variant coordinates (chr, start, end) with these intervals to retain only splice-site-overlapping variants, then extract reference and alternative allele sequence windows for scoring with position weight matrices built from human splice sites. The rationale is that Fig. 4a of the paper shows splice-site strength modification is enriched among sQTLs (63% vs 49%, OR = 1.79), so this intersection defines the variant set on which |PWM(alt) − PWM(ref)| is computed, partitioned into sQTLs vs non-sQTLs, and stratified by md effect-size bins.

## Related tools

- **sQTLseekeR2** (Produces the per-variant nominal sQTL test results (coordinates, snpId, md, pv) and FDR-based sQTL labels whose variants are intersected with splice-site intervals) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow pipeline that runs sQTLseekeR2 and emits the output folder (all-tests.nominal.tsv, sqtls-${level}fdr.nominal.tsv) consumed by the intersection step) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
nextflow run guigolab/sqtlseeker2-nf -with-docker
```

## Evaluation signals

- Every retained variant's coordinates (chr, start, end) fall within an annotated donor or acceptor splice-site interval on the correct strand
- Among retained splice-site variants, the odds ratio of strength-altering (|ΔPWM| > 0) variants in sQTLs vs non-sQTLs is greater than 1 (paper: OR = 1.79)
- Fraction of sQTLs modifying splice-site strength (~63%) exceeds that of non-sQTLs (~49%), reproducing Fig. 4a
- Absolute strength change |PWM(alt) − PWM(ref)| is larger for sQTLs than non-sQTLs (paper: Wilcoxon p = 1.98 × 10−7) and increases monotonically across md effect-size bins

## Limitations

- PWM-based strength scores capture only splice-site motif disruption; the paper notes many more sQTLs modify RNA-binding protein binding sites than splice-site sequences, so this intersection misses most splicing mechanisms
- LOFTEE-style caveats apply nearby: apparent splice-site disruption can be rescued by nearby in-frame splice sites (NAGNAG acceptors, rescue within ~15 bp) or lie in non-canonical introns (not starting GT / ending AG), which a plain interval intersection does not screen out
- Variant-to-sQTL causality cannot be guaranteed without LD-clustering/fine-mapping; the paper notes available fine-mapping methods are 'not directly applicable within our multivariate, non-parametric framework'

## Evidence

- [methods] Intersect tested variant coordinates with annotated splice-site intervals (donor and acceptor sites) to retain variants overlapping splice junction motifs.: [evidence span withheld — anchor status: not_found]
- [abstract] While many variants affect splicing patterns by altering the sequence of splice sites, many more modify the binding sites of RNA-binding proteins.: "While many variants affect splicing patterns by altering the sequence of splice sites, many more modify the binding sites of RNA-binding proteins."
- [discussion] our implementation of an enhanced pipeline for sQTL mapping based on sQTLseekeR2, Nextflow and Docker: "our implementation of an enhanced pipeline for sQTL mapping based on sQTLseekeR2, Nextflow and Docker"
- [readme] Test for association between splicing ratios and genetic variants in *cis* (nominal pass): "Test for association between splicing ratios and genetic variants in *cis* (nominal pass)"
- [readme] Variants where MaxEntScan does not predict an effect on splicing: [evidence span withheld — anchor status: external_doc]
