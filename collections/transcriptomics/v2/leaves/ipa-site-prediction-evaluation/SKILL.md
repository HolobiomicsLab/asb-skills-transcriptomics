---
name: ipa-site-prediction-evaluation
description: Use when comparing RNA-seq-based IPA site prediction tools for sensitivity,
  precision, and IPA-usage quantification accuracy, especially when ground-truth polyA
  sites are known and simulated reads at controlled coverage (10X–50X) are feasible.
when_to_use_negative:
- Ground truth consists only of 3'UTR APA events rather than intronic polyadenylation
  — tools like DaPars, Aptardi, APAtrap and TAPAS cover 3'UTR APA, but this benchmark
  targets IPA specifically
- No predefined/known polyA sites or ground-truth relative usage are available, so
  TP/FP classification and |ΔPAU| error cannot be computed
- The goal is novel IPA discovery in real biological samples (e.g. monocyte activation,
  scRNA-seq) rather than controlled tool benchmarking
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: InPACT
  role: Method under test; identifies skipped and composite IPA sites and quantifies
    IPA usage (PAU) from sorted BAM plus RefSeq GTF
  repo: https://github.com/YY-TMU/InPACT
- name: APAIQ
  role: Comparator tool for IPA site identification and quantification; not designed
    for IPA analysis and cannot assemble IPA isoforms
- name: IPAFinder
  role: Comparator tool for IPA site identification and quantification; relies on
    read coverage fluctuations
- name: Polyester
  role: R package used to simulate RNA-seq reads at 10X–50X coverage from transcripts
    with predefined polyA sites
- name: Salmon
  role: Quantifies transcript abundance of assembled IPA isoforms (with gffread-extracted
    transcripts.fa) for IPA-usage calculation
  repo: https://github.com/COMBINE-lab/salmon
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:08b9f296970de74b7f7065ef22d04aa22fcafda130731f483b5d400fa58bda6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - APAIQ
  - IPAFinder
  - InPACT
  - Polyester
  - Salmon
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ipa-site-prediction-evaluation

## Summary

Benchmark computational IPA (intronic polyadenylation) site identification and quantification tools — InPACT, APAIQ, IPAFinder — on simulated RNA-seq data generated with Polyester at 10X–50X coverage, scoring sensitivity, precision, and per-site quantification error (|ΔPAU|).

## When to use

Use when comparing RNA-seq-based IPA site prediction tools for sensitivity, precision, and IPA-usage quantification accuracy, especially when ground-truth polyA sites are known and simulated reads at controlled coverage (10X–50X) are feasible.

## When NOT to use

- Ground truth consists only of 3'UTR APA events rather than intronic polyadenylation — tools like DaPars, Aptardi, APAtrap and TAPAS cover 3'UTR APA, but this benchmark targets IPA specifically
- No predefined/known polyA sites or ground-truth relative usage are available, so TP/FP classification and |ΔPAU| error cannot be computed
- The goal is novel IPA discovery in real biological samples (e.g. monocyte activation, scRNA-seq) rather than controlled tool benchmarking

## Inputs

- Transcript set with predefined polyA sites (ground-truth IPA annotations)
- Simulated RNA-seq FASTQ/BAM datasets from Polyester at 10X, 20X, 30X, 40X, 50X coverage (five replicates each)
- Coordinate-sorted BAM alignments of simulated reads
- Reference gene annotation GTF (RefSeq, e.g. hg38.109.20211119.ncbiRefSeq.gtf)
- InPACT candidate polyA site SAF file (InPACT_polyAsites.hg38.saf)

## Outputs

- Per-tool, per-coverage sensitivity and precision values (averaged over five replicates)
- Per-site absolute quantification error |ΔPAU| for each tool
- Line plots of sensitivity/precision versus sequencing coverage for InPACT, APAIQ, IPAFinder
- Cumulative distribution plots of |ΔPAU| error across identified IPA sites

## How to apply

