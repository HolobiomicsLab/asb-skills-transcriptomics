---
name: phased-heterozygous-snp-baf-estimation
description: Use when scWGS analysis requires allele-specific copy-number states or
  BAF-based validation, and you have single-cell BAMs plus a matched bulk WGS sample
  per individual to source and phase gHETs.
when_to_use_negative:
- 'Analysis targets sex chromosomes: SCAN2 and the phased-gHET workflow support only
  diploid human autosomes 1–22 (female X in principle)'
- Only gHETs with bulk VAF outside 0.1–0.9 are available (no properly phased, heterozygous
  anchor sites), making BAF estimation unreliable
- No matched bulk WGS exists for the individual, so gHETs cannot be called and phased
  against a germline reference
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: HiScanner
  role: Consumes the phased gHETs to compute bin-wise pBAF jointly with read-depth
    ratios for allele-specific CNA calling; pipeline steps snp/phase prepare its phasing
    input
  repo: github.com/parklab/HiScanner
- name: SCAN2
  role: Prerequisite genotyper run before HiScanner; its output sites feed the snp/phase
    steps
  repo: https://github.com/parklab/SCAN2
- name: Ginkgo
  role: Downstream BAF-agnostic CNA caller whose read-depth-only calls are validated
    against the phased-gHET local BAF (97% oligodendrocyte / 66% neuron discordance)
  repo: https://github.com/robertaboukhalil/ginkgo
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:f77d1cd5815fc6b2ab385536a23bbe98c6f851dee3be8afbf9f8da0a8decb757
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Ginkgo
  - HiScanner
  - R
  - SCAN2
  - bcftools
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# phased-heterozygous-snp-baf-estimation

## Summary

Establishes a set of bulk-phased germline heterozygous SNPs (gHETs) and derives B-allele frequencies (BAF) at those sites so that allele-specific copy-number callers like HiScanner (and validation of BAF-agnostic callers like Ginkgo) can use allelic imbalance evidence from single-cell WGS.

## When to use

Use when scWGS analysis requires allele-specific copy-number states or BAF-based validation, and you have single-cell BAMs plus a matched bulk WGS sample per individual to source and phase gHETs.

## When NOT to use

- Analysis targets sex chromosomes: SCAN2 and the phased-gHET workflow support only diploid human autosomes 1–22 (female X in principle)
- Only gHETs with bulk VAF outside 0.1–0.9 are available (no properly phased, heterozygous anchor sites), making BAF estimation unreliable
- No matched bulk WGS exists for the individual, so gHETs cannot be called and phased against a germline reference

## Inputs

- Matched bulk WGS BAM (one per brain/individual) for germline SNP calling
- Single-cell PTA-amplified scWGS BAM files for BAF computation
- 1000 Genomes Phase 3 integrated haplotype reference panel for SHAPEIT2 phasing
- SCAN2 genotyping output for somatic/germline site annotation (pre-step in the HiScanner pipeline)

## Outputs

- Per-individual set of phased gHETs with bulk VAF in 0.1–0.9 and a 'phasedgt' genotype field
- Bin-wise pBAF values along the genome (autosomes) per single cell
- Local BAF values per candidate CNA region used for concordance/discordance classification

## How to apply

Call germline SNPs on the bulk WGS of each individual with the GATK HaplotypeCaller 'Best Practices' pipeline using default parameters, then phase the gHETs with SHAPEIT2 against the 1000 Genomes Phase 3 integrated haplotype panel. Retain only gHETs that SHAPEIT2 could phase and whose variant allele frequency lies between 0.1 and 0.9 in the bulk data, and ensure the phased genotype field is named 'phasedgt' so HiScanner can consume it (autosomal chromosomes only). Within HiScanner this phasing is run via the pipeline steps 'hiscanner run --step snp' (check SCAN2 results) and 'hiscanner run --step phase' (process SCAN2 results), after which bin-wise pBAF is computed from the single-cell BAMs at phased sites. The rationale is that reference-based phasing converts raw allelic ratios into coherent haplotype blocks, letting a caller distinguish copy-neutral LOH and allele-specific gains/losses from allelic dropout; the same phased gHETs and local BAF computation are reused to test whether read-depth-only calls (e.g., Ginkgo) deviate from 0.5 in the direction implied by their copy-number state.

## Related tools

- **HiScanner** (Consumes the phased gHETs to compute bin-wise pBAF jointly with read-depth ratios for allele-specific CNA calling; pipeline steps snp/phase prepare its phasing input) — github.com/parklab/HiScanner
- **SCAN2** (Prerequisite genotyper run before HiScanner; its output sites feed the snp/phase steps) — https://github.com/parklab/SCAN2
- **Ginkgo** (Downstream BAF-agnostic CNA caller whose read-depth-only calls are validated against the phased-gHET local BAF (97% oligodendrocyte / 66% neuron discordance)) — https://github.com/robertaboukhalil/ginkgo

## Examples

```
hiscanner run --step snp && hiscanner run --step phase
```

## Evaluation signals

- All retained gHETs have bulk VAF within 0.1–0.9, a SHAPEIT2 phasing assignment, and a genotype field named 'phasedgt'
- Only autosomal chromosomes are present in the phased site set; sex chromosomes are excluded
- Balanced regions show local BAF near 0.5; known allele-specific gains/losses shift BAF away from 0.5 in the direction implied by the copy-number state
- Ginkgo validation reproduces reported discordance fractions: 97% (581/599) of oligodendrocyte calls and 66% (1410/2121) of neuronal calls BAF-discordant
- hiscanner validate passes after config.yaml points to the phased SCAN2 output and input BAMs

## Limitations

- Phasing accuracy depends on SHAPEIT2 against the 1000 Genomes Phase 3 panel; unphasable gHETs must be dropped, reducing BAF coverage
- Allelic dropout in amplified single cells distorts BAF, which HiScanner models via its two-state ADO/non-ADO HMM to choose a minimum bin size
- BAF-based concordance cannot rescue read-depth calls overlapping germline/bulk CNAs, which are removed before comparison

## Evidence

- [discussion] bulk VAF filter for gHETs: "Only the gHETs that can be phased by SHAPEIT2 and have a variant allele frequency between 0.1 and 0.9 in bulk data were retained"
- [discussion] bulk SNP calling pipeline: "The GATK HaplotypeCaller “Best Practices” pipeline40 with default parameters was used to call SNPs on bulk sequencing data from each individual"
- [discussion] phasing reference panel: "SHAPEIT248 was applied to phase the gHETs using the 1000 Genomes Phase 3 integrated haplotype panel49"
- [methods] HiScanner phasing step: "hiscanner run --step phase    # Process SCAN2 results"
- [discussion] autosomal restriction: "To maintain consistency across samples from both sexes, our analysis was restricted to autosomal chromosomes"
