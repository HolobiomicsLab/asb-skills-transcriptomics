---
name: chimera-removal-from-de-novo-transcriptomes
description: Use when a de novo transcriptome assembly (e.g., Trinity-built from Illumina
  RNA-Seq reads) is suspected of containing chimeric or redundant contigs and no reference
  genome is available to assess assembly quality.
when_to_use_negative:
- A reference genome is available for assessing assembly quality — TransRate is reference-free,
  designed for cases where 'there is no reference genome available to assess de novo
  assembly quality'.
- Custom ordering of the filtering steps is required — 'Custom order of filtering
  steps is disabled in this version'.
- The input is already a curated/filtered transcriptome rather than a raw de novo
  assembly — re-applying the filters may remove alternative-splicing contigs misclassified
  as chimeras.
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Bellerophon
  role: Pipeline that orchestrates the TPM, CD-HIT-EST, and TransRate filtering steps
    and generates the results report
  repo: github.com/JesseKerkvliet/Bellerophon
- name: TransRate
  role: Reference-free quality assessment (TransRate-Q) and final chimera filtering
    (TransRate-C), requiring only the reads and the assembled transcriptome
- name: CD-HIT-EST
  role: Clusters and removes highly identical/redundant contigs at the default 95%
    identity cut-off
- name: BUSCO
  role: Establishes the ground quality score of the unfiltered assembly and re-assesses
    completeness of the fully filtered assembly
- name: Trinity
  role: De novo assembler that produced the source transcriptome (H. subflexa pheromone
    gland assembly per Appendix S1)
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:76b545ceeac342c96d12aa50d653474d83c3fa046204f7399a91c2eb9af70c20
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BUSCO
  - Bellerophon
  - CD-HIT-EST
  - TransRate
  - Trinity
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# chimera-removal-from-de-novo-transcriptomes

## Summary

A filtering pipeline (Bellerophon) that improves de novo transcriptome assemblies by removing chimeric, lowly expressed, and redundant contigs using TransRate quality assessment, a TPM filter (cut-off 1), CD-HIT-EST (95% identity), and TransRate-C, removing between 40% and 91.9% of chimeras.

## When to use

Use when a de novo transcriptome assembly (e.g., Trinity-built from Illumina RNA-Seq reads) is suspected of containing chimeric or redundant contigs and no reference genome is available to assess assembly quality.

## When NOT to use

- A reference genome is available for assessing assembly quality — TransRate is reference-free, designed for cases where 'there is no reference genome available to assess de novo assembly quality'.
- Custom ordering of the filtering steps is required — 'Custom order of filtering steps is disabled in this version'.
- The input is already a curated/filtered transcriptome rather than a raw de novo assembly — re-applying the filters may remove alternative-splicing contigs misclassified as chimeras.

## Inputs

- De novo transcriptome assembly in FASTA format (e.g., Trinity-based assembly)
- Paired Illumina RNA-Seq reads (e.g., --left All_R1_1000.fastq --right All_R2_1000.fastq)
- BUSCO ortholog set for quality assessment (e.g., Insecta odb9)

## Outputs

- Fully filtered transcriptome assembly with lowly expressed, redundant, and chimeric-suspect contigs removed
- Bellerophon results report (bellerophon_report.txt) with TransRate scores and BUSCO metrics before and after filtering
- Per-filter chimera-removal statistics (e.g., TPM removed 227 (8.5%) of chimeras; CD-HIT-EST removed 600 (24.5%); TransRate-C removed 792 (42.9%))

## How to apply

Run TransRate (with BUSCO, e.g., the Insecta odb9 ortholog set) on the unfiltered assembly using only the sequencing reads and transcriptome to establish a ground quality score. Apply Bellerophon's TPM filter with the default cut-off score of 1 to remove lowly expressed contigs, then reassess with TransRate-Q. Next cluster and remove redundant contigs with CD-HIT-EST using the default 95% identity cut-off (more lenient than the tool's own 0.9 default). Execute TransRate's filtering capabilities (TransRate-C) last — this final step removed the largest share of chimeras (42.9%) — then run TransRate-Q and BUSCO on the fully filtered assembly and let Bellerophon automatically generate the results report (bellerophon_report.txt). The default filtering order is (1) TPM filtering and (2) CD-HIT-EST, because ORF-length filtering was found to remove too many transcripts.

## Related tools

- **Bellerophon** (Pipeline that orchestrates the TPM, CD-HIT-EST, and TransRate filtering steps and generates the results report) — github.com/JesseKerkvliet/Bellerophon
- **TransRate** (Reference-free quality assessment (TransRate-Q) and final chimera filtering (TransRate-C), requiring only the reads and the assembled transcriptome)
- **CD-HIT-EST** (Clusters and removes highly identical/redundant contigs at the default 95% identity cut-off)
- **BUSCO** (Establishes the ground quality score of the unfiltered assembly and re-assesses completeness of the fully filtered assembly)
- **Trinity** (De novo assembler that produced the source transcriptome (H. subflexa pheromone gland assembly per Appendix S1))

## Examples

```
python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun
```

## Evaluation signals

- The report file bellerophon_report.txt exists and confirms all steps completed in the default order: (1) TPM filtering, (2) CD-HIT-EST, with TransRate-C applied last.
- On the H. subflexa pheromone gland benchmark, TransRate score improves from 0.247 to 0.312 and duplicated BUSCO groups drop from 562 to 182 (22.9% of total groups), within expected run-to-run variation.
- Removed-contig counts match reported values (6,838 low-evidence and 7,240 chimeric-suspect contigs removed for H. subflexa).
- The pipeline removed more chimeric than nonchimeric contigs, and between 40% and 91.9% of chimeras overall.
- CD-HIT-EST used the more lenient 95% identity cut-off rather than the tool's own 0.9 default.

## Limitations

- Contigs representative of transcripts arising from alternative splicing of one gene might be considered chimeras by TransRate if they are unevenly expressed along their sequence, so Bellerophon may erroneously remove such contigs.
- Custom order of filtering steps is disabled in this version of the pipeline.
- Backwards compatibility problems with TransRate are avoided by using the Vagrant virtual machine; local installation requires Conda (envs/Bellerophon.yml).
- The VM disk size is not large enough for most full transcriptome analyses; outputs should be written to the shared /vagrant/ directory.
- No tool version numbers are stated for BUSCO, TransRate, CD-HIT-EST, Trinity, or Polyester, complicating exact reproduction.

## Evidence

- [abstract] it uses a transcripts per million (TPM) filter to remove lowly expressed contigs and CD-HIT-EST to remove highly identical contigs: "after which it uses a transcripts per million (TPM) filter to remove lowly expressed contigs and CD-HIT-EST to remove highly identical contigs."
- [results] default Bellerophon filtering order: [evidence span withheld — anchor status: not_found]
- [results] The final Transrate-C filtering step removed the largest share of chimeras: [evidence span withheld — anchor status: not_found]
- [methods] Bellerophon then uses CDHIT-EST to filter redundant contigs: [evidence span withheld — anchor status: not_found]
- [readme] Test-run invocation from README: "python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun"
- [readme] Custom order of filtering steps is disabled in this version: "Custom order of filtering steps is disabled in this version"
