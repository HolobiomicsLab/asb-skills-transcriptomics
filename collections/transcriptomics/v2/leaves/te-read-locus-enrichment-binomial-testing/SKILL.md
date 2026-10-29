---
name: te-read-locus-enrichment-binomial-testing
description: 'Use when validating locus-specific TE quantification: after assigning
  TE reads to loci in single-cell data, test whether reads of a candidate marker TE
  family are significantly enriched in bins flanking known marker gene loci (e.g.,
  Zscan4c/Zscan4d in mouse 2CLC).'
when_to_use_negative:
- When the TE quantification is only at subfamily level (not locus-specific), since
  enrichment must be tested against specific gene-flanking bins rather than family-level
  counts
- For long autonomously expressed TEs such as L1HS spanning several thousand bases,
  where MATES' 2001 bp flanking coverage may yield inaccurate multi-mapping rates
  and thus unreliable locus assignments
- For TE loci with no uniquely mappable flanking regions (e.g., HERVK isoforms with
  high sequence similarity), where locus assignment uncertainty invalidates the binomial
  background expectation
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3170
tools:
- name: MATES
  role: Upstream deep-learning model that allocates multi-mapping reads to specific
    TE loci, producing the per-locus counts tested for enrichment
  repo: https://github.com/mcgilldinglab/MATES
- name: TE_quantifier
  role: MATES module computing per-locus total read counts (URi + ceil(αi × MRi))
    used as input to the enrichment test
  repo: https://github.com/mcgilldinglab/MATES
- name: bam_processor
  role: MATES module segregating unique- and multi-mapping reads aligned to TE loci
    from STARsolo-mapped 10x BAMs
  repo: https://github.com/mcgilldinglab/MATES
- name: bedtools
  role: Used in MATES reference building to trim TE reference against gene reference,
    defining the non-overlapping TE bins used for locus enrichment analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:fdb461ccfce9b8f8bd0dcdc99906af8b81264f9ce3bec01cd958102ee3ba7c4e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mcgilldinglab/MATES
  repo_url: https://github.com/mcgilldinglab/MATES
  tools:
  - MATES
  - MATES_model
  - TE_quantifier
  - TE_quantifier_Intronic
  - bam_processor
  - bedtools
  - data_processor
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 18
---

# TE read locus enrichment binomial testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A statistical validation step that uses a binomial test to assess whether reads from signature transposable elements (e.g., MT2_Mm and MERVL-int) are enriched in genomic bins flanking specific marker gene loci (Zscan4c/Zscan4d), used here to confirm that MATES locus-level TE quantification correctly localizes 2CLC marker TE expression.

## When to use

Use when validating locus-specific TE quantification: after assigning TE reads to loci in single-cell data, test whether reads of a candidate marker TE family are significantly enriched in bins flanking known marker gene loci (e.g., Zscan4c/Zscan4d in mouse 2CLC).

## When NOT to use

- When the TE quantification is only at subfamily level (not locus-specific), since enrichment must be tested against specific gene-flanking bins rather than family-level counts
- For long autonomously expressed TEs such as L1HS spanning several thousand bases, where MATES' 2001 bp flanking coverage may yield inaccurate multi-mapping rates and thus unreliable locus assignments
- For TE loci with no uniquely mappable flanking regions (e.g., HERVK isoforms with high sequence similarity), where locus assignment uncertainty invalidates the binomial background expectation

## Inputs

- MATES locus-level TE count matrix (per-cell, per-locus counts computed as URi + ceil(αi × MRi))
- TE locus genomic coordinates (BED/CSV reference, e.g., TE_nooverlap.bed or TE_full.bed)
- Gene reference annotation (GTF) for Zscan4c/Zscan4d locus coordinates
- Bin definitions around TE start positions (2001 bp coverage vectors, ±1000 bp flanking)

## Outputs

- Binomial test p-values for enrichment of MT2_Mm and MERVL-int reads flanking Zscan4c/Zscan4d loci
- Enrichment bar plots showing TE read counts per flanking bin
- Binary accept/reject validation decision for the 2CLC locus-enrichment reproduction

## How to apply

After the MATES pipeline (bam_processor → data_processor → MATES_model → TE_quantifier) assigns multi-mapping reads to TE loci, take the per-locus read counts (URi + ceil(αi × MRi)) for the candidate marker TE families MT2_Mm and MERVL-int and identify the genomic bins flanking the Zscan4c and Zscan4d gene loci. Count observed TE reads within these flanking bins versus the genomic background expectation and apply a binomial test to assess enrichment. Enrichment should be visualized as bar plots per locus, and the result is interpreted jointly with cell-level evidence: the 2CLC cluster (positive for Zscan4c/Zscan4d) must co-express MERVL-int and MT2_Mm. Acceptance follows the source card validation: significant binomial-test enrichment of MT2_Mm/MERVL-int flanking Zscan4c/Zscan4d, together with matching locus-level clustering agreement (ARI median 0.397, NMI 0.496), consistent with prior work recognizing these TEs as defining 2CLC markers.

## Related tools

- **MATES** (Upstream deep-learning model that allocates multi-mapping reads to specific TE loci, producing the per-locus counts tested for enrichment) — https://github.com/mcgilldinglab/MATES
- **TE_quantifier** (MATES module computing per-locus total read counts (URi + ceil(αi × MRi)) used as input to the enrichment test) — https://github.com/mcgilldinglab/MATES
- **bam_processor** (MATES module segregating unique- and multi-mapping reads aligned to TE loci from STARsolo-mapped 10x BAMs) — https://github.com/mcgilldinglab/MATES
- **bedtools** (Used in MATES reference building to trim TE reference against gene reference, defining the non-overlapping TE bins used for locus enrichment analysis)

## Examples

```
from MATES import bam_processor; bam_processor.split_count_10X_data(TE_mode, sample_list_file, bam_path_file, bc_path_file, bc_ind='CB', num_threads=1)
```

## Evaluation signals

- Binomial test on MT2_Mm and MERVL-int reads flanking Zscan4c/Zscan4d loci yields significant enrichment in the 2CLC population
- The 2CLC cluster identified by Louvain/Leiden clustering of the MATES TE count matrix is positive for Zscan4c/Zscan4d and co-expresses MERVL-int and MT2_Mm
- Locus-level clustering agreement with gene-based clustering reaches the reported medians ARI 0.397 and NMI 0.496
- Enrichment bar plots show TE read counts concentrated in bins flanking Zscan4c/Zscan4d rather than uniformly distributed genome-wide
- Locus-level ARI exceeds subfamily-level ARI in the Fig. 2h box-plot comparison across clustering resolutions

## Limitations

- Multi-mapping rate estimates may be inaccurate for TEs longer than 2001 bp due to incomplete flanking coverage, weakening locus-level enrichment interpretation
- MATES struggles to distinguish identical TE copies (e.g., HERVK isoforms) with high sequence similarity and no uniquely mappable flanking regions
- Biological discoveries from MATES, including marker TE enrichment calls, warrant further experimental validation

## Evidence