Simulate RNA-seq datasets with the R package Polyester at sequencing coverage levels from 10X to 50X in 10X increments, generating five replicates per coverage from a transcript set with predefined polyA sites. Run InPACT, APAIQ, and IPAFinder on each simulated dataset; classify predicted sites within 50 nt of a predefined polyA site as true positives and others as false positives, then compute Sensitivity = TP / Predefined and Precision = TP / (TP + FP) per coverage, averaged across the five replicates. For quantification, estimate relative usage (PAU) of each identified IPA site with each tool, take the ground-truth relative usage of the predefined sites as reference, and compute per-site error as the absolute difference |ΔPAU|. Plot sensitivity and precision versus sequencing coverage for the three tools and cumulative distributions of |ΔPAU| across identified sites comparing InPACT and IPAFinder. Judge success by reproducing the reported trend: InPACT attains the highest sensitivity/precision across the 10X–50X range and the lowest IPA-usage error, with ~90% sensitivity at 50X.

## Related tools

- **InPACT** (Method under test; identifies skipped and composite IPA sites and quantifies IPA usage (PAU) from sorted BAM plus RefSeq GTF) — https://github.com/YY-TMU/InPACT
- **APAIQ** (Comparator tool for IPA site identification and quantification; not designed for IPA analysis and cannot assemble IPA isoforms)
- **IPAFinder** (Comparator tool for IPA site identification and quantification; relies on read coverage fluctuations)
- **Polyester** (R package used to simulate RNA-seq reads at 10X–50X coverage from transcripts with predefined polyA sites)
- **Salmon** (Quantifies transcript abundance of assembled IPA isoforms (with gffread-extracted transcripts.fa) for IPA-usage calculation) — https://github.com/COMBINE-lab/salmon

## Examples

```
InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5
```

## Evaluation signals

- InPACT reaches approximately 90% sensitivity at 50X coverage and the highest sensitivity/precision across the 10X–50X range
- Predicted sites within 50 nt of a predefined polyA site count as true positives; precision = TP / (TP + FP) should exceed comparators
- Cumulative |ΔPAU| distribution for InPACT lies lowest (smallest per-site usage error) versus IPAFinder and APAIQ
- Each coverage level has five replicate datasets and metrics are averaged across replicates before plotting
- InPACT output contains required columns (Terminal exon, IPA type, Gene, PolyAsite, IPA usage) with PAU estimates in [0,1]

## Limitations

- IPAFinder depends on read coverage fluctuations requiring adequate sequencing depth; its precision of identified IPA sites raises concerns
- APAIQ is not specifically designed for IPA analysis and cannot assemble IPA isoforms or differentiate skipped from composite IPA events
- No version numbers are given for InPACT or any compared tool, and no parameters or thresholds are reported for the simulations or comparative benchmarks in some sections
- Benchmark judged against predefined sites in simulated data; real-data agreement relies on 3P-seq, A-seq, polyA-seq, and long-read Iso-seq ground truth

## Evidence

- [methods] Simulation design: "The RNA-seq data were simulated with varying sequencing coverage levels ranging from 10✕ to 50✕ with a 10✕ increment"
- [abstract] InPACT outperforms comparators: "We have demonstrated that InPACT outperforms other methods in terms of IPA identification and quantification"
- [intro] IPAFinder limitation: "IPAFinder relies on read coverage fluctuations, which necessitates adequate sequencing depth, and the precision of identified IPA sites raises concerns"
- [intro] APAIQ limitation: "APAIQ is not specifically designed for IPA analysis and therefore cannot assemble IPA isoforms and cannot differentiate between skipped and composite IPA events"
- [readme] Quantification workflow: "gffread and salmon is used to index and quantify the transcriptome, and then the usage is calculated."
- [discussion] Ground-truth validation: "InPACT outperforms them in precisely identifying IPA sites, as evidenced by comparison to sites captured by 3P-seq, A-seq, polyA-seq, and long-read Iso-seq"
