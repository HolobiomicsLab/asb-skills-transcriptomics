---
name: read-depth-segmentation
description: Use when segmenting high-coverage (>20x) PTA-amplified single-cell WGS
  read-depth profiles into CNAs at 200 kb–5 Mb scales, especially when allele-specific
  calls and comparison against BAF-agnostic RDR-only callers are needed.
when_to_use_negative:
- Low-coverage (<0.05x) single-cell data from DLP+, 10x Genomics CNV, or DOP-PCR platforms
  — CHISEL is designed for that regime, and ADO-based bin-size selection for PTA data
  will not transfer
- No matched bulk WGS is available to call and phase gHETs, so BAF integration and
  the RDR+BAF joint segmentation cannot be performed
- Analysis targets sex chromosomes — SCAN2 (the upstream SNV step for HiScanner) supports
  only diploid autosomes 1-22
edam_operation: http://edamontology.org/operation_3230
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: HiScanner
  role: 'Primary caller: nucleotide-resolution RDR normalization, ADO-based bin sizing,
    joint BIC segmentation, and RDR+BAF copy-number calling'
  repo: https://github.com/parklab/HiScanner
- name: BICseq2
  role: GC correction and binning (BICseq2-norm) and the single-sample segmentation
    algorithm extended to multi-sample joint BIC segmentation
  repo: https://www.math.pku.edu.cn/teachers/xirb/downloads/software/BICseq2/BICseq2.html
- name: SCAN2
  role: Upstream somatic SNV/indel genotyper on PTA-amplified single cells; results
    are checked and phased before HiScanner runs
  repo: https://github.com/parklab/SCAN2
- name: Ginkgo
  role: RDR-only baseline caller run on the same single-cell BAM via its command line
    version
  repo: https://github.com/robertaboukhalil/ginkgo
- name: CHISEL
  role: BAF-aware baseline run at 100 kb and optimized 5 Mb bins; its likelihood-based
    ploidy/CN method is also extended by HiScanner
  repo: https://github.com/raphael-group/chisel
- name: SCOPE
  role: RDR baseline run at 100 kb resolution with MAPQ threshold of 40
- name: SCYN
  role: RDR baseline single-cell CNA caller used in the method comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:cf017c8f83f1848b9d71a9982de11cbb57fce5e595e1d14f919a80b2ff9ca2d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/parklab/HiScanner
  tools:
  - BICseq2
  - CHISEL
  - Ginkgo
  - HiScanner
  - SCAN2
  - SCOPE
  - SCYN
  edam_operation: http://edamontology.org/operation_3230
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# read-depth-segmentation

## Summary

Detect single-cell copy number alterations by normalizing read depth (with GC correction via BICseq2-norm), jointly segmenting single-cell RDR profiles across samples with a BIC criterion extended from BICseq2-seg, and optionally integrating phased BAF to infer allele-specific integer copy numbers, as implemented in HiScanner and benchmarked against Ginkgo, CHISEL, SCYN, and SCOPE.

## When to use

Use when segmenting high-coverage (>20x) PTA-amplified single-cell WGS read-depth profiles into CNAs at 200 kb–5 Mb scales, especially when allele-specific calls and comparison against BAF-agnostic RDR-only callers are needed.

## When NOT to use

- Low-coverage (<0.05x) single-cell data from DLP+, 10x Genomics CNV, or DOP-PCR platforms — CHISEL is designed for that regime, and ADO-based bin-size selection for PTA data will not transfer
- No matched bulk WGS is available to call and phase gHETs, so BAF integration and the RDR+BAF joint segmentation cannot be performed
- Analysis targets sex chromosomes — SCAN2 (the upstream SNV step for HiScanner) supports only diploid autosomes 1-22

## Inputs

- Single-cell BAM (e.g., PTA-amplified scWGS from neuron 5823PFC-B at 29x)
- Paired bulk whole-genome sequencing BAM
- Split reference genome (hg19/GRCh37+decoy)
- 100mer mappability track (hg19CRG.100bp from BICseq2)
- SCAN2 somatic SNV/indel genotyper output
- Phased gHET list (GATK HaplotypeCaller + SHAPEIT2 with 1000 Genomes Phase 3 panel)

## Outputs

- Joint segmented RDR profiles across cells (BIC-based breakpoints)
- Per-cell ploidy and integer allele-specific copy-number states
- Copy-number tracks per method for comparison figures
- Breakpoint density per method (e.g., per 100 kb)
- Length distribution of HiScanner loss calls concordant with BAF evidence

## How to apply

