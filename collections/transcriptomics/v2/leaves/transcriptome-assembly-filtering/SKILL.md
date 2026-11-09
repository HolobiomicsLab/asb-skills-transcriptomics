---
name: transcriptome-assembly-filtering
description: Use when you have a de novo assembled transcriptome (e.g. Trinity output)
  plus the original RNA-Seq reads and need to remove low-expression, redundant, and
  chimeric contigs without a reference genome.
when_to_use_negative:
- When a custom order of filtering steps is required — the README states 'Custom order
  of filtering steps is disabled in this version'
- When you need a reference-based transcriptome assessment — the article notes 'When
  there is no reference genome available, the quality of a de novo assembled transcriptome
  is difficult to assess' only in the reference-free context; this skill is reference-free,
  so it is the wrong choice if a curated reference genome/transcriptome exists and
  reference-guided methods are preferred
- When transcripts from alternative splicing of one gene are present and must be preserved
  — Bellerophon may erroneously remove such contigs misclassified as chimeras
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Bellerophon
  role: 'Pipeline that orchestrates the filtering sequence: TPM filter, CD-HIT-EST,
    and TransRate filtering, with quality scoring'
  repo: https://github.com/JesseKerkvliet/Bellerophon
- name: TransRate
  role: Reference-free quality assessment tool; also provides TransRate-C filtering
    of chimeric/low-quality contigs
- name: CD-HIT-EST
  role: Removes redundant/highly identical contigs at a default 95% identity cut-off
- name: BUSCO
  role: Establishes ground quality score of the unfiltered assembly and assesses completeness
    of the filtered transcriptome
- name: Trinity
  role: De novo assembler whose output (contigs with isoforms) is the typical input
    to the filtering pipeline
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_006
  - task_007
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:f9bad2be378780091a6fa5fe0323e08e76057e31f5e9296f13f643cfdf7c6f9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JesseKerkvliet/Bellerophon
  tools:
  - BLASTn
  - BUSCO
  - Bellerophon
  - CD-HIT-EST
  - Polyester
  - TransRate
  - Trinity
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# transome-assembly-filtering

## Summary

Filter a de novo transcriptome assembly with the Bellerophon pipeline, which uses TransRate quality assessment, a TPM filter (cut-off 1), CD-HIT-EST (95% identity), and TransRate-C to remove lowly expressed, redundant, and chimeric contigs. It is specifically aimed at removing falsely assembled chimeras to reduce false-positive gene discovery and false-negative differential expression calls.

## When to use

Use when you have a de novo assembled transcriptome (e.g. Trinity output) plus the original RNA-Seq reads and need to remove low-expression, redundant, and chimeric contigs without a reference genome.

## When NOT to use

- When a custom order of filtering steps is required — the README states 'Custom order of filtering steps is disabled in this version'
- When you need a reference-based transcriptome assessment — the article notes 'When there is no reference genome available, the quality of a de novo assembled transcriptome is difficult to assess' only in the reference-free context; this skill is reference-free, so it is the wrong choice if a curated reference genome/transcriptome exists and reference-guided methods are preferred
- When transcripts from alternative splicing of one gene are present and must be preserved — Bellerophon may erroneously remove such contigs misclassified as chimeras

## Inputs

- De novo transcriptome assembly in FASTA format (e.g. Trinity contigs, Demo.fasta)
- Paired-end RNA-Seq reads in FASTQ format (left R1 and right R2 files, e.g. Illumina reads from NCBI BioProject PRJNA527373)
- Optional: BLASTn/BLASTX annotations (e.g. against NCBI RefSeq GCF_000001215.4 or the NR database) to classify contigs as chimeric or non-chimeric

## Outputs

- Filtered transcriptome assembly in FASTA format with low-expression, redundant, and chimeric contigs removed
- TransRate-Q and BUSCO quality scores for the filtered transcriptome (e.g. per-condition scores like filter_order_scores.csv)
- Per-step removal counts (TPM, CD-HIT-EST, TransRate-C) of chimeric vs non-chimeric contigs

## How to apply

