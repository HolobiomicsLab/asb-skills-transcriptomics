---
name: plink-ld-reference-processing
description: Use when training eQTL weights from summary-level data (OTTERS-style
  TWAS Stage I) and you must supply a PLINK 1.9 .bed/.bim/.fam LD reference panel
  plus gene annotation to drive LD clumping and per-gene extraction.
when_to_use_negative:
- The analysis already has individual-level reference genotype-expression data (e.g.
  GTEx V6/V8), where FUSION-style individual-level training is the more direct choice
- The reference panel genotype build or ancestry does not match the eQTL summary statistics
  or the selected lassosum LD blocks (e.g. hg38 genotypes with hg19 blocks)
- The trait association test is already done and you only need Stage II TWAS testing,
  which consumes the trained weights rather than rebuilding the LD reference
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0625
tools:
- name: PLINK 1.9
  role: Extracts per-target-gene genotypes from the PLINK-binary LD reference panel
    (via call_PLINK_extract) and supplies the LD basis for clumping
  repo: https://www.cog-genomics.org/plink
- name: OTTERS
  role: Wraps TABIX and PLINK 1.9 to prepare LD reference and eQTL summary statistics
    in the required input formats for P+T, lassosum, SDPR, and PRS-CS
  repo: https://github.com/daiqile96/OTTERS
- name: TABIX
  role: Extracts per-gene regions from the tabixed eQTL summary statistics (extract_vcf_region_with_tabix)
  repo: http://www.htslib.org/doc/tabix.html
- name: lassosum
  role: Consumes the LD reference panel and pre-defined LD blocks (--lassosum_ld_blocks,
    default EUR.hg38) during weight training with pseudo-validation
  repo: https://github.com/tshmak/lassosum
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:bc871544a86daa35eeb87473b437c9e0f42da21e2314962c6d3fcc52620dae3d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OTTERS
  - PLINK
  - PLINK 1.9
  - PRS-CS
  - SDPR
  - TABIX
  - fdrtool
  - lassosum
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# PLINK LD reference processing

## Summary

Prepare and extract a PLINK-binary LD reference panel and per-gene LD blocks so that OTTERS Stage I can clump cis-eQTLs (r2=0.99) and train eQTL weights with P+T, lassosum, SDPR, and PRS-CS. PLINK 1.9 and TABIX are integrated into OTTERS to slice the reference per target gene.

## When to use

Use when training eQTL weights from summary-level data (OTTERS-style TWAS Stage I) and you must supply a PLINK 1.9 .bed/.bim/.fam LD reference panel plus gene annotation to drive LD clumping and per-gene extraction.

## When NOT to use

- The analysis already has individual-level reference genotype-expression data (e.g. GTEx V6/V8), where FUSION-style individual-level training is the more direct choice
- The reference panel genotype build or ancestry does not match the eQTL summary statistics or the selected lassosum LD blocks (e.g. hg38 genotypes with hg19 blocks)
- The trait association test is already done and you only need Stage II TWAS testing, which consumes the trained weights rather than rebuilding the LD reference

## Inputs

- LD reference panel genotypes in PLINK 1.9 binary format (.bed/.bim/.fam)
- Gene annotation file with columns CHROM, GeneStart, GeneEnd, TargetID
- eQTL summary statistics sorted by chromosome then ascending SNP position
- LD block definitions for lassosum (EUR.hg19/EUR.hg38/ASN/AFR from Berisa & Pickrell 2015)

## Outputs

- Per-gene extracted LD reference genotype matrices and matching eQTL records
- LD-clumped cis-eQTL sets at r2=0.99
- Trained eQTL weights per gene in the OTTERS Results directory (P0.001, P0.05, lassosum, SDPR, PRScs)

## How to apply

