---
name: baf-binarization-at-germline-hets
description: Use when single-cell scWGS BAF data must be converted to a clean allelic
  state matrix at reliable germline SNPs — e.g., before ADO-event HMM decoding or
  allele-specific CNA calling — and phased gHET calls from SHAPEIT2 (shapeit/phased_hets.vcf)
  are available.
when_to_use_negative:
- Input SNPs are somatic rather than germline heterozygous calls — SCAN2 explicitly
  excludes variants with any read support in matched bulk, and binarization assumes
  known germline genotypes.
- The dataset lacks paired bulk WGS for phasing; without SHAPEIT2-phased gHETs (e.g.,
  low-coverage scWGS without bulk), CHISEL-style block phasing should be used instead.
- Analysis targets sex chromosomes or non-autosomal regions, since HiScanner's analysis
  is restricted to autosomal chromosomes.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: HiScanner
  role: Consumes the binarized BAF matrix at gHETs; the --step ado and --step phase
    subcommands operate on these inputs for ADO and phasing analysis
  repo: https://github.com/parklab/HiScanner
- name: SHAPEIT2
  role: Phases the bulk-called gHETs using the 1000 Genomes Phase 3 integrated haplotype
    panel; only SHAPEIT2-phased gHETs are retained for binarization
- name: GATK HaplotypeCaller
  role: Calls germline heterozygous SNPs on bulk sequencing data with default 'Best
    Practices' parameters, defining the SNP sites for binarization
- name: SCAN2
  role: Runs separately before HiScanner; provides the upstream somatic/germline variant
    context for the pipeline feeding this skill
  repo: https://github.com/parklab/SCAN2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:c2d78085f36debedc3772b62124cd7feb251b0fc5eb761d56fde6f56945b3e26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/parklab/HiScanner
  tools:
  - GATK HaplotypeCaller
  - HiScanner
  - SCAN2
  - SHAPEIT2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# BAF binarization at phased germline heterozygous SNPs (gHETs)

## Summary

Prepares the per-cell B-allele-frequency matrix that downstream HiScanner ADO/CNA modeling consumes: only germline heterozygous SNPs phased by SHAPEIT2 with bulk VAF between 0.1 and 0.9 are retained, and each single cell's BAF at these sites is binarized into allele-specific states.

## When to use

Use when single-cell scWGS BAF data must be converted to a clean allelic state matrix at reliable germline SNPs — e.g., before ADO-event HMM decoding or allele-specific CNA calling — and phased gHET calls from SHAPEIT2 (shapeit/phased_hets.vcf) are available.

## When NOT to use

- Input SNPs are somatic rather than germline heterozygous calls — SCAN2 explicitly excludes variants with any read support in matched bulk, and binarization assumes known germline genotypes.
- The dataset lacks paired bulk WGS for phasing; without SHAPEIT2-phased gHETs (e.g., low-coverage scWGS without bulk), CHISEL-style block phasing should be used instead.
- Analysis targets sex chromosomes or non-autosomal regions, since HiScanner's analysis is restricted to autosomal chromosomes.

## Inputs

- Phased germline heterozygous SNP calls (SHAPEIT2 output, e.g. shapeit/phased_hets.vcf) from paired bulk WGS
- Bulk WGS BAM/VCF for GATK HaplotypeCaller germline SNP calling
- Single-cell scWGS BAM files with allele-specific reads at gHET sites
- 1000 Genomes Phase 3 integrated haplotype reference panel for phasing

## Outputs

- Per-cell binarized BAF matrix at filtered gHET sites (allelic state per cell per SNP)
- Filtered set of SHAPEIT2-phased gHETs with bulk VAF in [0.1, 0.9] used for downstream ADO/CNA modeling

## How to apply

Call germline heterozygous SNPs on the paired bulk WGS with the GATK HaplotypeCaller 'Best Practices' pipeline and phase them with SHAPEIT2 using the 1000 Genomes Phase 3 integrated haplotype panel. Filter the phased gHETs to those with bulk variant allele frequency between 0.1 and 0.9, since sites near these bounds are likely genotype errors or uninformative for dropout detection. For each site and each cell, compute BAF from allele-specific read counts at the SNP and binarize it into two allelic states (copy of each haplotype observed or not), excluding sites the cell does not cover. Restrict to autosomes and mappable, uniquely aligned reads (MAPQ ≥ 30) so that RDR and BAF features remain consistent across cells. The resulting binarized BAF matrix at gHETs is the input to HiScanner's ADO HMM (run --step ado), which groups consecutive ADO-state sites into dropout events to derive the assay-specific optimal bin size.

## Related tools

- **HiScanner** (Consumes the binarized BAF matrix at gHETs; the --step ado and --step phase subcommands operate on these inputs for ADO and phasing analysis) — https://github.com/parklab/HiScanner
- **SHAPEIT2** (Phases the bulk-called gHETs using the 1000 Genomes Phase 3 integrated haplotype panel; only SHAPEIT2-phased gHETs are retained for binarization)
- **GATK HaplotypeCaller** (Calls germline heterozygous SNPs on bulk sequencing data with default 'Best Practices' parameters, defining the SNP sites for binarization)
- **SCAN2** (Runs separately before HiScanner; provides the upstream somatic/germline variant context for the pipeline feeding this skill) — https://github.com/parklab/SCAN2

## Examples

```
hiscanner run --step phase    # Process SCAN2 results
```

## Evaluation signals

- Retained gHET set contains only SNPs phased by SHAPEIT2 with bulk variant allele frequency strictly between 0.1 and 0.9.
- Every binarized BAF value is one of two allelic states per cell; cells with no read coverage at a gHET must be excluded rather than assigned an arbitrary state.
- The gHET sites derive from GATK HaplotypeCaller calls on the matched bulk sample, not from single-cell data.
- The binarized matrix, when fed to hiscanner --step ado, yields a dropout-size distribution whose 95th percentile falls near ~100 kb for PTA-amplified cells (1278-Oligo-5 control) and ~10^5 bp-scale for MDA-amplified cells (1278_ct_p1G9 control).
- Analysis is restricted to autosomal chromosomes and uniquely mappable reads with mapping quality ≥ 30.

## Limitations

- Only gHETs phaseable by SHAPEIT2 and with bulk VAF in [0.1, 0.9] are usable, so regions with sparse phasing yield sparse binarized BAF coverage.
- The binarized BAF feeds an ADO-based choice of smallest permissible bin size; residual amplification artifacts can still confound downstream CNA detection even at high sequencing depth.
- Analysis restricted to autosomes limits applicability to sex-chromosome aneuploidy questions.

## Evidence

- [discussion] VAF filter and SHAPEIT2 phasing requirement: "Only the gHETs that can be phased by SHAPEIT2 and have a variant allele frequency between 0.1 and 0.9 in bulk data were retained"
- [discussion] Germline SNP calling with GATK on bulk data: "The GATK HaplotypeCaller “Best Practices” pipeline40 with default parameters was used to call SNPs on bulk sequencing data from each individual"
- [discussion] Phasing reference panel: "SHAPEIT248 was applied to phase the gHETs using the 1000 Genomes Phase 3 integrated haplotype panel49"
- [methods] Pipeline step handling phased results: "hiscanner run --step phase    # Process SCAN2 results"
- [discussion] Read-level quality filter applied to input reads: "Only uniquely mappable reads with a minimum mapping quality of 30 were considered"
