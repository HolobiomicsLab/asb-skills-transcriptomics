---
name: clinical-concordance-correlation-analysis
description: Use when a computational IGHV mutation classifier's calls must be validated
  against clinical Sanger-based mCLL/uCLL labels, to quantify concordance and correlation
  of percent-mutation values in a CLL RNA-seq cohort.
when_to_use_negative:
- Input BAMs are not aligned to the hg38 (GRCh38) genome build with BWA and coordinate-sorted/indexed
  — CRIS calls feeding this comparison would be invalid
- Samples have fewer than 25M sequence reads, below CRIS's stated minimum for reliable
  IGHV reconstruction
- The goal is T-cell receptor (TR) repertoire or non-IGHV mutation analysis rather
  than CLL IGHV hypermutation status classification
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CRIS
  role: Produces the per-sample IGHV gene, percent identity, and IgBLAST alignment
    outputs whose mCLL/uCLL calls are compared against clinical Sanger labels
  repo: https://github.com/Rashedul/CRIS
- name: igblast
  role: Generates the IgBLAST_out.txt files containing percent identity and alignment
    between assembled Ig transcripts and top germline IGHV hits, the values used for
    classification and correlation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:4f2c1f4333b5a93db767b13c4f3c06b3f1fe7a2b43938f31ee5b6d3098440161
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - CRIS
  - Docker
  - blast
  - igblast
  - jellyfish
  - picard
  - salmon
  - sambamba
  - seqkit
  - trinity
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# clinical-concordance-correlation-analysis

## Summary

Compares computationally inferred IGHV somatic hypermutation (mCLL/uCLL) calls from CRIS RNA-seq reconstructions against gold-standard clinical PCR-Sanger classifications, reporting per-sample concordance and Pearson correlation between percent-mutation values (target: 16/16 concordant, r = 0.95 in GSE66228).

## When to use

Use when a computational IGHV mutation classifier's calls must be validated against clinical Sanger-based mCLL/uCLL labels, to quantify concordance and correlation of percent-mutation values in a CLL RNA-seq cohort.

## When NOT to use

- Input BAMs are not aligned to the hg38 (GRCh38) genome build with BWA and coordinate-sorted/indexed — CRIS calls feeding this comparison would be invalid
- Samples have fewer than 25M sequence reads, below CRIS's stated minimum for reliable IGHV reconstruction
- The goal is T-cell receptor (TR) repertoire or non-IGHV mutation analysis rather than CLL IGHV hypermutation status classification

## Inputs

- Per-sample CRIS output files: *.IgBLAST_out.txt (percent identity and alignment to top germline IGHV hits)
- *.ig-transcripts.sortedbyTPM.txt and *.ig-transcripts.sortedbyTPM.fasta (Ig transcript IDs/sequences sorted by TPM)
- Clinical gold-standard mCLL/uCLL labels from Sanger-based IGHV mutation testing (e.g. Blachly et al. 2015, Table 2, for GSE66228)

## Outputs

- Per-sample CSV table cris_gse66228_ighv_status.csv with columns for dominant IGHV gene, percent mutation, and mCLL/uCLL classification
- Concordance count (e.g. N concordant / N total vs Sanger labels)
- Pearson correlation coefficient between CRIS and clinical percent-mutation values

## How to apply

For each sample, collect CRIS outputs (*.IgBLAST_out.txt for percent identity to the top germline IGHV hit, *.ig-transcripts.sortedbyTPM.txt and .fasta) and extract the dominant IGHV gene, percent mutation, and mCLL/uCLL call into a per-sample table (cris_gse66228_ighv_status.csv). Classification threshold: samples are called mutated (mCLL) vs unmutated (uCLL) by comparing CRIS percent identity of the assembled Ig transcript to the top germline IGHV gene against the clinical Sanger-based label (Table 2 reference calls). Then tally concordance (CRIS call == clinical call) per sample and compute the Pearson correlation between CRIS and clinical percent-mutation values. Decision point: perfect concordance (e.g. 16/16 in GSE66228) and high correlation (r ≈ 0.95) indicate the RNA-seq-based classifier agrees with the gold standard; discordant samples should be inspected in the IgBLAST alignment output before blaming the classifier. Rationale: the article validated CRIS specifically against clinical PCR-Sanger hypermutation classification in CLL cohorts, so agreement with those labels is the skill's success criterion.

## Related tools

- **CRIS** (Produces the per-sample IGHV gene, percent identity, and IgBLAST alignment outputs whose mCLL/uCLL calls are compared against clinical Sanger labels) — https://github.com/Rashedul/CRIS
- **igblast** (Generates the IgBLAST_out.txt files containing percent identity and alignment between assembled Ig transcripts and top germline IGHV hits, the values used for classification and correlation)

## Examples

```
python -c "import pandas as pd; d=pd.read_csv('cris_gse66228_ighv_status.csv'); print(sum(d.cris_call==d.clinical_call),'/',len(d)); print(d.cris_pct.corr(d.clinical_pct))"
```

## Evaluation signals

- GSE66228 benchmark: 16/16 CRIS mCLL/uCLL calls concordant with clinical Sanger-based classification
- Pearson correlation between CRIS and clinical percent-mutation values reaches r = 0.95
- Compiled cris_gse66228_ighv_status.csv contains one row per sample with dominant IGHV gene, percent mutation, and classification populated
- Each sample's three CRIS output products exist (*.IgBLAST_out.txt, *.ig-transcripts.sortedbyTPM.txt, *.ig-transcripts.sortedbyTPM.fasta)
- Sanity check on the supplied test BAM: expected IGHV3-74 with percent identity 94.6

## Limitations

- CRIS requires BAMs aligned to hg38 (GRCh38) with BWA, coordinate-sorted and indexed; misaligned or unsorted BAMs invalidate the calls being compared
- Minimum 25M reads per sample, minimum 16GB RAM and 4 threads; fewer reads or resources can compromise Ig transcript assembly
- Concordance was demonstrated in CLL cohorts (three independent patient cohorts including GSE66228); generalization to other diseases' IGHV classification was not validated here

## Evidence

- [other] CRIS classification of mutated/unmutated CLL from RNA-seq showed perfect concordance (16/16) with Sanger-based clinical calls in the GSE66228 dataset: [evidence span withheld — anchor status: not_found]
- [abstract] SHM status identified by CRIS using the entire IGHV gene segment is highly concordant with clinical classification: "We show that the SHM status identified by CRIS using the entire IGHV gene segment is highly concordant with clinical classification in three independent chronic lymphocytic leukemia patient cohorts"
- [readme] CRIS has been validated against clinical PCR-Sanger based hypermutation classification in the context of Chronic Lymphocytic Leukemia.: "CRIS has been validated against clinical PCR-Sanger based hypermutation classification in the context of Chronic Lymphocytic Leukemia."
- [readme] IgBLAST_out.txt contains percent identity and alignment to top germline IGHV hits, the quantitative basis for concordance comparison: "Contains the percent identity and alignment between Ig-transcript and top germline IGHV gene hits."
