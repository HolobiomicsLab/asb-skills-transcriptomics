---
name: fdr-multiple-testing-correction
description: Use when calling significant sQTLs (or other high-multiple-testing association
  scans) from a nominal association pass over GTEx-style transcript-expression and
  genotype files, where per-test P-values must be converted to empirical, permutation-calibrated
  FDR values.
when_to_use_negative:
- When the number of tests is small or a per-test error rate is required instead of
  a proportion of false discoveries among calls (FDR is inappropriate for single-hypothesis
  confirmation)
- When only nominal P-values are available without the permutation pass — empirical
  FDR cannot be computed from all-tests.nominal.tsv alone
- 'When fine-mapping or co-localization is the goal: the paper notes currently available
  methods are not directly applicable within the multivariate, non-parametric framework'
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: sQTLseekeR2
  role: R package providing the permutation-based multiple testing correction schema
    and empirical P-value computation for splicing QTL mapping
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow pipeline wrapping sQTLseekeR2; exposes --fdr, --mode permuted, --max_perm
    and --min_md parameters used to threshold calls at 0.05 FDR
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: Nextflow
  role: Workflow engine that executes the nominal and permuted passes reproducibly
    in Docker/Singularity containers
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:ec19637934d0bc77726d71adc3e0fd66abff165d4dcfdb099256ba8db6ad63ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - Docker
  - Nextflow
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# fdr-multiple-testing-correction

## Summary

Controls the false discovery rate in genome-wide splicing QTL (sQTL) mapping by sQTLseekeR2's permutation-based schema: a nominal pass over all gene-variant tests is followed by an adaptive permutation pass (--max_perm 1000) yielding empirical P-values, from which a per-gene FDR is computed and significant associations are called at a user-set threshold (e.g. --fdr 0.05).

## When to use

Use when calling significant sQTLs (or other high-multiple-testing association scans) from a nominal association pass over GTEx-style transcript-expression and genotype files, where per-test P-values must be converted to empirical, permutation-calibrated FDR values.

## When NOT to use

- When the number of tests is small or a per-test error rate is required instead of a proportion of false discoveries among calls (FDR is inappropriate for single-hypothesis confirmation)
- When only nominal P-values are available without the permutation pass — empirical FDR cannot be computed from all-tests.nominal.tsv alone
- When fine-mapping or co-localization is the goal: the paper notes currently available methods are not directly applicable within the multivariate, non-parametric framework

## Inputs

- all-tests.nominal.tsv (per-gene nominal test results from sqtlseeker2-nf result/groups/<group>/)
- all-tests.permuted.tsv (empirical P-values from the adaptive permutation pass)
- Genotype file sorted by coordinate (0/1/2 coding, -1 missing)
- Transcript expression file with trId and geneId columns
- Metadata file with indId, sampleId, group columns plus covariates
- Gene location file with chr, start, end and geneId

## Outputs

- sqtls-0.05fdr.permuted.tsv (significant sQTLs passing the 0.05 FDR threshold)
- Per-tissue aggregated counts of significant sQTLs and sGenes for comparison against catalog totals (e.g. 210,485 sQTLs, 6,963 sGenes for GTEx V7)
- fig3a_enrichment_table.tsv with odds_ratio, p_value and fdr fields per annotation category (enrichment analyses)

## How to apply

Run the sqtlseeker2-nf pipeline (Nextflow with Docker/Singularity) in --mode permuted with --max_perm 1000 so that, after the nominal pass (all-tests.nominal.tsv), each gene phenotype receives an empirical P-value via adaptive permutation written to all-tests.permuted.tsv. Set --fdr 0.05 to threshold the FDR computed across empirical P-values, and optionally apply the minimum effect size filter --min_md 0.05 so only splicing ratios changing by at least 0.05 are reported. For enrichment-style secondary analyses (e.g. sQTL vs matched non-sQTL overlap with splice sites, H3K36me3, GWAS LD variants), generate 1,000 matched null sets and compute two-sided Fisher's exact odds ratios, then apply Benjamini-Hochberg-style FDR correction at 0.05 before ranking annotations. Rationale: sQTLseekeR2's permutation-based schema replaces parametric multiple-testing correction because transcript-expression phenotypes violate standard assumptions and tests per gene vary widely.

## Related tools

- **sQTLseekeR2** (R package providing the permutation-based multiple testing correction schema and empirical P-value computation for splicing QTL mapping) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow pipeline wrapping sQTLseekeR2; exposes --fdr, --mode permuted, --max_perm and --min_md parameters used to threshold calls at 0.05 FDR) — https://github.com/guigolab/sqtlseeker2-nf
- **Nextflow** (Workflow engine that executes the nominal and permuted passes reproducibly in Docker/Singularity containers) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
NXF_VER=22.04.0 nextflow run guigolab/sqtlseeker2-nf -with-docker --mode permuted --max_perm 1000 --fdr 0.05 --min_md 0.05 --genotype gt.vcf.gz --trexp te.tsv --metadata md.tsv --genes genes.tsv --dir out
```

## Evaluation signals

- Each tissue's result/groups/<group>/sqtls-0.05fdr.permuted.tsv exists and contains an fdr column computed across empirical P-values, with all reported values <= 0.05
- Aggregated per-tissue sQTL and sGene counts reproduce the GTEx V7 catalog totals: 210,485 sQTLs affecting 6,963 sGenes (6,685 protein-coding, 278 lincRNAs)
- For enrichment tables (fig3a_enrichment_table.tsv), every reported FDR value for significant categories is below 0.05 and each row has odds_ratio and p_value fields (e.g. donor OR = 12.98, acceptor OR = 12.23)
- The permuted pass ran with --max_perm 1000 so empirical P-values in all-tests.permuted.tsv are present for genes advanced from the nominal pass
- Reported associations respect the minimum effect size filter --min_md 0.05 when enabled (all |md| >= 0.05)

## Limitations

- Recent Nextflow releases no longer support DSL1, so the workflow must be pinned with NXF_VER (e.g. 22.04.0)
- Permutation-based FDR requires the optional permuted pass; skipping it leaves only nominal P-values without valid multiple-testing control
- The minimum effect size filter (--min_md 0.05) trades sensitivity for effect-size stringency and may exclude low-effect sQTLs from the catalog

## Evidence

- [readme] Permutation-based schema for multiple testing correction: [evidence span withheld — anchor status: external_doc]
- [methods] fdr threshold in pipeline: "--min_md MIN_MD             minimum effect size reported (default: 0.05)"
- [other] catalog counts: [evidence span withheld — anchor status: not_found]
- [intro] pipeline availability: "computational workflow named sqtlseeker2-nf, available at https://github.com/guigolab/sqtlseeker2-nf"
