---
name: binary-classification-concordance-analysis
description: Use when a per-sample binary call (e.g., mCLL vs uCLL from RNA-seq IGHV
  percent identity) must be compared sample-by-sample against matched clinical gold-standard
  labels to quantify agreement.
when_to_use_negative:
- No matched gold-standard (e.g., PCR-Sanger) labels exist for the samples, so agreement
  metrics cannot be computed
- The outcome is not a two-class call — continuous IGHV percent-identity values should
  be compared directly rather than collapsed to mCLL/uCLL concordance
- Input BAMs are not aligned to hg38 (GRCh38), coordinate-sorted, and indexed, or
  have far fewer than the required ~25M reads, so the predictions feeding the comparison
  are invalid
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2830
- http://edamontology.org/topic_3308
tools:
- name: CRIS
  role: Produces the predicted binary IGHV SHM status (mCLL vs uCLL) from each RNA-seq
    BAM via IgBLAST percent identity to the top germline IGHV hit, which is then compared
    to Sanger labels
  repo: https://github.com/Rashedul/CRIS
- name: igblast
  role: Generates the per-sample output file whose top germline IGHV gene hit and
    percent identity are parsed to derive the predicted class
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:34e9534f473b25e3f24504f1d5b3eb70390008ff3c25e3e2d8e1ca24e2a3c5db
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
  - seqkit
  - trinity
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2830
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# binary-classification-concordance-analysis

## Summary

Validates a binary classifier (e.g., CRIS IGHV somatic hypermutation status calls, mutated mCLL vs unmutated uCLL) against an independent gold-standard label set (clinical PCR-Sanger classification) by computing per-sample concordance, accuracy, sensitivity, and specificity and flagging discordant cases.

## When to use

Use when a per-sample binary call (e.g., mCLL vs uCLL from RNA-seq IGHV percent identity) must be compared sample-by-sample against matched clinical gold-standard labels to quantify agreement.

## When NOT to use

- No matched gold-standard (e.g., PCR-Sanger) labels exist for the samples, so agreement metrics cannot be computed
- The outcome is not a two-class call — continuous IGHV percent-identity values should be compared directly rather than collapsed to mCLL/uCLL concordance
- Input BAMs are not aligned to hg38 (GRCh38), coordinate-sorted, and indexed, or have far fewer than the required ~25M reads, so the predictions feeding the comparison are invalid

## Inputs

- Per-sample prediction outputs (e.g., *.IgBLAST_out.txt files giving top germline IGHV hit and percent identity) for each of the 51 BAM files
- Matched clinical PCR-Sanger IGHV hypermutation labels (gold-standard binary mCLL/uCLL classes) for the phs000435.v3 cohort
- Classification table of per-sample IGHV gene/percent-identity pairs and derived CRIS calls, sorted by sample ID

## Outputs

- Per-sample concordance table of CRIS call vs Sanger call, keyed by sample ID, with discordant samples flagged
- Summary metrics: accuracy (98.3%), sensitivity (100%), and specificity (97.3%) for the cohort
- List of discordant samples with class directions (e.g., DFCI-5121: Sanger mCLL vs CRIS uCLL)

## How to apply

Run the prediction method (CRIS via CRIS.sh with default -threads 4 and -memory 16G) on each sample's hg38-aligned, coordinate-sorted, indexed BAM, then parse each IgBLAST output file to extract the top germline IGHV gene hit and its percent identity (benchmark: IGHV3-74 at 94.6% for the SRR1814049 test run). Classify each sample as mutated mCLL or unmutated uCLL from percent identity to the top germline IGHV hit and compile a per-sample table of gene/identity pair and call, sorted by sample ID. Join the computational calls to the matched clinical PCR-Sanger labels and compute concordance, accuracy, sensitivity, and specificity. In the phs000435.v3 (Wang et al., 2011) cohort of 51 CLL samples, CRIS matched Sanger in 50/51 cases; the acceptance criterion is 98.3% accuracy, 100% sensitivity, 97.3% specificity, with DFCI-5121 (Sanger mCLL vs CRIS uCLL) as the single expected discordant sample.

## Related tools

- **CRIS** (Produces the predicted binary IGHV SHM status (mCLL vs uCLL) from each RNA-seq BAM via IgBLAST percent identity to the top germline IGHV hit, which is then compared to Sanger labels) — https://github.com/Rashedul/CRIS
- **igblast** (Generates the per-sample output file whose top germline IGHV gene hit and percent identity are parsed to derive the predicted class)

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam
```

## Evaluation signals

- Concordance count equals 50/51 samples called identically to PCR-Sanger (98.3% accuracy, 100% sensitivity, 97.3% specificity)
- Exactly one discordant sample, DFCI-5121, with Sanger mCLL vs CRIS uCLL direction
- Test-run benchmark reproduced: top germline IGHV gene IGHV3-74 with percent identity 94.6
- Classification table contains a gene/identity pair and a binary call for every one of the 51 sample IDs, with no missing entries
- Sensitivity is 1.0 (no unmutated/uCLL false negatives) and specificity ~0.973, consistent with a single mCLL-side discordance

## Limitations

- Concordance is cohort- and platform-dependent: the validated figure (98.3%) applies to phs000435.v3 with PCR-Sanger labels; other cohorts or label sources may differ
- A single discordant case (DFCI-5121, Sanger mCLL vs CRIS uCLL) persists even in the validated setting, so 100% agreement should not be expected
- CRIS requires a minimum of 25M sequence reads per sample and 16GB RAM / 4 threads; input BAMs must be BWA-aligned to hg38 and coordinate-sorted, or calls feeding the comparison will be unreliable

## Evidence

- [other] CRIS calls were identical to the Sanger-based calls in 50/51 cases, with 98.3% accuracy, 100% sensitivity and 97.3% specificity.: [evidence span withheld — anchor status: not_found]
- [methods] Test-run benchmark for output parsing.: "- IGHV gene: IGHV3-74
- Percent identity: 94.6"
- [intro] Validation against gold standard.: "CRIS has been validated against clinical PCR-Sanger based hypermutation classification in the context of Chronic Lymphocytic Leukemia."
- [readme] Input requirements for the predictions being compared.: "bam file must be aligned to hg38 (GRCh38) genome build using [BWA](http://bio-bwa.sourceforge.net/), coordinate-sorted and indexed"
- [abstract] Cohort-level validation claim.: "We show that the SHM status identified by CRIS using the entire IGHV gene segment is highly concordant with clinical classification in three independent chronic lymphocytic leukemia patient cohorts"
