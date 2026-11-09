---
name: sqtls-fdr-threshold-classification
description: Use when you have sqtlseeker2-nf sQTL mapping output (nominal per-variant
  tests plus FDR-significant sQTL tables) and need binary sQTL/non-sQTL labels for
  downstream functional-enrichment comparisons such as splice-site PWM strength analysis.
when_to_use_negative:
- You only have nominal p-values without the FDR-selected sQTL table (sqtls-${level}fdr.nominal.tsv)
  — thresholding raw p-values is not the paper's multiple-testing scheme
- Your question concerns eQTL/eGene classification or gene expression, not splicing
  phenotypes; this skill only labels splicing QTLs from sQTLseekeR2 output
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: sQTLseekeR2
  role: R package performing the association tests and permutation-based multiple
    testing correction whose output tables (nominal and FDR-selected sQTLs) supply
    the labels
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow pipeline that produces all-tests.nominal.tsv and sqtls-${level}fdr.nominal.tsv,
    the files from which sQTL/non-sQTL labels are drawn
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: Nextflow
  role: Workflow engine executing the reproducible sQTL mapping pipeline that generates
    the labeled output
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:6c65d034c6415e3c8d15afdfc4900a91b559ee7d110f73dfe10f2fc42f272ff9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - Nextflow
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# sqtls-fdr-threshold-classification

## Summary

Classify tested variants as sQTLs versus non-sQTLs by joining per-variant nominal tests from the sqtlseeker2-nf output (all-tests.nominal.tsv) with the FDR-selected sQTL labels in sqtls-${level}fdr.nominal.tsv (default FDR 0.05), so downstream analyses can contrast the two variant sets.

## When to use

Use when you have sqtlseeker2-nf sQTL mapping output (nominal per-variant tests plus FDR-significant sQTL tables) and need binary sQTL/non-sQTL labels for downstream functional-enrichment comparisons such as splice-site PWM strength analysis.

## When NOT to use

- You only have nominal p-values without the FDR-selected sQTL table (sqtls-${level}fdr.nominal.tsv) — thresholding raw p-values is not the paper's multiple-testing scheme
- Your question concerns eQTL/eGene classification or gene expression, not splicing phenotypes; this skill only labels splicing QTLs from sQTLseekeR2 output

## Inputs

- all-tests.nominal.tsv from the sqtlseeker2-nf output folder (per-variant tests with chr, start, end, snpId, md effect size, pv)
- sqtls-${level}fdr.nominal.tsv from the sqtlseeker2-nf output folder (FDR-selected sQTLs, default FDR 0.05)

## Outputs

- Variant-level table with binary sQTL / non-sQTL labels plus md effect size
- Partitioned variant sets (sQTL vs non-sQTL) ready for enrichment or distributional comparisons (e.g. odds ratio of strength-altering variants, effect-size-binned analyses)

## How to apply

Load the nominal results from the sqtlseeker2-nf output folder using all-tests.nominal.tsv, which carries per-variant columns (chr, start, end, snpId, md effect size, pv), and load sqtls-${level}fdr.nominal.tsv (default FDR threshold 0.05) to obtain the set of significant sQTLs. Assign each tested variant the binary label sQTL if it appears in the FDR-passing table and non-sQTL otherwise; the nominal table preserves the md effect size for each variant so you can additionally stratify by effect size bins. The labels are then used as the grouping variable for downstream contrasts — for example, comparing splice-site strength alteration (odds ratio) or |PWM(alt) − PWM(ref)| distributions between sQTLs and non-sQTLs. The underlying pipeline computes these FDR labels via a permutation-based multiple-testing correction schema in sQTLseekeR2, so nominal p-values should never be thresholded directly as a substitute for the FDR-selected set.

## Related tools

- **sQTLseekeR2** (R package performing the association tests and permutation-based multiple testing correction whose output tables (nominal and FDR-selected sQTLs) supply the labels) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow pipeline that produces all-tests.nominal.tsv and sqtls-${level}fdr.nominal.tsv, the files from which sQTL/non-sQTL labels are drawn) — https://github.com/guigolab/sqtlseeker2-nf
- **Nextflow** (Workflow engine executing the reproducible sQTL mapping pipeline that generates the labeled output) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
./nextflow run guigolab/sqtlseeker2-nf -with-docker
```

## Evaluation signals

- Every variant labeled sQTL must appear in the sqtls-${level}fdr.nominal.tsv file at the chosen FDR level (0.05); no labeled sQTL may be missing from it
- The union of sQTL and non-sQTL sets must equal the variant set in all-tests.nominal.tsv with no overlaps
- Labeled sQTLs must have lower nominal p-values than non-sQTLs overall, consistent with FDR selection from the nominal pass
- Downstream sanity checks reproduce the paper's contrasts, e.g. splice-site strength-altering variants enriched among sQTLs with odds ratio > 1 (reported OR = 1.79, 63% of sQTLs vs 49% of non-sQTLs)
- The md effect size column retained from all-tests.nominal.tsv should support the expected monotonic trend of absolute strength change with sQTL effect size

## Limitations

- The FDR label is a binary threshold at the chosen level; variants just above the cutoff are not necessarily 'no effect'
- Co-localization and fine-mapping methods are not directly applicable within the multivariate non-parametric framework, so labels cannot disentangle causal vs linked variants
- The sQTL vs non-sQTL contrast can be sensitive to the number of variants actually tested per gene; nominal results only cover tested variants

## Evidence

- [full_text] Load the sQTL mapping results from the sqtlseeker2-nf output folder, using all-tests.nominal.tsv: [evidence span withheld — anchor status: not_found]
- [discussion] our implementation of an enhanced pipeline for sQTL mapping based on sQTLseekeR2, Nextflow and Docker: "our implementation of an enhanced pipeline for sQTL mapping based on sQTLseekeR2, Nextflow and Docker"
- [readme] README pipeline steps: "Test for association between splicing ratios and genetic variants in *cis* (nominal pass)"
- [readme] README multiple testing: [evidence span withheld — anchor status: external_doc]