Compute bin-level read depth using only uniquely mappable reads with mapping quality ≥ 30, then run GC correction and binning with BICseq2-norm (default parameters) on both single-cell and paired bulk data. Select the bin size from assay-specific allelic dropout patterns by training a two-state HMM ('ADO'/'non-ADO') with Baum-Welch, which determines the dropout size distribution and minimum bin size. Perform joint multi-sample segmentation by extending BICseq2-seg's Bayesian-information-criterion framework and greedily merging adjacent bins across cells. For allele-specific calls, retain only gHETs phasable by SHAPEIT2 with bulk VAF between 0.1 and 0.9, and infer ploidy plus integer allele-specific copy number states by extending CHISEL's likelihood-based method with a GMM on 'balanced' bins. For benchmarking, run the same single-cell BAMs through Ginkgo (RDR-only command line), CHISEL (at 100 kb and 5 Mb bins), SCYN, and SCOPE (100 kb resolution, MAPQ 40), and compare breakpoint counts and densities per method.

## Related tools

- **HiScanner** (Primary caller: nucleotide-resolution RDR normalization, ADO-based bin sizing, joint BIC segmentation, and RDR+BAF copy-number calling) — https://github.com/parklab/HiScanner
- **BICseq2** (GC correction and binning (BICseq2-norm) and the single-sample segmentation algorithm extended to multi-sample joint BIC segmentation) — https://www.math.pku.edu.cn/teachers/xirb/downloads/software/BICseq2/BICseq2.html
- **SCAN2** (Upstream somatic SNV/indel genotyper on PTA-amplified single cells; results are checked and phased before HiScanner runs) — https://github.com/parklab/SCAN2
- **Ginkgo** (RDR-only baseline caller run on the same single-cell BAM via its command line version) — https://github.com/robertaboukhalil/ginkgo
- **CHISEL** (BAF-aware baseline run at 100 kb and optimized 5 Mb bins; its likelihood-based ploidy/CN method is also extended by HiScanner) — https://github.com/raphael-group/chisel
- **SCOPE** (RDR baseline run at 100 kb resolution with MAPQ threshold of 40)
- **SCYN** (RDR baseline single-cell CNA caller used in the method comparison)

## Examples

```
hiscanner run --step all
```

## Evaluation signals

- On the aneuploid neuron 5823PFC-B, CHISEL's 100 kb mode yields ~10,322 breakpoints (0.38 per 100 kb) while HiScanner and the other callers remain conservative — a correct run reproduces this fragmentation contrast
- All read counts derive only from uniquely mappable reads with MAPQ ≥ 30; verify the filter was applied before binning
- Only gHETs with bulk VAF between 0.1 and 0.9 that SHAPEIT2 phased enter BAF computation — check the retained SNP set
- HiScanner should detect small CNAs with fewer spurious breakpoints than Ginkgo, CHISEL, SCYN, and SCOPE, with concordant loss calls showing expected BAF shifts
- The ADO HMM must report a minimum bin size consistent with the PTA dropout size distribution before segmentation proceeds

## Limitations

- CHISEL at 100 kb bins showed high-ploidy predictions, overfitting, and strong GC bias in its preprocessed RDR due to lack of GC normalization in its pipeline; the comparison required parameter fine-tuning (optimized 5 Mb configuration)
- The joint model assumes independence between single-cell genomes, which can be violated for cells sharing clonal origins
- Single-cell CNA calls overlapping bulk CNA calls were removed, so germline/clonal bulk events will not appear in the single-cell call set

## Evidence

- [intro] Nucleotide-resolution read depth normalization correcting GC bias: "HiScanner performs read depth normalization at nucleotide resolution, followed by joint segmentation of single-cell profiles using the Bayesian information criterion (BIC)"
- [discussion] BICseq2-norm GC correction and binning: "BICseq2-norm25 with default parameters was used to perform GC correction and binning on single-cell and bulk sequencing data from each individual"
- [discussion] Multi-sample extension of BICseq2-seg: "We extended the single-sample segmentation algorithm implemented by BICseq2-seg25 to the multi-sample case"
- [intro] ADO-based bin size selection: "It also considers assay-specific ADO patterns to find an optimal bin size, a nuanced yet crucial parameter in CNA analysis"
- [discussion] MAPQ and mappability filter: "Only uniquely mappable reads with a minimum mapping quality of 30 were considered"
- [full_text] CHISEL fragmentation on aneuploid neuron: [evidence span withheld — anchor status: not_found]
- [discussion] gHET retention filter: "Only the gHETs that can be phased by SHAPEIT2 and have a variant allele frequency between 0.1 and 0.9 in bulk data were retained"
