---
name: asas-significant-event-filtering
description: Use when a PAIRADISE results table has been produced (e.g., for GM12878
  or Geuvadis populations) and a statistically significant ASAS SNP/event list is
  needed for downstream GWAS-LD overlap or enrichment testing.
when_to_use_negative:
- The task is sQTL association testing between genotypes and splicing across unrelated
  individuals rather than within-individual allele-specific analysis (use GLiMMPS-style
  sQTL testing instead).
- The heterozygous tagging SNP lies within the alternative exon or too far from it
  to co-occur on short RNA-seq reads, since PAIRADISE cannot analyze such SNPs.
- Reads do not align uniquely to both haplotype-modified personal genomes at the same
  location, as these must be discarded upstream before significance testing.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: PAIRADISE
  role: Runs the paired statistical model (Step 5) and outputs the significant ASAS
    event table with FDR <= 10% default significance
  repo: https://github.com/Xinglab/PAIRADISE
- name: STAR
  role: Aligns RNA-seq reads to the haplotype-modified personal genomes in Step 1,
    providing the counts that the significance filter acts upon
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:89e3c42aa9ea10f68959e375524d461e4c5a1887c8283af03a01d73437c16084
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Xinglab/PAIRADISE
  repo_url: https://github.com/Xinglab/PAIRADISE
  tools:
  - PAIRADISE
  - STAR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# asas-significant-event-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters the PAIRADISE output table to retain significant allele-specific alternative splicing (ASAS) events at the default FDR <= 10% threshold, yielding per-dataset lists of significant ASAS SNPs used for downstream LD/enrichment analyses.

## When to use

Use when a PAIRADISE results table has been produced (e.g., for GM12878 or Geuvadis populations) and a statistically significant ASAS SNP/event list is needed for downstream GWAS-LD overlap or enrichment testing.

## When NOT to use

- The task is sQTL association testing between genotypes and splicing across unrelated individuals rather than within-individual allele-specific analysis (use GLiMMPS-style sQTL testing instead).
- The heterozygous tagging SNP lies within the alternative exon or too far from it to co-occur on short RNA-seq reads, since PAIRADISE cannot analyze such SNPs.
- Reads do not align uniquely to both haplotype-modified personal genomes at the same location, as these must be discarded upstream before significance testing.

## Inputs

- Merged per-sample exon inclusion/skipping count table from PAIRADISE Step 4
- PAIRADISE Step 5 statistical model output table (p-value and FDR columns)
- sample_name.txt with sample IDs and RNA-seq read lengths (defines the dataset/population being filtered)

## Outputs

- Per-dataset list of significant ASAS events/SNPs at FDR <= 10% (ExonID + SNP identifiers)
- Tallied event counts per dataset (e.g., 133 significant events in GM12878)

## How to apply

Run Step 5 of the PAIRADISE pipeline (statistical modeling) on the merged count table produced by Steps 1-4 (personalization/mapping/assignment, joint annotation, counting, merging). PAIRADISE then outputs a table of significant ASAS events with the default significance set to FDR <= 10%; each row contains ExonID (gene name, chromosome, strand, SNP name, location, alleles), IJC/SJC counts for reference and alternative alleles, isoform effective lengths, and the raw p-value. Filter to rows passing the FDR cutoff to extract the significant ASAS SNP list per dataset (e.g., 133 events in GM12878 and 153/116/101/104/199 in Geuvadis CEU/FIN/GBR/TSI/YRI if the sigma_i1 = sigma_i2 option is used, vs. 116 and 91/82/75/71/143 under the conservative default). Note that the results are intentionally conservative because sigma_i1 is not restricted to equal sigma_i2 by default; enabling that user option increases event counts and should be applied consistently across all datasets.

## Related tools

- **PAIRADISE** (Runs the paired statistical model (Step 5) and outputs the significant ASAS event table with FDR <= 10% default significance) — https://github.com/Xinglab/PAIRADISE
- **STAR** (Aligns RNA-seq reads to the haplotype-modified personal genomes in Step 1, providing the counts that the significance filter acts upon) — https://github.com/alexdobin/STAR

## Examples

```
qsub -cwd -l h_vmem=10G qsub_files/pairadise_step5_stat.qsub
```

## Evaluation signals

- Every retained row satisfies FDR <= 10% (the PAIRADISE default significance).
- Output schema matches the PAIRADISE table: ExonID, IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen, pval.
- Filtered event counts are consistent with published numbers under the chosen sigma setting (e.g., 116 conservative vs. 133 with sigma_i1 = sigma_i2 in GM12878; 91 vs. 153 in Geuvadis CEU).
- Only SNPs with a heterozygous tagging SNP outside but near the alternative exon appear in the filtered list, matching PAIRADISE's detectability constraint.
- The filtered SNP list is non-empty and reproducible when the pipeline is re-run on the same Geuvadis population directory (e.g., CEU).

## Limitations

- Results are intentionally conservative: by default PAIRADISE does not restrict sigma_i1 to equal sigma_i2; allowing it increases significant events (116 to 133 in GM12878; 91 to 153 in Geuvadis CEU).
- Causal SNPs located within the alternative exon or nearby intronic regions are not tested, so only tagging SNPs are identified among significant events.
- Detection requires a heterozygous SNP outside the alternative exon but close enough to be on the same RNA-seq read; SNPs within or too distant from alternative exons cannot be analyzed with short reads.
- ASAS signals may vary across individuals due to cis SNPs or trans-acting splicing regulators, which PAIRADISE's current framework does not model with individual-specific covariates.

## Evidence
