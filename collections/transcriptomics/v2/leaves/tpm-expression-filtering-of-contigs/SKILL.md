---
name: tpm-expression-filtering-of-contigs
description: Use when a de novo transcriptome assembly (e.g., Trinity FASTA plus RNA-Seq
  reads) contains lowly expressed contigs that should be removed prior to CD-HIT-EST
  redundancy filtering, following Bellerophon's default TPM cut-off of 1.
when_to_use_negative:
- When the aim is redundancy removal alone — CD-HIT-EST only achieved the highest
  TransRate score (0.319) in the filter-order comparison, so an expression filter
  may be unnecessary if only highly identical contigs are the concern
- When a custom filtering order is required — the README states custom ordering of
  filtering steps is disabled in this version of Bellerophon
- When contigs are suspected to represent alternative splicing isoforms of one gene,
  since unevenly expressed isoform contigs may be wrongly discarded (the article warns
  these can be misclassified as chimeras by TransRate)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: Bellerophon
  role: Pipeline that executes the TPM filter (default cut-off 1) as step (b), ahead
    of CD-HIT-EST and TransRate-C, and generates the results report
  repo: github.com/JesseKerkvliet/Bellerophon
- name: TransRate
  role: Provides quality assessment (TransRate/TransRate-Q) before and after the TPM
    filtering step; its final TransRate-C stage follows later in the pipeline
- name: CD-HIT-EST
  role: Subsequent redundancy filter (95% identity) applied after TPM filtering removes
    lowly expressed contigs
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:5bb0f76826289caeb370666fe41f81e5be0442abefcfdce6cc9220f7d2069fdf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bellerophon
  - CD-HIT-EST
  - TransRate
  - Trinity
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# tpm-expression-filtering-of-contigs

## Summary

Expression-based filtering step of the Bellerophon pipeline that removes lowly expressed contigs from a de novo transcriptome assembly using a transcripts-per-million (TPM) filter with the community default cut-off of 1.0, applied before CD-HIT-EST redundancy removal.

## When to use

Use when a de novo transcriptome assembly (e.g., Trinity FASTA plus RNA-Seq reads) contains lowly expressed contigs that should be removed prior to CD-HIT-EST redundancy filtering, following Bellerophon's default TPM cut-off of 1.

## When NOT to use

- When the aim is redundancy removal alone — CD-HIT-EST only achieved the highest TransRate score (0.319) in the filter-order comparison, so an expression filter may be unnecessary if only highly identical contigs are the concern
- When a custom filtering order is required — the README states custom ordering of filtering steps is disabled in this version of Bellerophon
- When contigs are suspected to represent alternative splicing isoforms of one gene, since unevenly expressed isoform contigs may be wrongly discarded (the article warns these can be misclassified as chimeras by TransRate)

## Inputs

- De novo assembled transcriptome FASTA (e.g., Trinity assembly of H. subflexa pheromone gland RNA-Seq)
- Paired-end Illumina RNA-Seq FASTQ reads (e.g., PRJNA527373: SRR8735410, SRR8735411, SRR8735412)

## Outputs

- TPM-filtered transcriptome FASTA (lowly expressed contigs removed)
- Intermediate quality re-assessment via TransRate-Q on the TPM-filtered assembly
- Bellerophon run report (bellerophon_report.txt) recording the filtering steps and removed-contig counts

## How to apply

After establishing a ground quality score of the unfiltered assembly with TransRate and BUSCO (e.g., Insecta odb9), quantify expression of the assembly contigs against the paired FASTQ reads and discard contigs below the TPM cut-off of 1.0. Apply this step FIRST in the default Bellerophon order — TPM filtering followed by CD-HIT-EST (95% identity) and final TransRate-C filtering — because the article set this order (filtering order 12 in Figure 2) after ORF-length filtering appeared to remove too many transcripts. In the H. subflexa pheromone gland benchmark, the TPM step removed 227 (8.5%) of chimeric contigs before CD-HIT-EST and TransRate-C handled the remainder. Reassess the TPM-filtered assembly with TransRate-Q before proceeding to CD-HIT-EST, and let Bellerophon write the run report (bellerophon_report.txt). The filtered assembly is expected to show improved quality scores, e.g., TransRate 0.247 → 0.312 for the H. subflexa assembly.

## Related tools

- **Bellerophon** (Pipeline that executes the TPM filter (default cut-off 1) as step (b), ahead of CD-HIT-EST and TransRate-C, and generates the results report) — github.com/JesseKerkvliet/Bellerophon
- **TransRate** (Provides quality assessment (TransRate/TransRate-Q) before and after the TPM filtering step; its final TransRate-C stage follows later in the pipeline)
- **CD-HIT-EST** (Subsequent redundancy filter (95% identity) applied after TPM filtering removes lowly expressed contigs)

## Examples

```
python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun
```

## Evaluation signals

- Bellerophon report confirms TPM filtering was executed first in the default order: (1) TPM filtering and (2) CD-HIT-EST (filtering order 12 in Figure 2), followed by TransRate-C
- The TPM filter used the default cut-off score of 1.0 (community default), not the pipeline's alternative ORF-length (50 aa / 100 aa) or CD-HIT-EST (0.95) parameters
- Removed-contig and chimera counts are plausible: in the article's benchmark, TPM filtering removed 227 (8.5%) of chimeras as the first filtering step
- Downstream quality metrics improve after the full filter chain, e.g., TransRate score 0.247 → 0.312 for the H. subflexa pheromone gland assembly
- The output file bellerophon_report.txt exists and the filtered FASTA is written to the --outdir location

## Limitations

- Custom ordering of filtering steps (e.g., running CD-HIT-EST before TPM) is disabled in the current Bellerophon version
- TPM filtering alone only removed 8.5% of chimeras; the bulk of chimera removal (42.9%) required the final TransRate-C step, so TPM filtering is not sufficient for chimera removal on its own
- TransRate may misclassify alternative-splicing isoforms (unevenly expressed along their sequence) as chimeras, and expression-based filtering can compound this bias
- No tool version numbers are stated for BUSCO, TransRate, CD-HIT-EST, Trinity, or Polyester, complicating exact reproduction

## Evidence

- [methods] Bellerophon filters the assembly using a TPM filter with a default cut-off score of 1.: [evidence span withheld — anchor status: not_found]
- [abstract] it uses a transcripts per million (TPM) filter to remove lowly expressed contigs: "it uses a transcripts per million (TPM) filter to remove lowly expressed contigs and CD‐HIT‐EST to remove highly identical contigs."
- [results] the TPM filtering step first removed 227 (8.5%) of the chimeras: "the TPM filtering step first removed 227 (8.5%) of the chimeras"
- [results] we set the default order to be (1) TPM filtering and (2) CD‐HIT‐EST (filtering order 12 in Figure 2): "we set the default order to be (1) TPM filtering and (2) CD‐HIT‐EST (filtering order 12 in Figure 2), as the ORF‐length filtering appeared to be removing too much transcripts."
- [methods] Default thresholds were determined using community defaults for TPM (1.0): "Default thresholds were determined using community defaults for TPM (1.0) and ORF length (50 aa or 100 aa)."
