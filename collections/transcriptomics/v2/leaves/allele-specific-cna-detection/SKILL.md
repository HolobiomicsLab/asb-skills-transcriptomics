---
name: allele-specific-cna-detection
description: Use when analyzing PTA-amplified high-coverage (>20x) scWGS BAMs with
  matched bulk WGS, to call small (200 kb–5 Mb) allele-specific CNAs that RDR-only
  callers miss and to obtain integer copy-number states per allele per cell.
when_to_use_negative:
- Data are low-coverage (<0.05x per cell) scWGS from DLP+/10x-style platforms — CHISEL/Alleloscope
  are designed for that regime and BAF at HiScanner's resolution is unreliable
- The matched bulk is absent — reference-based phasing and BAF estimation (and SCAN2
  germline exclusion) both require bulk WGS
- The cell is known aneuploid across whole chromosomes and the goal is megabase-scale
  mosaic CNA prevalence analysis — the article excluded aneuploid neurons from CNA
  length and prevalence analyses
edam_operation: http://edamontology.org/operation_3231
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3308
tools:
- name: HiScanner
  role: 'Primary caller: joint RDR+BAF allele-specific CNA detection with ADO-based
    bin-size selection and BIC joint segmentation'
  repo: https://github.com/parklab/HiScanner
- name: SCAN2
  role: Somatic SNV/indel genotyper on PTA-amplified single cells; must be run before
    HiScanner and its output checked/phased by HiScanner
  repo: https://github.com/parklab/SCAN2
- name: BICseq2-norm / BICseq2-seg
  role: GC correction, binning (single-cell and bulk) and the BIC segmentation framework
    extended to multi-sample joint segmentation
- name: SHAPEIT2
  role: Phases gHETs using the 1000 Genomes Phase 3 integrated haplotype panel before
    BAF computation
- name: GATK HaplotypeCaller
  role: Calls SNPs on bulk sequencing data per individual with default 'Best Practices'
    parameters
- name: Ginkgo
  role: BAF-agnostic RDR-only comparator run on the same single-cell BAM to benchmark
    allele-specific calls
  repo: https://github.com/robertaboukhalil/ginkgo
- name: CHISEL
  role: BAF-aware comparator whose likelihood-based copy-state inference HiScanner
    extends; run at 100 kb and 5 Mb bin settings in benchmarks
  repo: https://github.com/raphael-group/chisel
- name: SCOPE
  role: BAF-agnostic comparator run at 100 kb resolution with MAPQ threshold 40
- name: SCYN
  role: BAF-agnostic RDR-only comparator on the same single-cell inputs
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:f027e3e6030dd5f589dc7d455639ed4609c554d29772be0730a4f4283ac2b6bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/parklab/HiScanner
  tools:
  - BICseq2-norm / BICseq2-seg
  - CHISEL
  - GATK HaplotypeCaller
  - Ginkgo
  - HiScanner
  - SCAN2
  - SCOPE
  - SCYN
  - SHAPEIT2
  edam_operation: http://edamontology.org/operation_3231
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# allele-specific-cna-detection

## Summary

Detect allele-specific (haplotype-specific) copy number alterations in single-cell WGS by jointly modeling read-depth ratios (RDR) and B-allele frequencies (BAF) derived from reference-phased germline heterozygous SNPs, as implemented in HiScanner's standard pipeline.

## When to use

Use when analyzing PTA-amplified high-coverage (>20x) scWGS BAMs with matched bulk WGS, to call small (200 kb–5 Mb) allele-specific CNAs that RDR-only callers miss and to obtain integer copy-number states per allele per cell.

## When NOT to use

- Data are low-coverage (<0.05x per cell) scWGS from DLP+/10x-style platforms — CHISEL/Alleloscope are designed for that regime and BAF at HiScanner's resolution is unreliable
- The matched bulk is absent — reference-based phasing and BAF estimation (and SCAN2 germline exclusion) both require bulk WGS
- The cell is known aneuploid across whole chromosomes and the goal is megabase-scale mosaic CNA prevalence analysis — the article excluded aneuploid neurons from CNA length and prevalence analyses

## Inputs

- Single-cell scWGS BAM file (PTA-amplified, high coverage ~29-30x)
- Matched bulk whole-genome sequencing BAM (for gHET calling and phasing)
- SCAN2 somatic SNP/indel genotyping output for the single cell (run before HiScanner)
- Split reference genome FASTA (e.g. hg19/GRCh37+decoy)
- Mappability track (hg19CRG.100bp from BICseq2, or hg38/GRCh38 150mer from figshare DOI 10.6084/m9.figshare.28370357.v1)
- config.yaml project configuration and metadata.txt

## Outputs

- Per-cell integer allele-specific copy-number states (per bin/segment, haplotype a and b)
- Cell ploidy estimates
- Joint segmentation breakpoints per chromosome (BIC-based)
- CNV calls with concordant BAF evidence (BAF shifts for loss calls)
- Copy-number tracks suitable for conversion to bed-like start/end/state format for caller comparison

## How to apply

