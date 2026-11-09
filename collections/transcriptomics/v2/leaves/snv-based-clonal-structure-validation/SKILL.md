---
name: snv-based-clonal-structure-validation
description: 'Use when single-cell CNA clone assignments need orthogonal validation:
  paired single-cell + bulk WGS exists, somatic SNVs have been called (e.g. SCAN2/Mutect2),
  and an ARI between CNA- and SNV-based clusterings is the acceptance metric.'
when_to_use_negative:
- No matched bulk sample is available — SCAN2 excludes any mutation candidate with
  any supporting reads in matched bulk, so somatic SNV calls cannot be produced
- Analysis targets sex chromosomes — SCAN2 should only be applied to diploid autosomes
  1-22
- Cells lack sufficient SNV burden (e.g. non-cancerous neurons/oligodendrocytes with
  only small mosaic CNAs) so mutation-genotype clustering is uninformative
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: SCAN2
  role: Calls somatic SNVs/indels from paired bulk and single-cell PTA-amplified BAMs;
    run until shapeit/phased_hets.vcf.gz
  repo: https://github.com/parklab/SCAN2
- name: HiScanner
  role: Processes SCAN2 output (--step snp, --step phase) to produce phased SNV/genotype
    data used in downstream clone comparison
  repo: https://github.com/parklab/HiScanner
- name: SECEDO
  role: Clusters the 179 cells by somatic point-mutation genotype from Mutect2 PASS
    SNVs
- name: Mutect2
  role: Somatic SNV caller whose 'PASS' variants are input to SECEDO clustering
- name: SHAPEIT
  role: Phases gHETs with the 1000 Genomes Phase 3 integrated haplotype panel before
    HiScanner --step phase
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:de8f05be2ca73a56337e7901b34594e8caaad82ff1c77de3b2e2e400eac41712
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/parklab/SCAN2
  tools:
  - BICseq2
  - HiScanner
  - Mutect2
  - SCAN2
  - SECEDO
  - SHAPEIT
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# SNV-based clonal structure validation

## Summary

Independently cluster single tumor cells by somatic point-mutation genotype (SECEDO on Mutect2 PASS SNVs from SCAN2 calls) and compare against CNA-derived clone assignments using the Adjusted Rand Index, as done for 179 serial meningioma cells (ARI 0.83).

## When to use

Use when single-cell CNA clone assignments need orthogonal validation: paired single-cell + bulk WGS exists, somatic SNVs have been called (e.g. SCAN2/Mutect2), and an ARI between CNA- and SNV-based clusterings is the acceptance metric.

## When NOT to use

- No matched bulk sample is available — SCAN2 excludes any mutation candidate with any supporting reads in matched bulk, so somatic SNV calls cannot be produced
- Analysis targets sex chromosomes — SCAN2 should only be applied to diploid autosomes 1-22
- Cells lack sufficient SNV burden (e.g. non-cancerous neurons/oligodendrocytes with only small mosaic CNAs) so mutation-genotype clustering is uninformative

## Inputs

- Paired bulk and single-cell BAM files (low-coverage ~0.5x scWGS, serial meningioma tumor pair)
- SCAN2 raw somatic variant VCF (gatk/hc_raw.mmq60.vcf.gz) and phased gHETs from SHAPEIT using the 1000 Genomes Phase 3 panel
- Mutect2 somatic SNV calls filtered to 'PASS' flag
- Per-cell CNA cluster assignments from the 500 kb copy-number matrix (complete-linkage hierarchical clustering, four clusters)

## Outputs

- SECEDO SNV-based cluster assignments for each of the 179 cells
- Adjusted Rand Index between CNA-based and SNV-based clusterings (expected ~0.83)
- Output table of cluster assignments and ARI

## How to apply

Run SCAN2 on paired bulk and single-cell BAMs stopping at the phasing step (--until shapeit/phased_hets.vcf.gz), since only gatk/hc_raw.mmq60.vcf.gz and phased gHETs (genotype field renamed 'phasedgt') feed downstream steps. Process with 'hiscanner run --step snp' and '--step phase', retaining gHETs phasable by SHAPEIT with bulk VAF between 0.1 and 0.9 and reads with mapping quality >= 30 on the hg19 100mer mappability track. Independently run SECEDO on Mutect2-called somatic SNVs, keeping only 'PASS' variants, to cluster the same cells (e.g. 179 meningioma cells from initial n=88 and recurrent n=91 biopsies) by point-mutation genotype. Compute the Adjusted Rand Index between the SECEDO SNV clusters and the CNA-based hierarchical clusters (complete linkage on the 500 kb copy-number matrix, cut into four clusters corresponding to Clusters 8, 11, 13, and 15); agreement near the reported 0.83 validates the CNA caller's subclone assignments, including assignment of the grade II cell BOS-EE06 to Cluster 15.

## Related tools

- **SCAN2** (Calls somatic SNVs/indels from paired bulk and single-cell PTA-amplified BAMs; run until shapeit/phased_hets.vcf.gz) — https://github.com/parklab/SCAN2
- **HiScanner** (Processes SCAN2 output (--step snp, --step phase) to produce phased SNV/genotype data used in downstream clone comparison) — https://github.com/parklab/HiScanner
- **SECEDO** (Clusters the 179 cells by somatic point-mutation genotype from Mutect2 PASS SNVs)
- **Mutect2** (Somatic SNV caller whose 'PASS' variants are input to SECEDO clustering)
- **SHAPEIT** (Phases gHETs with the 1000 Genomes Phase 3 integrated haplotype panel before HiScanner --step phase)

## Examples

```
hiscanner run --step snp && hiscanner run --step phase   # process SCAN2 raw VCF (gatk/hc_raw.mmq60.vcf.gz) and phased gHETs for downstream SECEDO/ARI comparison
```

## Evaluation signals

- Adjusted Rand Index between SECEDO SNV clusters and HiScanner CNA hierarchical clusters should approach the reported 0.83
- Four CNA clusters should correspond to the recovered clone structure (Clusters 8, 11, 13, and 15)
- Grade II cell BOS-EE06 should be assigned to Cluster 15
- Retained gHETs must be SHAPEIT-phased with bulk VAF between 0.1 and 0.9; reads must have mapping quality >= 30
- SNV clustering input should contain only Mutect2 variants with the 'PASS' flag

## Limitations

- Clonal classification may conflate true clonal expansions with independent recurrent mutations sharing identical coordinates
- SCAN2 will not call mutation candidates with any read support in the matched bulk sample, excluding lower-frequency clonal somatic mutations at standard depths
- Current HiScanner model assumes independence between single-cell genomes, which may be violated for cells sharing clonal origins

## Evidence

- [results] Independent SECEDO clustering of SNVs validates HiScanner CNA clone structure: [evidence span withheld — anchor status: not_found]
- [discussion] Only Mutect2 PASS variants are retained: "Only variants with “PASS” flag were kept"
- [discussion] gHET VAF filter for phasing: "Only the gHETs that can be phased by SHAPEIT2 and have a variant allele frequency between 0.1 and 0.9 in bulk data were retained"
- [methods] SCAN2 is a prerequisite for HiScanner: "[SCAN2](https://github.com/parklab/SCAN2) needs to be run separately before using HiScanner."
- [intro] Orthogonal mutation-based clustering corroborates CNAs: "HiScanner revealed phases and timing of tumor progression by detecting subclone-defining CNAs, which were subsequently corroborated by an orthogonal clustering based on somatic point mutations"
