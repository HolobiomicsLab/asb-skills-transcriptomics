---
name: single-cell-copy-number-calling
description: Use when calling allele-specific somatic CNAs from paired single-cell
  + matched bulk WGS BAMs (PTA-amplified, ~20-30x scWGS), when small (200 kb-5 Mb)
  or subclonal CNAs must be resolved, or when benchmarking single-cell CNA callers.
when_to_use_negative:
- Data are low-coverage (<0.05x per cell) single-cell cancer datasets from DLP+ or
  10x Genomics platforms — CHISEL/Alleloscope-style callers designed for clonal CNAs
  are the intended setting, while HiScanner exploits high-coverage (>20x) scWGS
- Input lacks a matched bulk sample or phased germline heterozygous SNPs — the BAF/phasing
  steps require phased gHETs and bulk VAF 0.1-0.9
- Calling germline variants or working on sex chromosomes — SCAN2 excludes mutations
  with any bulk read support and only diploid autosomes (1-22) are supported
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: HiScanner
  role: 'Primary CNA caller: phasing, ADO-based bin-size selection, normalization,
    joint BIC segmentation, and joint RDR/BAF copy-number inference'
  repo: https://github.com/parklab/HiScanner
- name: SCAN2
  role: Upstream somatic SNV genotyper for PTA-amplified single cells; must be run
    before HiScanner and stopped at the phasing step
  repo: https://github.com/parklab/SCAN2
- name: Ginkgo
  role: BAF-agnostic comparator caller run on the same single-cell BAM for RDR-based
    copy-number profiles in benchmarking
  repo: https://github.com/robertaboukhalil/ginkgo
- name: CHISEL
  role: BAF-aware comparator caller (bioconda package) run at 100 kb and 5 Mb bin
    settings for method comparison
  repo: https://github.com/raphael-group/chisel
- name: SCOPE
  role: BAF-agnostic R comparator caller run at 100 kb resolution with MAPQ threshold
    40
- name: SHAPEIT2
  role: Phases gHETs against the 1000 Genomes Phase 3 panel before HiScanner's phase
    step
- name: BICseq2
  role: Basis for GC correction/binning (BICseq2-norm) and the joint multi-sample
    BIC segmentation (BICseq2-seg extension)
  repo: https://www.math.pku.edu.cn/teachers/xirb/downloads/software/BICseq2/BICseq2.html
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:4fb51d1b70f55c20274ebec2472884c2e34b51aa224575a5f438b4a10f11b73b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/parklab/HiScanner
  tools:
  - BICseq2
  - CHISEL
  - Ginkgo
  - HiScanner
  - Mutect2
  - R
  - SCAN2
  - SCOPE
  - SCYN
  - SECEDO
  - SHAPEIT
  - SHAPEIT2
  - bcftools
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 21
---

# single-cell-copy-number-calling

## Summary

A high-resolution, allele-specific somatic CNA calling workflow for single-cell WGS that combines read depth (RDR), B-allele frequency (BAF), and reference-based haplotype phasing via HiScanner, benchmarked against Ginkgo, SCOPE, SCYN, and CHISEL on PTA-amplified high-coverage scWGS data.

## When to use

Use when calling allele-specific somatic CNAs from paired single-cell + matched bulk WGS BAMs (PTA-amplified, ~20-30x scWGS), when small (200 kb-5 Mb) or subclonal CNAs must be resolved, or when benchmarking single-cell CNA callers.

## When NOT to use

- Data are low-coverage (<0.05x per cell) single-cell cancer datasets from DLP+ or 10x Genomics platforms — CHISEL/Alleloscope-style callers designed for clonal CNAs are the intended setting, while HiScanner exploits high-coverage (>20x) scWGS
- Input lacks a matched bulk sample or phased germline heterozygous SNPs — the BAF/phasing steps require phased gHETs and bulk VAF 0.1-0.9
- Calling germline variants or working on sex chromosomes — SCAN2 excludes mutations with any bulk read support and only diploid autosomes (1-22) are supported

## Inputs

- PTA-amplified single-cell WGS BAM files (hg19/GRCh37+decoy aligned)
- Matched bulk whole-genome sequencing BAM files (~30x)
- SCAN2 output VCFs (gatk/hc_raw.mmq60.vcf.gz and phased gHETs with 'phasedgt' genotype field)
- Split reference genome FASTA
- 100mer mappability track (hg19CRG.100bp from BICseq2, or hg38/GRCh38 150mer track)
- 1000 Genomes Phase 3 integrated haplotype panel (for SHAPEIT2 phasing)
- config.yaml and metadata.txt for the HiScanner project

## Outputs

- Integer allele-specific per-cell copy-number states at 100 kb (or 500 kb) bin resolution
- Per-cell RDR and BAF profiles after GC-bias normalization and joint BIC segmentation
- Cell ploidy estimates and inferred CNA calls (gain/loss/cnLOH) per cell
- Per-cell CNA catalog with counts, size distribution, clonal vs subclonal classification, and FDR estimate
- Copy-number matrix suitable for downstream clustering (e.g., 500 kb matrix for meningioma clone assignment)

## How to apply

