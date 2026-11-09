---
name: cross-platform-correlation-validation
description: Use when a newly produced poly(A) site or APA quantification matrix (e.g.,
  polyA_sites.tsv from scAPAtrap on ST-MOB data) must be checked for agreement against
  a second transcriptomic platform.
when_to_use_negative:
- Input is not a quantified poly(A) site or APA usage matrix (e.g., raw FASTQ or a
  genome alignment BAM) — correlation validation presupposes a site-level expression
  table
- The goal is APA site identification from platforms lacking spatial barcodes, e.g.,
  MERFISH or seqFISH — the article notes APA sites can only be identified from spatially
  barcoded ST data with existing scRNA-seq tools such as scAPAtrap or Sierra
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0080
tools:
- name: stAPAminer
  role: Provides computePearsonIndex for verifying imputation correctness by correlation,
    and validation of the downstream APA spatial analysis whose inputs this skill
    checks
  repo: https://github.com/BMILAB/stAPAminer
- name: scAPAtrap
  role: Generates the poly(A) site expression table (polyA_sites.tsv) that is the
    object being cross-platform validated
- name: umi_tools
  role: Extracts barcodes/UMIs from read 1 and removes PCR duplicates (dedup) upstream
    of the quantification being validated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:49851f92ff2f36332dbe55088089d20e92d40e94b1d2ad1eb19f207f569c4ce6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - STAR
  - scAPAtrap
  - stAPAminer
  - umi_tools
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Cross-platform correlation validation

## Summary

Validates poly(A) site-level expression quantification from spatially barcoded ST data by computing Pearson correlations of the site expression table against independently generated datasets from other platforms — here, single-cell MOB (SC-MOB) data and neural bulk 3'-seq data.

## When to use

Use when a newly produced poly(A) site or APA quantification matrix (e.g., polyA_sites.tsv from scAPAtrap on ST-MOB data) must be checked for agreement against a second transcriptomic platform.

## When NOT to use

- Input is not a quantified poly(A) site or APA usage matrix (e.g., raw FASTQ or a genome alignment BAM) — correlation validation presupposes a site-level expression table
- The goal is APA site identification from platforms lacking spatial barcodes, e.g., MERFISH or seqFISH — the article notes APA sites can only be identified from spatially barcoded ST data with existing scRNA-seq tools such as scAPAtrap or Sierra

## Inputs

- polyA_sites.tsv — poly(A) site expression table (rows: 47,886 poly(A) sites; columns: 260 ST spots)
- SC-MOB single-cell MOB expression/reference data
- Neural bulk 3'-seq expression data

## Outputs

- Pearson correlation coefficient for ST-MOB vs SC-MOB (expected ~0.7)
- Pearson correlation coefficient for ST-MOB vs neural bulk 3'-seq (expected ~0.69)
- Pass/fail assessment of site-level quantification fidelity prior to APA spatial analysis

## How to apply

After building the poly(A) site expression table (polyA_sites.tsv) from the ST-MOB data (SRA: SRR3382373, processed with umi_tools dedup and scAPAtrap, 47,886 sites across 260 spots retained at ten or more read counts), compute Pearson correlations of site-level expression between the ST-derived matrix and external reference datasets: single-cell MOB (SC-MOB) data, with an expected correlation of 0.7, and neural bulk 3'-seq data, with an expected correlation of 0.69. Correlation values near these expected magnitudes indicate the spatial site quantification recapitulates independent platforms and is fit for downstream stAPAminer APA-usage analysis (computeAPAIndex, imputeAPAIndex). Within stAPAminer itself, an analogous verification step uses computePearsonIndex on the RUD (APA ratio) matrix to check imputation correctness, with representative genes selected using Pearson's correlation > 0.5 and P < 0.05 against group mean profiles.

## Related tools

- **stAPAminer** (Provides computePearsonIndex for verifying imputation correctness by correlation, and validation of the downstream APA spatial analysis whose inputs this skill checks) — https://github.com/BMILAB/stAPAminer
- **scAPAtrap** (Generates the poly(A) site expression table (polyA_sites.tsv) that is the object being cross-platform validated)
- **umi_tools** (Extracts barcodes/UMIs from read 1 and removes PCR duplicates (dedup) upstream of the quantification being validated)

## Examples

```
Rscript -e 'stObj <- createStAPAminerObject(RUD,position); pearson_RUD <- computePearsonIndex(stObj,"RUD")'
```

## Evaluation signals

- Pearson correlation of site-level expression between ST-MOB and SC-MOB should approximate 0.7
- Pearson correlation of site-level expression between ST-MOB and neural bulk 3'-seq data should approximate 0.69
- polyA_sites.tsv should contain 47,886 identified poly(A) sites from the ST-MOB data before correlation testing
- Spot count entering quantification should be 260 (spots with ten or more read counts)
- Within stAPAminer verification, representative spatial APA pattern genes require Pearson's correlation > 0.5 and P < 0.05 to the group mean profile

## Limitations

- Correlation thresholds (0.7 vs SC-MOB; 0.69 vs neural bulk 3'-seq) are dataset-specific to MOB ST data and may not transfer to other tissues or platforms
- The article flags that benchmarking of methods for obtaining gene expression matrices and RUD matrices is still needed, which affects what the correlations certify
- Discrepancies among MOB ST replicates exist and no method is proposed to mitigate them for reproducible spatial patterns, limiting cross-dataset agreement expectations

## Evidence

- [readme] Validate by correlation via computePearsonIndex on the imputed RUD matrix: "##Verify by correlation
pearson_RUD <- computePearsonIndex(stObj,"RUD")"
- [discussion] Representative spatial APA genes selected with Pearson correlation > 0.5 and P < 0.05: [evidence span withheld — anchor status: not_found]
- [discussion] APA site identification is restricted to spatially barcoded ST data: "APA sites can only be identified from the spatially barcoded ST data using existing scRNA-seq tools such as scAPAtrap or Sierra"
- [abstract] Spatial APA patterns are reproducible across replicates, the property cross-platform correlation probes: "By extending this analysis to two additional replicates of the MOB ST data, we observed that the spatial APA patterns of several genes were reproducible among replicates"
