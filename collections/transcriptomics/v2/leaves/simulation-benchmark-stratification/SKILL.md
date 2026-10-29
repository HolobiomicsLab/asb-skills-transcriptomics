---
name: simulation-benchmark-stratification
description: Use when evaluating assembly/quantification accuracy on simulated data
  where performance may depend on structural complexity (e.g., overlapping genomic
  regions among circRNAs), requiring metrics stratified by a complexity bin such as
  RI 1-4 vs RI > 4.
when_to_use_negative:
- Data are real experimental reads without simulated ground-truth transcripts — sensitivity/precision
  against truth cannot be computed
- Each BSJ site is isolated (no overlapping genomic regions among circRNAs), so all
  sites fall in a single RI bin and stratification is uninformative
- Goal is circRNA detection (BSJ identification) rather than full-length isoform assembly
  and quantification evaluation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: CircAST
  role: Produces the assembled circular transcripts, FPKM/TPM expression estimates,
    and assigned read counts that are stratified by relation index against simulation
    ground truth
  repo: https://github.com/xiaofengsong/CircAST
- name: pandas
  role: Loads and manipulates the CircAST output table for RI computation and bin-stratified
    metric calculation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:dd942592c86f34a6cd81641b1e715d95489af1f4768b2df75b05a2928de0db2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CircAST
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# simulation-benchmark-stratification

## Summary

Stratifies benchmark results on simulated circRNA data by a computed overlap metric — the relation index (RI), the cardinality of the equivalence class of overlapping back-splice junction (BSJ) sites — to quantify how locus complexity degrades CircAST's assembly precision, sensitivity, and quantification accuracy.

## When to use

Use when evaluating assembly/quantification accuracy on simulated data where performance may depend on structural complexity (e.g., overlapping genomic regions among circRNAs), requiring metrics stratified by a complexity bin such as RI 1-4 vs RI > 4.

## When NOT to use

- Data are real experimental reads without simulated ground-truth transcripts — sensitivity/precision against truth cannot be computed
- Each BSJ site is isolated (no overlapping genomic regions among circRNAs), so all sites fall in a single RI bin and stratification is uninformative
- Goal is circRNA detection (BSJ identification) rather than full-length isoform assembly and quantification evaluation

## Inputs

- CircAST output table on simulated data (chromosome, start, end, circular transcript name, junction read counts, FPKM, TPM, assigned reads)
- Simulated ground-truth circular transcript definitions with BSJ genomic coordinates
- Theoretical (simulated) expression levels (FPKM/TPM) per isoform

## Outputs

- Per-site RI assignments (BSJ site to relation-index value)
- RI-stratified performance metrics table (sensitivity, precision, expression correlation per RI bin) written to ri_performance_metrics.csv

## How to apply

Load the CircAST output table (columns: chromosome, start, end, circular transcript name, junction read counts, FPKM, TPM, assigned reads) with pandas. Define the BSJ overlap relation between simulated back-splice junction sites from their genomic coordinates and compute the relation index (RI) for each site as the cardinality of its equivalence class under this relation. Stratify BSJ sites into RI bins (RI 1-4 vs RI > 4), then per bin compare CircAST-assembled circular transcripts against simulated ground truth to compute sensitivity (fraction of true isoforms recovered) and precision (fraction of assembled isoforms that are true), and assess quantification accuracy as the correlation between theoretical and estimated FPKM/TPM. Write per-site RI assignments and the RI-stratified metrics table to ri_performance_metrics.csv and validate against the expected pattern: sensitivity above 93% for RI 1-4 declining to about 86% for RI > 4.

## Related tools

- **CircAST** (Produces the assembled circular transcripts, FPKM/TPM expression estimates, and assigned read counts that are stratified by relation index against simulation ground truth) — https://github.com/xiaofengsong/CircAST
- **pandas** (Loads and manipulates the CircAST output table for RI computation and bin-stratified metric calculation)

## Examples

```
df = pd.read_csv('CircAST_result.txt', sep='\t', header=None, names=['chr','start','end','name','gene','strand','nexons','sizes','offsets','junc_reads','fpkm','tpm','assigned']); df['RI'] = compute_relation_index(df)
```

## Evaluation signals

- Sensitivity stays above 93% for RI bins 1-4 and drops to about 86% for RI > 4
- Overall simulation metrics fall within reported ranges: sensitivity 85.63%-94.32% and precision 81.96%-87.55%
- Quantification correlation between theoretical and estimated FPKM/TPM falls within the reported 0.85-0.99 range
- ri_performance_metrics.csv contains both per-site RI assignments and the stratified metrics table with RI 1-4 vs RI > 4 bins
- Sum of site counts across RI bins equals the total number of simulated BSJ sites

## Limitations

- CircAST quantification and assembly are limited to exon-annotated circular transcripts; intronic or intergenic circular transcripts may be missed
- CircAST can be applied only to RNase R-treated samples, and remnant linear mRNA can generate false transcripts affecting assembly performance
- BSJ overlap is used as the sole complexity axis; other sources of ambiguity (e.g., read depth) are not captured by RI stratification

## Evidence

- [abstract] CircAST correctly assembled the full sequences of circRNAs with a sensitivity of 85.63%-94.32% and a precision of 81.96%-87.55%: "CircAST correctly assembled the full sequences of circRNAs with a sensitivity of 85.63%-94.32% and a precision of 81.96%-87.55%"
- [abstract] quantification correlation range: "CircAST quantified the expression of circRNA isoforms with correlation coefficients of 0.85-0.99 between theoretical and estimated values"
- [readme] output file columns: "11) Expression level (FPKM) 12) Expression level (TPM) 13) Read counts (assigned reads)"
- [discussion] RNase R limitation: "CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over."
- [discussion] splice graph assembly basis: "We constructed multiple splice graphs at each gene locus on the basis of the BSJ signatures of sequencing data without relying on any prior assumptions"