Supply the LD reference panel in PLINK 1.9 binary (bed/bim/fam) format alongside a gene annotation file (CHROM, GeneStart, GeneEnd, TargetID) and eQTL summary statistics pre-sorted by chromosome then ascending SNP position. OTTERS integrates TABIX and PLINK 1.9 (via extract_vcf_region_with_tabix and call_PLINK_extract in ottersutils.py) to efficiently extract, per target gene, the LD reference genotypes and matching eQTL records. Set the LD-clumping threshold --r2=0.99 so Stage I retains highly linked cis-eQTLs before fitting the four PRS-based weight models. For lassosum, choose the correct ancestry/build LD blocks via --lassosum_ld_blocks (EUR.hg38 by default; also EUR/ASN/AFR .hg19 or .hg38 from Berisa & Pickrell 2015). Verify the output Results directory contains trained weights for all five model variants (P0.001, P0.05, lassosum, SDPR, PRScs) for every gene on the chromosome.

## Related tools

- **PLINK 1.9** (Extracts per-target-gene genotypes from the PLINK-binary LD reference panel (via call_PLINK_extract) and supplies the LD basis for clumping) — https://www.cog-genomics.org/plink
- **OTTERS** (Wraps TABIX and PLINK 1.9 to prepare LD reference and eQTL summary statistics in the required input formats for P+T, lassosum, SDPR, and PRS-CS) — https://github.com/daiqile96/OTTERS
- **TABIX** (Extracts per-gene regions from the tabixed eQTL summary statistics (extract_vcf_region_with_tabix)) — http://www.htslib.org/doc/tabix.html
- **lassosum** (Consumes the LD reference panel and pre-defined LD blocks (--lassosum_ld_blocks, default EUR.hg38) during weight training with pseudo-validation) — https://github.com/tshmak/lassosum

## Examples

```
python training.py --chrom=4 --gene_anno=anno.txt --eqtl=eqtl_sorted.txt --ldref=EUR.bed --r2=0.99 --models=PT,lassosum,SDPR,PRScs --lassosum_ld_blocks=EUR.hg38
```

## Evaluation signals

- Stage I Results directory contains trained eQTL weights for all five model variants (P0.001, P0.05, lassosum, SDPR, PRScs) for every annotated gene on the chromosome
- LD clumping was run with --r2=0.99 and eQTL summary statistics were sorted by chromosome then ascending SNP position before extraction
- The reference panel matches the requested lassosum LD blocks (e.g. EUR.hg38 from Berisa & Pickrell 2015) in ancestry and genome build
- Per-gene extracted LD genotype rows correspond to the genes' CHROM/GeneStart/GeneEnd windows in the annotation file
- Replication on chr 4 example data reproduces OTTERS STAGE I weights consistent with Stage II TWAS p values and the ACAT-O omnibus p value

## Limitations

- Available eQTL summary statistics are mainly from individuals of European descent, limiting cross-population transferability of the LD reference and TWAS results
- Training GReX imputation models with all five methods costs ~20 min per gene, which is computationally challenging genome-wide (~20K genes) and scales with the LD reference
- OTTERS currently considers only cis-eQTL effects, so the per-gene LD extraction is restricted to the cis window; trans-eQTLs are not incorporated

## Evidence

- [readme] OTTERS integrates TABIX and PLINK 1.9 to efficiently prepare per-gene LD reference inputs: "integrate [TABIX](http://www.htslib.org/doc/tabix.html) and [PLINK 1.9](https://www.cog-genomics.org/plink) tools to extract input data per target gene more efficiently"
- [methods] OTTERS utility functions perform the per-gene LD and eQTL extraction: "See the functions [extract_vcf_region_with_tabix](https://github.com/daiqile96/OTTERS/blob/main/ottersutils.py#L233) and [call_PLINK_extract](https://github.com/daiqile96/OTTERS/blob/main/ottersutils."
- [readme] lassosum LD block selection is exposed as an OTTERS argument with ancestry/build options: [evidence span withheld — anchor status: not_found]
- [readme] PRScs requires LD reference panels that must match the GWAS sample ancestry: [evidence span withheld — anchor status: external_doc]