Run SCAN2 on the paired bulk and single-cell BAMs, stopping at phasing (gatk/hc_raw.mmq60.vcf.gz plus SHAPEIT2-phased gHETs with the genotype field renamed 'phasedgt'; retain gHETs with bulk VAF 0.1-0.9). Initialize the HiScanner project with 'hiscanner init' (config.yaml pointing to single-cell/bulk BAMs, split reference genome, 100mer mappability track, 100 kb bins, autosomes only) and validate it. Execute the pipeline steps in order: 'hiscanner run --step snp' and '--step phase' to process SCAN2 results (MAPQ >= 30, uniquely mappable reads), '--step ado' to fit a two-state ADO/non-ADO HMM (Baum-Welch, Gaussian emissions) and select the assay-specific bin size (e.g., 100 kb for high-coverage, 500 kb for ~0.5x data), '--step normalize' for GC-corrected binning (BICseq2-norm style, optionally with --use-cluster), '--step segment' for joint multi-sample BIC-based segmentation (extension of BICseq2-seg), and '--step cnv' to infer cell ploidy and integer allele-specific copy states by joint RDR/BAF likelihood modeling (extension of CHISEL's likelihood method with a GMM on 'balanced' bins). Post-process by removing single-cell calls overlapping bulk CNA calls and excluding aneuploid cells from size/prevalence statistics.

## Related tools

- **HiScanner** (Primary CNA caller: phasing, ADO-based bin-size selection, normalization, joint BIC segmentation, and joint RDR/BAF copy-number inference) — https://github.com/parklab/HiScanner
- **SCAN2** (Upstream somatic SNV genotyper for PTA-amplified single cells; must be run before HiScanner and stopped at the phasing step) — https://github.com/parklab/SCAN2
- **Ginkgo** (BAF-agnostic comparator caller run on the same single-cell BAM for RDR-based copy-number profiles in benchmarking) — https://github.com/robertaboukhalil/ginkgo
- **CHISEL** (BAF-aware comparator caller (bioconda package) run at 100 kb and 5 Mb bin settings for method comparison) — https://github.com/raphael-group/chisel
- **SCOPE** (BAF-agnostic R comparator caller run at 100 kb resolution with MAPQ threshold 40)
- **SHAPEIT2** (Phases gHETs against the 1000 Genomes Phase 3 panel before HiScanner's phase step)
- **BICseq2** (Basis for GC correction/binning (BICseq2-norm) and the joint multi-sample BIC segmentation (BICseq2-seg extension)) — https://www.math.pku.edu.cn/teachers/xirb/downloads/software/BICseq2/BICseq2.html

## Examples

```
hiscanner run --step ado && hiscanner run --step normalize --use-cluster && hiscanner run --step segment && hiscanner run --step cnv
```

## Evaluation signals

- On SDX simulations, HiScanner should reproduce Fig. 2b-i trends: sensitivity ~0.90 and precision ~0.96 for >=1 Mb single-copy losses, and reduced sensitivity ~0.65 for 500 kb single-copy gains, outperforming the four comparator callers
- All callers should struggle with CNAs below 500 kb; verify per-size precision = TP/(TP+FP) and sensitivity = TP/(TP+FN) computed after harmonizing calls to a common bed-like format
- In the aneuploid neuron 5823PFC-B comparison, breakpoint density should be far lower than CHISEL's 10,322 breakpoints (0.38 per 100 kb), with HiScanner loss calls showing concordant BAF shifts
- On the meningioma dataset, CNA-based hierarchical clustering (complete linkage, four clusters on the 500 kb matrix) should agree with SECEDO sSNV clustering with ARI near 0.83
- 'hiscanner validate' must pass on config.yaml, and the ADO step should select 100 kb bins for ~30x data vs 500 kb for the ~0.5x meningioma dataset

## Limitations

- Amplification artifacts in single-cell whole-genome amplification confound CNA detection even at high depth; all simulated callers had reduced sensitivity for CNAs below 500 kb
- CHISEL comparison required parameter fine-tuning and showed high-ploidy predictions, overfitting, and strong GC bias at 100 kb bins due to missing GC normalization
- The model assumes independence between single-cell genomes, which may be violated for cells sharing clonal origins, and the FDR estimate is conservative since FDR increases as CNA size decreases
- SCAN2/HiScanner support only diploid autosomes (1-22); sex chromosomes are not supported

## Evidence

- [abstract] HiScanner combines RDR, BAF, and phasing: "we introduce HiScanner (High-resolution Single-Cell Allelic copy Number callER), which combines read depth, B-allele frequency, and haplotype phasing to identify CNAs with high resolution"
- [discussion] ADO HMM determines minimum bin size: "we designed a HMM with two distinct states—“ADO” and “non-ADO”—and used Gaussian distribution probabilities to describe them"
- [discussion] gHET bulk VAF filter: "Only the gHETs that can be phased by SHAPEIT2 and have a variant allele frequency between 0.1 and 0.9 in bulk data were retained"
- [discussion] MAPQ and mappability filter: "Only uniquely mappable reads with a minimum mapping quality of 30 were considered"
- [readme] Demo pipeline commands: "hiscanner run --step ado      # ADO analysis to identify optimal bin size"
- [methods] SCAN2 must run first: "SCAN2 needs to be run separately before using HiScanner."
- [intro] Benchmark scope vs four methods: "HiScanner outperforms four representative single-cell CNA methods—CHISEL (BAF-aware), Ginkgo, SCOPE, and SCYN (all BAF-agnostic methods)"