First establish a ground quality score of the unfiltered assembly by running BUSCO (e.g. the Insecta odb9 ortholog set) and TransRate on the assembly plus sequencing reads. Then run Bellerophon in its default order: a TPM filter with a default cut-off score of 1 to remove lowly expressed contigs, followed by CD-HIT-EST at a default 95% identity cut-off to remove redundant contigs, and finally TransRate's filtering capabilities (TransRate-C). The default order (TPM before CD-HIT-EST, filtering order 12 in Figure 2) was chosen because ORF-length filtering removed too many transcripts and because TransRate-C applied before RSEM-based TPM filtering fails to remove chimeric isoforms efficiently. Run the pipeline via the Vagrant VM (recommended to avoid TransRate backwards-compatibility problems) or a conda environment, and write output to the /vagrant/ shared directory. Evaluate success by re-scoring the filtered assembly with TransRate-Q and BUSCO and, if benchmarking, tracking how many known or planted chimeras survive each filter step.

## Related tools

- **Bellerophon** (Pipeline that orchestrates the filtering sequence: TPM filter, CD-HIT-EST, and TransRate filtering, with quality scoring) — https://github.com/JesseKerkvliet/Bellerophon
- **TransRate** (Reference-free quality assessment tool; also provides TransRate-C filtering of chimeric/low-quality contigs)
- **CD-HIT-EST** (Removes redundant/highly identical contigs at a default 95% identity cut-off)
- **BUSCO** (Establishes ground quality score of the unfiltered assembly and assesses completeness of the filtered transcriptome)
- **Trinity** (De novo assembler whose output (contigs with isoforms) is the typical input to the filtering pipeline)

## Examples

```
python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun
```

## Evaluation signals

- TransRate score of the filtered assembly improves over the unfiltered ground score (e.g. H. subflexa pheromone gland transcriptome improved from 0.247 to 0.312; CD-HIT-EST-only filter reached 0.319)
- In the 500 synthetic-chimera benchmark, ~97% of planted chimeras (485 ± 3.06 of 500) are removed versus ~70% of non-chimeric sequences, with the TPM step alone discarding ~95.7% of chimeras
- In the simulated D. melanogaster benchmark, the pipeline removes 136 of 295 chimeras (46.1%) and 575 of 3,283 correct sequences (17.5%) — more chimeras than correct contigs proportionally
- Per-step chimera removal breakdown is reproducible: TPM first removed 227 (8.5%), CD-HIT-EST removed 600 (24.5%), and TransRate-C removed 792 (42.9%) chimeras
- The combined CD-HIT + TPM + ORF-length filter leaves roughly 16,057 contigs, matching the reported second-best TransRate score of 0.316

## Limitations

- Custom order of filtering steps is disabled in the current version of the pipeline
- Bellerophon may erroneously remove contigs from alternative splicing that TransRate misclassifies as chimeras when expression is uneven along the sequence
- No tool version numbers are stated for BUSCO, TransRate, CD-HIT-EST, Trinity, or Polyester; TransRate has backwards-compatibility problems, so the Vagrant VM installation is recommended
- ORF-length filtering appeared to remove too many transcripts and is not part of the default filter order
- The Vagrant VM disk size is too small for most full transcriptome analyses; output must be written to the shared /vagrant/ directory

## Evidence

- [abstract] Bellerophon first uses the quality assessment tool TransRate to indicate the quality, after which it uses a transcripts per million (TPM) filter to remove lowly expressed contigs and CD-HIT-EST to remove highly identical contigs.: "Bellerophon first uses the quality assessment tool TransRate to indicate the quality, after which it uses a transcripts per million (TPM) filter to remove lowly expressed contigs and CD-HIT-EST to"
- [methods] (b) Bellerophon filters the assembly using a TPM filter with a default cut-off score of 1.: [evidence span withheld — anchor status: not_found]
- [results] we set the default order to be (1) TPM filtering and (2) CD‐HIT‐EST (filtering order 12 in Figure 2), as the ORF‐length filtering appeared to be removing too much transcripts.: "we set the default order to be (1) TPM filtering and (2) CD‐HIT‐EST (filtering order 12 in Figure 2), as the ORF‐length filtering appeared to be removing too much transcripts."
- [readme] python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun: "python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun"
- [discussion] Contigs representative of transcripts arising from alternative splicing of one gene might be considered as chimera by Transrate if they are unevenly expressed along their sequence.: "Contigs representative of transcripts arising from alternative splicing of one gene might be considered as chimera by Transrate if they are unevenly expressed along their sequence."
