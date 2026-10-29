---
name: small-rna-seq-read-classification-summarization
description: Use when processing single-end Illumina sRNA-seq FASTQ reads (14–50 nt)
  from plant samples and you need to classify them into tRF/tRNA-half subtypes and
  reconcile per-species totals against published reference counts.
when_to_use_negative:
- Reads are paired-end or from non-plant species — the toolkit and its bowtie/HAMR
  settings were designed for single-end plant sRNA-seq
- The input is already a classified/annotated tncRNA count table — only the summarization
  step (pandas aggregation into species_tncRNA_summary.csv) is needed, not reclassification
- Reads contain tRNA post-transcriptional modifications that block alignment — the
  paper notes conventional RNA-seq modification biases; tRNA-seq would be required
  instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: tncRNA Toolkit
  role: Core pipeline (tncRNAs.py) that masks tRNA regions into artificial chromosomes,
    aligns reads with bowtie, classifies reads into tRF-5/tRF-3/tRF-1/tRNA halves,
    and emits per-sample count tables
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: pandas
  role: Python module (v1.1.0) used to load per-sample classification outputs and
    aggregate unique-read and distinct-locus counts per species and tncRNA class into
    the summary CSV
- name: tRNAscan-SE (v2.0.6)
  role: Predicts nuclear (eukaryotes model) and organellar ('-O' option) tRNAs; pseudogenes
    and score < 50 tRNAs are eliminated before mapping
- name: NCBI SRA Toolkit (v2.10)
  role: Downloads the single-end Illumina sRNA-seq datasets from SRA that supply the
    input FASTQ files (2,448 samples across six angiosperms)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:12ade4dfae4a4daf3c339b2c7a0a8c10873575fbd2672afa94e2741b3220dbfd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - NCBI SRA Toolkit (v2.10)
  - pandas
  - python3
  - tRNAscan-SE (v2.0.6)
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# small-rna-seq-read-classification-summarization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies quality-filtered small RNA-seq reads into tRNA-derived non-coding RNA (tncRNA) subtypes (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) using the tncRNA Toolkit, then aggregates per-sample classification output into per-species counts of unique reads and loci with pandas.

## When to use

Use when processing single-end Illumina sRNA-seq FASTQ reads (14–50 nt) from plant samples and you need to classify them into tRF/tRNA-half subtypes and reconcile per-species totals against published reference counts.

## When NOT to use

- Reads are paired-end or from non-plant species — the toolkit and its bowtie/HAMR settings were designed for single-end plant sRNA-seq
- The input is already a classified/annotated tncRNA count table — only the summarization step (pandas aggregation into species_tncRNA_summary.csv) is needed, not reclassification
- Reads contain tRNA post-transcriptional modifications that block alignment — the paper notes conventional RNA-seq modification biases; tRNA-seq would be required instead

## Inputs

- Quality-filtered, adapter-trimmed single-end small RNA-seq FASTQ (.fastq/.fq) files per sample
- Genome FASTA with headers beginning '>chr[Num]' ('>chrMt', '>chrPt' for mitochondria/plastid)
- tRNA gene pool from tRNAscan-SE v2.0.6 (pseudogenes removed, score ≥ 50)
- tncRNAs.py per-sample classification output tables (read counts and locus per classified read)

## Outputs

- Per-sample tncRNA classification tables with read count and locus per classified read
- species_tncRNA_summary.csv: total and unique tncRNA counts per species per subtype (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH)
- RPM values (reads mapped to progenitor tRNA / total mapped reads × 10^6) for expression-level aggregation

## How to apply

Run tncRNAs.py per species with a genome FASTA whose headers start with '>chr[Num]' ('>chrMt' and '>chrPt' for organellar) to build the bowtie index, then supply quality-filtered, adapter-trimmed single-end reads via '-i' and an output directory via '-o'. The toolkit masks genuine tRNA gene regions with 50 nt upstream/downstream into artificial chromosomes, aligns reads with bowtie ('-v 0 -m 50 --norc' for multi-mapping suppression; '-v2 --best' for error-tolerant unique alignment), and classifies a read as tncRNA only if it maps exclusively to artificial chromosomes; the default read-count cut-off is 10 and mismatches default to 0. For tRNA inputs, pseudogenes are eliminated and only tRNAs with a tRNAscan-SE score ≥ 50 are used. After running, load the per-sample classification output tables with pandas, group by species and tncRNA class, and sum unique-read counts and distinct locus counts; where expression aggregation is needed, compute RPM = reads mapped to progenitor tRNA / (total mapped reads / 10^6). Write the per-species totals into species_tncRNA_summary.csv and verify class sums reconcile with reported reference values (e.g. A. thaliana 1,676 tncRNAs; O. sativa 243; Z. mays 221, from 2,448 sRNA-seq samples across six angiosperms).

## Related tools

- **tncRNA Toolkit** (Core pipeline (tncRNAs.py) that masks tRNA regions into artificial chromosomes, aligns reads with bowtie, classifies reads into tRF-5/tRF-3/tRF-1/tRNA halves, and emits per-sample count tables) — https://github.com/skbinfo/tncRNA-Toolkit
- **pandas** (Python module (v1.1.0) used to load per-sample classification outputs and aggregate unique-read and distinct-locus counts per species and tncRNA class into the summary CSV)
- **tRNAscan-SE (v2.0.6)** (Predicts nuclear (eukaryotes model) and organellar ('-O' option) tRNAs; pseudogenes and score < 50 tRNAs are eliminated before mapping)
- **NCBI SRA Toolkit (v2.10)** (Downloads the single-end Illumina sRNA-seq datasets from SRA that supply the input FASTQ files (2,448 samples across six angiosperms))

## Examples

```
python3 tncRNAs.py -s Arabidopsis_thaliana -i processed_reads.fq -o outdir
```

## Evaluation signals

- Aggregated per-species totals and per-class sums equal the reference values reported in the article (e.g. A. thaliana 1,676 tncRNAs; O. sativa 243; Z. mays 221) across 2,448 sRNA-seq samples
- Every classified read maps exclusively to artificial chromosomes (tRNA region plus 50 nt leader/trailer); no read maps only to non-tRNA genomic regions
- Bowtie parameters match the paper: -m 50 with --norc -v 0 for multi-mapping discard, and -v2 --best for error-tolerant unique alignments; read-count cut-off defaults to 10
- tRNA pool contains no pseudogenes and no tRNAs with tRNAscan-SE score below 50
- Species summary table schema contains one row per species per tncRNA class with both unique-read count and distinct-locus count columns

## Limitations

- Allowing mismatches to tolerate sequencing errors may lead to base misidentification and inflation of false negatives (default in the toolkit is -v 0, i.e. no mismatch)
- HAMR v1.2 used for modified nucleoside prediction requires python2, and bowtie1/HAMR must be built in the 'util' directory or globally installed
- The toolkit lacks standardized tncRNA nomenclature support and results need experimental validation; conventional RNA-seq carries tRNA modification biases that tRNA-seq would mitigate

## Evidence
