---
name: unpaired-students-t-test
description: Use when two sample groups' circRNA expression values (e.g., CIRCscore
  or FPBcirc per circRNA from CLEAR quant.txt files) must be compared for a statistically
  significant distributional shift, as in HCC vs paired-normal ribo− RNA-seq.
when_to_use_negative:
- CIRCscore computation is unsuitable for poly(A)−/ribo− or RNase R-treated RNA-seq,
  since polyadenylated linear RNAs are largely depleted and cannot serve as an accurate
  linear RNA background
- Do not test raw FPBcirc values alone to separate groups; the source task showed
  FPBcirc fails to distinguish HCC from paired normal samples (P = 0.99)
- Input circRNA quantification must come from CLEAR quant.txt output (BSJ/SJ-based
  FPB values); do not feed in FPM/FPKM-style counts that are unscaled between circRNA
  and linear RNA
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: CIRCexplorer3-CLEAR
  role: Quantifies circRNA and cognate linear RNA expression per sample via clear_quant,
    producing quant.txt with FPBcirc, FPBlinear, and CIRCscore used for the group
    comparison
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Underlying circRNA detection engine (>=2.3.6) invoked within clear_quant to
    identify back-splicing junction reads
  repo: https://github.com/YangLab/CIRCexplorer2
- name: HISAT2
  role: Maps RNA-seq reads (>=2.0.5) against the hg38 index during clear_quant prior
    to FPB quantification
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
- name: scipy.stats.ttest_ind
  role: Performs the two-tailed unpaired Student's t-test comparing HCC vs normal
    FPBcirc and CIRCscore values
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:06f678425883cc45e4c7dc09a94e898efd6dd8deb5ab8c7c8ae9568357fa165b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLab/CLEAR
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - HISAT2
  - scipy.stats.ttest_ind
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# unpaired-students-t-test

## Summary

Applies a two-tailed unpaired Student's t-test (scipy.stats.ttest_ind) to compare circRNA expression distributions (FPBcirc and CIRCscore from CIRCexplorer3-CLEAR quant.txt outputs) between 20 HCC tumors and paired normal liver samples, testing whether CIRCscore normalization reduces individual differences that FPBcirc alone cannot resolve.

## When to use

Use when two sample groups' circRNA expression values (e.g., CIRCscore or FPBcirc per circRNA from CLEAR quant.txt files) must be compared for a statistically significant distributional shift, as in HCC vs paired-normal ribo− RNA-seq.

## When NOT to use

- CIRCscore computation is unsuitable for poly(A)−/ribo− or RNase R-treated RNA-seq, since polyadenylated linear RNAs are largely depleted and cannot serve as an accurate linear RNA background
- Do not test raw FPBcirc values alone to separate groups; the source task showed FPBcirc fails to distinguish HCC from paired normal samples (P = 0.99)
- Input circRNA quantification must come from CLEAR quant.txt output (BSJ/SJ-based FPB values); do not feed in FPM/FPKM-style counts that are unscaled between circRNA and linear RNA

## Inputs

- ribo-depleted paired-end RNA-seq FASTQ files for 20 HCC and 20 paired normal liver samples (GEO accession GSE77509)
- hg38 genome FASTA, hg38 HISAT2 index, hg38 bowtie1 index (TopHat-Fusion), and annotation GTF
- Per-sample CLEAR quant/quant.txt files containing FPBcirc, FPBlinear, and CIRCscore per circRNA

## Outputs

- circscore_ttest_results.csv containing the two-tailed unpaired t-test P-values for FPBcirc (HCC vs normal) and CIRCscore (HCC vs normal)
- Cumulative distribution curves of FPBcirc and CIRCscore for the HCC mixture versus the normal mixture

## How to apply

Run CIRCexplorer3-CLEAR clear_quant on each ribo− RNA-seq FASTQ (paired -1/-2 mates, hg38 FASTA, HISAT2 index, bowtie1 index for TopHat-Fusion, annotation GTF), then parse output_dir/quant/quant.txt from each of the 40 libraries to extract FPBcirc, FPBlinear, and CIRCscore per circRNA. Build cumulative distribution curves of FPBcirc and CIRCscore for the HCC mixture versus the normal mixture. Rationale: CIRCscore = FPBcirc/FPBlinear uses the cognate linear RNA background to normalize circRNA expression, whereas raw FPBcirc reflects mostly low circRNA abundance and should fail to separate groups. Run scipy.stats.ttest_ind separately for FPBcirc and CIRCscore (two-tailed, unpaired), and write both P-values plus cumulative distribution data to a results table (circscore_ttest_results.csv).

## Related tools

- **CIRCexplorer3-CLEAR** (Quantifies circRNA and cognate linear RNA expression per sample via clear_quant, producing quant.txt with FPBcirc, FPBlinear, and CIRCscore used for the group comparison) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Underlying circRNA detection engine (>=2.3.6) invoked within clear_quant to identify back-splicing junction reads) — https://github.com/YangLab/CIRCexplorer2
- **HISAT2** (Maps RNA-seq reads (>=2.0.5) against the hg38 index during clear_quant prior to FPB quantification) — https://ccb.jhu.edu/software/hisat2/index.shtml
- **scipy.stats.ttest_ind** (Performs the two-tailed unpaired Student's t-test comparing HCC vs normal FPBcirc and CIRCscore values)

## Examples

```
from scipy import stats; stats.ttest_ind(hcc_circscore, normal_circscore)
```

## Evaluation signals

- FPBcirc t-test between HCC and normal mixtures reproduces P = 0.99, confirming raw circRNA FPB cannot separate the groups
- CIRCscore t-test between HCC and normal mixtures reproduces P = 3.59 × 10−5, confirming the linear-background normalization reveals low circRNA expression in HCC
- circscore_ttest_results.csv contains both P-values (FPBcirc and CIRCscore) plus the cumulative distribution data
- Each of the 40 libraries yields a quant/quant.txt with FPBcirc, FPBlinear, and CIRCscore fields populated per circRNA

## Limitations

- CIRCscore cannot be applied to poly(A)−/ribo− or RNase R-treated datasets because polyadenylated linear RNAs are largely depleted, making linear RNA quantification (and thus FPBcirc/FPBlinear) inaccurate
- The unpaired t-test ignores the paired tumor/normal design of GSE77509; if pairing matters, an alternative paired test would be more powerful
- Threshold of FPB for choosing circRNAs to filter linear SJ defaults to 1, and --ratio (default 1) adjusts the circ/linear comparison; changing these alters CIRCscore and hence the test result

## Evidence

- [abstract] Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore: "Comparison of circular and linear RNA expression levels is directly achieved by dividing FPBcirc by FPBlinear to generate a CIRCscore"
- [readme] output file and output fields: "* output_dir/quant/quant.txt"
- [readme] CIRCscore output column: "CIRCscore   | Relative expression of circRNA"
- [intro] FPB quantification of circular and linear RNA: "RNA-seq fragments mapped to circRNA-specific BSJ sites or linear RNA-specific SJ sites are individually normalized to evaluate circular or linear RNA expression"
- [methods] CLEAR installation: "git clone https://github.com/YangLab/CLEAR"
