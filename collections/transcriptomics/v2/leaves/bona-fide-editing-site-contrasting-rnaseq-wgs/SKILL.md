---
name: bona-fide-editing-site-contrasting-rnaseq-wgs
description: Use when you need an independent labeled benchmark of A-to-G positions
  from a cell line/sample with matched RNAseq and WGS/WES data, to evaluate editing
  classifiers like REDInet on data never seen during training.
when_to_use_negative:
- When only RNAseq data are available without matched WGS/WES DNA data — DNA evidence
  is required to separate true edits from RNA variants and SNPs
- When the goal is de novo editing detection rather than building a labeled benchmark
  set for classifier evaluation
- When interested in non A-to-I editing types, since this procedure only labels A-to-G/A-to-I
  positions
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3673
tools:
- name: REDItools
  role: Runs REDItoolDnaRna.py at low stringency over RNAseq BAMs (with DNA evidence)
    to produce the outTables of candidate A-to-G positions feeding the ground truth
    set
  repo: https://github.com/BioinfoUNIBA/REDItools
- name: REDInet
  role: Downstream classifier (REDInet_Inference.py) evaluated against the bona fide
    ground truth labels on tabix-indexed outTables
  repo: https://github.com/BioinfoUNIBA/REDInet.git
- name: pysam
  role: Python wrapper of the samtools C-API used by the REDItools/REDInet Python
    stack to read and manipulate BAM alignment files
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:ed0003c0de469d5f0de4a0bc495eb2b83e0a0336913487121746fab8799a45ce
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/BioinfoUNIBA/REDInet
  repo_url: https://github.com/BioinfoUNIBA/REDItools
  tools:
  - Python
  - REDInet
  - REDItools
  - pysam
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# bona-fide-editing-site-contrasting-rnaseq-wgs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds an independent bona fide A-to-I RNA editing ground truth set (e.g., >2800 U87 sites) by contrasting RNAseq evidence with matched WGS/WES DNA data: positives are A-to-G changes absent from DNA, negatives are A-to-G changes present in both, i.e., RNA variants not due to editing.

## When to use

Use when you need an independent labeled benchmark of A-to-G positions from a cell line/sample with matched RNAseq and WGS/WES data, to evaluate editing classifiers like REDInet on data never seen during training.

## When NOT to use

- When only RNAseq data are available without matched WGS/WES DNA data — DNA evidence is required to separate true edits from RNA variants and SNPs
- When the goal is de novo editing detection rather than building a labeled benchmark set for classifier evaluation
- When interested in non A-to-I editing types, since this procedure only labels A-to-G/A-to-I positions

## Inputs

- RNAseq BAM files aligned to GRCh37 (e.g., U87 SRX110671-SRX110674)
- Matched WGS/WES BAM or variant evidence (e.g., U87 SRX5466662)
- REDIportal database for annotating known editing sites
- GRCh37 (or GRCh38) reference genome FASTA indexed with samtools faidx

## Outputs

- Tabix-indexed, bgzip-compressed REDItools outTables over candidate A-to-G positions
- Labeled bona fide site set (>2800 sites) with positive/negative ground truth labels per position
- Evaluation metrics (AUC, accuracy, specificity, precision) of classifier predictions against the labels

## How to apply

Retrieve RNAseq reads (e.g., U87 SRX110671-SRX110674) and matched WGS (e.g., SRX5466662) from SRA, prepare RNAseq BAMs per the REDItools protocol aligned to GRCh37. Run REDItools REDItoolDnaRna.py at low stringency (-t 40 -c 0,1 -m 0,255 -v 1 -q 0,30 -s 2 -g 2) over candidate sites to produce outTables, compress with bgzip and index with tabix (-s 1 -b 2 -e 2 -c R). Label positives as sites with A-to-G evidence in RNAseq, no base change in DNAseq, and presence in REDIportal; label negatives as sites with A-to-G in both RNAseq and DNAseq, retaining only those with an RNAseq A-to-G rate >= 0.4 and a DNAseq rate diverging from the RNAseq rate <5%. Enforce support thresholds of 50 RNAseq reads and 10 DNAseq reads, require each A-to-G change to be backed by at least 3 Gs or a substitution rate >=1%, and keep only sites whose 101-base window is fully covered by RNAseq reads. Judge success by computing AUC, accuracy, specificity, and precision of classifier predictions against these labels (e.g., REDInet on U87: AUC 0.99, accuracy >97%, specificity >96%, precision >97%).

## Related tools

- **REDItools** (Runs REDItoolDnaRna.py at low stringency over RNAseq BAMs (with DNA evidence) to produce the outTables of candidate A-to-G positions feeding the ground truth set) — https://github.com/BioinfoUNIBA/REDItools
- **REDInet** (Downstream classifier (REDInet_Inference.py) evaluated against the bona fide ground truth labels on tabix-indexed outTables) — https://github.com/BioinfoUNIBA/REDInet.git
- **pysam** (Python wrapper of the samtools C-API used by the REDItools/REDInet Python stack to read and manipulate BAM alignment files) — https://github.com/pysam-developers/pysam

## Examples

```
python ../REDItools/main/REDItoolDnaRna.py -o ../REDInet/Package/Data -i sample.bam -f ../REDInet/Package/Utilities/GRCh37.primary_assembly.genome.fa -t 40 -c 0,1 -m 0,255 -v 1 -q 0,30 -s 2 -g 2 -S
```

## Evaluation signals

- Every positive site shows A-to-G evidence in RNAseq, no base change in DNAseq, and is present in REDIportal
- Every negative site has RNAseq A-to-G rate >= 0.4 with a DNAseq A-to-G rate within 5% of the RNAseq rate
- All retained sites are supported by >= 50 RNAseq reads and >= 10 DNAseq reads, with A-to-G changes backed by >= 3 Gs or substitution rate >= 1%
- All final sites have their 101-base window fully covered by RNAseq reads
- Classifier evaluation against the labels reproduces expected performance (U87: AUC 0.99, accuracy >97%, specificity >96%, precision >97%)

## Limitations

- Requires high-quality RNAseq data coupled with WGS/WES data plus multiple filtering steps, which the paper notes is the time-consuming burden this benchmark approach shares with empirical methods
- Empirical filters are needed to remove background noise, sequencing errors, and artifacts; low-coverage windows cannot be labeled
- Complete ground truth datasets and metric formulas live in the paper's GitHub repository and Supplementary Methods, which may not be fully self-contained in the main text

## Evidence

- [discussion] we selected as positives all sites with A-to-G evidence in RNAseq, no base change in DNAseq (WGS or WES data), and present in the latest REDIportal: "we selected as positives all sites with A-to-G evidence in RNAseq, no base change in DNAseq (WGS or WES data), and present in the latest REDIportal"
- [discussion] we retained as negatives all sites with an RNAseq A-to-G rate >= 0.4 and a comparable DNAseq A-to-G rate diverging from the RNAseq A-to-G rate <5%.: [evidence span withheld — anchor status: not_found]
