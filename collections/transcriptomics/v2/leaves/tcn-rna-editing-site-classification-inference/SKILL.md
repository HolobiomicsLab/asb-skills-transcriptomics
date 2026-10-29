---
name: tcn-rna-editing-site-classification-inference
description: Use when you have tabix-indexed REDItools outTables from RNAseq BAMs
  and need to classify A-to-G substitutions as genuine A-to-I RNA editing vs RNA variants/sequencing
  artifacts without coupled DNA sequencing.
when_to_use_negative:
- Input is raw RNAseq FASTQ/BAM not yet processed through REDItools — REDInet requires
  tabix-indexed REDItools outTables as input
- Detecting non-A-to-I editing types (e.g. C-to-U) — REDInet is trained specifically
  to classify A-to-G substitutions as A-to-I editing
- A full empirical variant-calling workflow with matched WGS/WES DNA data is available
  and desired — empirical REDItools filtering may be preferable since REDInet was
  designed to avoid coupled genomic data
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3300
- http://edamontology.org/topic_0099
tools:
- name: REDInet
  role: TCN classifier run via REDInet_Inference.py or REDInet_Inference_light_ver.py
    on REDItools outTables to produce per-site editing probabilities
  repo: https://github.com/BioinfoUNIBA/REDInet
- name: REDItools
  role: Generates the RNAseq/DNA-RNA outTable pileups that REDInet consumes, run at
    low stringency to retrieve complete 101-base intervals
  repo: https://github.com/BioinfoUNIBA/REDItools
- name: REDItools3
  role: Faster re-implementation alternative for producing pileup profiles feeding
    REDInet
  repo: https://github.com/BioinfoUNIBA/REDItools3
- name: pysam
  role: Python wrapper used by REDInet for reading BAM/tabix-indexed outTable data
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:3ec983f687cb6e3c552c8cd27963705b2eedaacad380a1026ee8306b92ff84b7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/BioinfoUNIBA/REDInet
  repo_url: https://github.com/BioinfoUNIBA/REDInet
  tools:
  - Python
  - REDInet
  - REDItools
  - REDItools3
  - pysam
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3300
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# tcn-rna-editing-site-classification-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply REDInet, a temporal convolutional network (WaveNet-inspired) classifier, to label A-to-G substitutions in RNAseq data as A-to-I RNA editing events using nucleotide frequencies from 101-base windows, without requiring matched WGS/WES data.

## When to use

Use when you have tabix-indexed REDItools outTables from RNAseq BAMs and need to classify A-to-G substitutions as genuine A-to-I RNA editing vs RNA variants/sequencing artifacts without coupled DNA sequencing.

## When NOT to use

- Input is raw RNAseq FASTQ/BAM not yet processed through REDItools — REDInet requires tabix-indexed REDItools outTables as input
- Detecting non-A-to-I editing types (e.g. C-to-U) — REDInet is trained specifically to classify A-to-G substitutions as A-to-I editing
- A full empirical variant-calling workflow with matched WGS/WES DNA data is available and desired — empirical REDItools filtering may be preferable since REDInet was designed to avoid coupled genomic data

## Inputs

- Tabix-indexed, bgzip-compressed REDItools outTable (.gz) from RNAseq BAM pileups
- RNAseq BAM files prepared per the REDItools protocol (aligned to GRCh37 or GRCh38)
- Reference genome FASTA (GRCh37 and/or GRCh38, e.g. GENCODE release 46 primary assembly) for imputation of missing 101-nt windows
- REDItools outTable folder path and outTable names (for the multiprocessing inference script)

## Outputs

- Per-site predictions: snp_proba, ed_proba, and y_hat (edit/non-edit call) for each A-to-G position
- <outTable name>_discarded.txt listing genomic positions filtered out by minimum coverage, A-to-G substitution rate, or guanosine count thresholds

## How to apply

Prepare RNAseq BAMs through a REDItools version (e.g. REDItoolDnaRna.py with low-stringency settings such as -t 40 -c 0,1 -m 0,255 -v 1 -q 0,30 -s 2 -g 2) so complete 101-base intervals centered on each putative editing position are captured, then compress with bgzip and index with tabix (-s 1 -b 2 -e 2 -c R). Launch REDInet_Inference.py (multiprocessing, with optional missing-value imputation on GRCh37/GRCh38) or REDInet_Inference_light_ver.py on the indexed outTables, setting minimum coverage (--C 50), minimum A-to-G substitutions (--M 3), and minimum A-to-G frequency (--F 0.01); the model consumes 8 x 101 feature matrices of RNAseq nucleotide frequencies per site. Rationale: REDInet was trained on ~13 million examples from 8404 GTEx RNAseq experiments and REDIportal events, so it classifies editing harnessing RNAseq nucleotide frequencies of 101-base windows alone. Outputs per-site snp_proba, ed_proba, and y_hat predictions, with positions failing coverage/substitution filters written to a <name>_discarded.txt file. Correct application is judged by computing accuracy, specificity, and precision against a bona fide ground truth (e.g. U87 A-to-G RNAseq/DNAseq contrasted site set), expecting values on the order of the reported AUC 0.99, accuracy >97%, specificity >96%, precision >97%.

## Related tools

- **REDInet** (TCN classifier run via REDInet_Inference.py or REDInet_Inference_light_ver.py on REDItools outTables to produce per-site editing probabilities) — https://github.com/BioinfoUNIBA/REDInet
- **REDItools** (Generates the RNAseq/DNA-RNA outTable pileups that REDInet consumes, run at low stringency to retrieve complete 101-base intervals) — https://github.com/BioinfoUNIBA/REDItools
- **REDItools3** (Faster re-implementation alternative for producing pileup profiles feeding REDInet) — https://github.com/BioinfoUNIBA/REDItools3
- **pysam** (Python wrapper used by REDInet for reading BAM/tabix-indexed outTable data) — https://github.com/pysam-developers/pysam

## Examples

```
python3 ../REDInet/Package/Utilities/REDInet_Inference.py --I <REDItools_outTable_folderpath> --O <output_folder> --A GRCh37 --C 50 --M 3 --N <outTable_name>
```

## Evaluation signals

- Compute AUC, accuracy, specificity, and precision of REDInet y_hat predictions against an independent bona fide ground truth (e.g. U87 set) and confirm values consistent with reported AUC 0.99, accuracy >97%, specificity >96%, precision >97%
- Check that the <name>_discarded.txt file accounts for every site excluded by --C (min coverage), --M (min AG count), and --F (min AG frequency) thresholds
- Verify outTables are bgzip-compressed and tabix-indexed with -s 1 -b 2 -e 2 -c R before inference; inference fails otherwise
- Confirm each retained site has a fully covered 101-nucleotide window (or that imputation was applied with --A GRCh37/GRCh38 for missing nucleotides)
- On training/validation/test-style data, expect accuracy >99%; on held-out cell-line datasets expect accuracy >95% with specificity >93% and precision >96%

## Limitations

- REDInet was trained on GTEx/REDIportal A-to-I events; performance on tissues or conditions far outside the GTEx atlas is not established
- Sites with missing nucleotides in the 101-nt window must be imputed (BETA feature in the multiprocessing version using reference genome) or discarded
- Imputation in the multiprocessing REDInet_Inference.py supports only hg38 or hg19 builds; other builds require the light version
- Requires Python 3.9.0 environment with tensorflow 2.14.0, pysam 0.22.1, and other pinned package versions
- Performance comparisons depend on following each program's guidelines; REDInet's advantage over EditPredict (AUC 0.51 on U87) was measured on the paper's bona fide site sets

## Evidence