Run the HiScanner standard pipeline (RDR + BAF) via 'hiscanner run --step all' after 'hiscanner init' and 'hiscanner validate': first check SCAN2 somatic SNP output on the bulk data ('--step snp', '--step phase'), then use the ADO step to train a two-state ('ADO'/'non-ADO') Baum-Welch HMM that selects an assay-specific minimum bin size. Read depth is GC-corrected and binned with BICseq2-norm (default parameters, e.g. the hg19CRG.100bp mappability track), retaining only uniquely mappable reads with MAPQ >= 30 and only phased gHETs with bulk VAF between 0.1 and 0.9 (phased with SHAPEIT2 against the 1000 Genomes Phase 3 panel). Segmentation is joint across single-cell profiles by extending BICseq2-seg's BIC framework, and ploidy plus integer allele-specific copy states are inferred by extending CHISEL's likelihood-based method with a GMM on 'balanced' bins, modeling RDR and BAF simultaneously. Restrict to autosomes, remove single-cell calls overlapping bulk CNA calls, and for benchmarks harmonize all callers' output into a common start/end/allele-specific-state format before computing precision and sensitivity per CNA type and size.

## Related tools

- **HiScanner** (Primary caller: joint RDR+BAF allele-specific CNA detection with ADO-based bin-size selection and BIC joint segmentation) — https://github.com/parklab/HiScanner
- **SCAN2** (Somatic SNV/indel genotyper on PTA-amplified single cells; must be run before HiScanner and its output checked/phased by HiScanner) — https://github.com/parklab/SCAN2
- **BICseq2-norm / BICseq2-seg** (GC correction, binning (single-cell and bulk) and the BIC segmentation framework extended to multi-sample joint segmentation)
- **SHAPEIT2** (Phases gHETs using the 1000 Genomes Phase 3 integrated haplotype panel before BAF computation)
- **GATK HaplotypeCaller** (Calls SNPs on bulk sequencing data per individual with default 'Best Practices' parameters)
- **Ginkgo** (BAF-agnostic RDR-only comparator run on the same single-cell BAM to benchmark allele-specific calls) — https://github.com/robertaboukhalil/ginkgo
- **CHISEL** (BAF-aware comparator whose likelihood-based copy-state inference HiScanner extends; run at 100 kb and 5 Mb bin settings in benchmarks) — https://github.com/raphael-group/chisel
- **SCOPE** (BAF-agnostic comparator run at 100 kb resolution with MAPQ threshold 40)
- **SCYN** (BAF-agnostic RDR-only comparator on the same single-cell inputs)

## Examples

```
hiscanner init && hiscanner validate && hiscanner run --step all
```

## Evaluation signals

- Loss calls concordant with BAF evidence must show the expected BAF shifts away from 0.5 (allele imbalance), confirming haplotype-specific loss rather than RDR-only artifacts
- In the benchmark (task_002), expect high sensitivity (~0.90) / precision (~0.96) for >=1 Mb single-copy losses and reduced sensitivity (~0.65) for 500 kb single-copy gains; all callers should struggle below 500 kb
- Segmentation should be conservative relative to CHISEL at 100 kb: on aneuploid neuron 5823PFC-B, CHISEL produced 10,322 breakpoints (0.38 per 100 kb) while HiScanner maintained fewer spurious breakpoints
- Only gHETs with bulk VAF 0.1–0.9 that were phased by SHAPEIT2 and uniquely mappable MAPQ >= 30 reads contribute to BAF estimates; verify filters were applied
- CN state assignment is integer and allele-specific; 'balanced' bins (BAF ~0.5) should be identified by the GMM before inferring whether the largest balanced cluster is {1,1}, {2,2}, or higher ploidy

## Limitations

- The model assumes independence between single-cell genomes, which may be violated for cells sharing clonal origins
- Clonal classification may conflate true clonal expansions with independent recurrent mutations sharing identical coordinates
- CHISEL (comparator) showed a tendency for high-ploidy predictions and overfitting, plus strong GC bias in CHISEL-preprocessed RDR at 100 kb bins due to lack of GC normalization; careful parameter fine-tuning was needed
- FDR estimates are conservative because FDR increases as CNA size decreases
- SCAN2 must only be applied to diploid autosomes 1-22; sex chromosomes are not supported

## Evidence

- [intro] HiScanner leverages reference-based phasing and local sequence bias to efficiently estimate BAF to complement read depth-based CNA identification: "HiScanner leverages reference-based phasing and local sequence bias to efficiently estimate BAF to complement read depth-based CNA identification"
- [intro] ADO-based bin size selection: "It also considers assay-specific ADO patterns to find an optimal bin size, a nuanced yet crucial parameter in CNA analysis"
- [discussion] gHET bulk VAF filter: "Only the gHETs that can be phased by SHAPEIT2 and have a variant allele frequency between 0.1 and 0.9 in bulk data were retained"
- [intro] RDR+BAF joint modeling: "By simultaneously modeling RDR and BAF for individual cells, HiScanner infers cell ploidy and integer allele-specific copy number states"
- [readme] HiScanner two pipeline modes: "Standard Pipeline (RDR + BAF): Full analysis using both read depth ratios (RDR) and B-allele frequencies (BAF)"
