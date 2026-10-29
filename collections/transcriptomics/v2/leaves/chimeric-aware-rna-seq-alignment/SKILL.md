---
name: chimeric-aware-rna-seq-alignment
description: Use when RNA-seq FASTQ reads from a tumor sample must be aligned for
  fusion detection (Arriba) or for RNA-side variant callers (transIndel, ScanExitron)
  and chimeric events such as gene fusions must not be missed.
when_to_use_negative:
- Input is DNA-seq (WES/WGS) — ScanNeo2 aligns DNA-seq read groups directly with BWA-MEM
  at mapq 30, not with chimeric-aware STAR.
- Only fusion calling is needed and the STAR parameter --alignIntronMax would have
  to be reduced — Arriba explicitly avoids this by working on chimeric-aware STAR
  alignments.
- Reads are long reads (Nanopore/PacBio) — the short-read STAR workflow described
  here is not designed for them.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STAR
  role: Chimeric-aware splice-aware aligner run on RNA-seq read groups with the ScanNeo2
    chim* parameter set; also used to realign reads before Arriba fusion detection
  repo: https://github.com/alexdobin/STAR
- name: ScanNeo2
  role: Snakemake workflow that orchestrates the chimeric-aware STAR alignment and
    the 'realign' rule producing BWA-style CIGARs for transIndel/ScanExitron
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: BWA
  role: Provides the BWA-style CIGAR alignment required by the downstream RNA variant
    callers via the 'realign' rule on the STAR BAM
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: Arriba
  role: Downstream consumer of the chimeric-aware STAR alignments for gene fusion
    detection from RNA-seq
  repo: https://github.com/suhrig/arriba
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:ac9fefd7bdfd3a014b7d056b3e343d9bf67b1845b516db078e33155638f459e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexdobin/STAR
  tools:
  - Arriba
  - BWA
  - STAR
  - ScanNeo2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# chimeric-aware-rna-seq-alignment

## Summary

Align RNA-seq read groups against the reference genome with STAR run in chimeric-aware mode (chimSegmentMin 20, chimScoreMin 10, chimJunctionOverhangMin 10, chimScoreDropMax 30, chimScoreSeparation 10, matching STAR v2.7.10b) so that split/contiguous alignments spanning gene fusions and structural rearrangements are captured for downstream neoantigen discovery in ScanNeo2.

## When to use

Use when RNA-seq FASTQ reads from a tumor sample must be aligned for fusion detection (Arriba) or for RNA-side variant callers (transIndel, ScanExitron) and chimeric events such as gene fusions must not be missed.

## When NOT to use

- Input is DNA-seq (WES/WGS) — ScanNeo2 aligns DNA-seq read groups directly with BWA-MEM at mapq 30, not with chimeric-aware STAR.
- Only fusion calling is needed and the STAR parameter --alignIntronMax would have to be reduced — Arriba explicitly avoids this by working on chimeric-aware STAR alignments.
- Reads are long reads (Nanopore/PacBio) — the short-read STAR workflow described here is not designed for them.

## Inputs

- Preprocessed RNA-seq reads (FASTQ, output of fastp trimming with quality filtering, adapter removal, and minlen/sliding-window options)
- Reference genome indexed for STAR (ENSEMBL release genome)
- Sample sheet with (sample, seqtype, group) rows

## Outputs

- Chimeric-aware STAR BAM (sorted) with STAR SJ.out / chimeric junction records feeding Arriba fusion calling
- Realigned BWA-style BAM retained for transIndel and ScanExitron
- .bai index files in results/<sample>/rnaseq/align/

## How to apply

Launch the Snakemake workflow so that independent per-group alignment branches fan out across cores/nodes for every (sample, seqtype, group) row in the sample sheet. Align each RNA-seq read group with STAR in chimeric-aware mode using the ScanNeo2 parameter set: chimSegmentMin 20, chimScoreMin 10, chimJunctionOverhangMin 10, chimScoreDropMax 30, chimScoreSeparation 10 (identical to STAR v2.7.10b). Because the downstream RNA variant callers transIndel and ScanExitron require a BWA-style CIGAR, realign the STAR-produced BAM with BWA via the 'realign' rule, keeping both intermediates and the final BAM. Finally, write .bai index files alongside every aligned BAM in results/<sample>/rnaseq/align/ via the postproc_bam_index rule. This chimeric-aware STAR alignment is separate from, and complements, the BWA alignment of RNA-seq reads used for SNV/indel calling.

## Related tools

- **STAR** (Chimeric-aware splice-aware aligner run on RNA-seq read groups with the ScanNeo2 chim* parameter set; also used to realign reads before Arriba fusion detection) — https://github.com/alexdobin/STAR
- **ScanNeo2** (Snakemake workflow that orchestrates the chimeric-aware STAR alignment and the 'realign' rule producing BWA-style CIGARs for transIndel/ScanExitron) — https://github.com/ylab-hi/ScanNeo2/
- **BWA** (Provides the BWA-style CIGAR alignment required by the downstream RNA variant callers via the 'realign' rule on the STAR BAM) — https://github.com/ylab-hi/ScanNeo2/
- **Arriba** (Downstream consumer of the chimeric-aware STAR alignments for gene fusion detection from RNA-seq) — https://github.com/suhrig/arriba

## Examples

```
snakemake --use-conda --cores 8 results/sample1/rnaseq/align/sample1.bam results/sample1/rnaseq/align/sample1.bam.bai
```

## Evaluation signals

- results/<sample>/rnaseq/align/ contains the aligned BAM plus a matching .bai index from the postproc_bam_index rule
- The chim* parameters in the executed Snakemake config match chimSegmentMin 20, chimScoreMin 10, chimJunctionOverhangMin 10, chimScoreDropMax 30, chimScoreSeparation 10 (STAR v2.7.10b defaults used by ScanNeo2)
- Both the STAR BAM and its BWA-realigned counterpart ('realign' rule intermediates) are retained for every RNA-seq group
- STAR junction/chimeric output exists and is consumable by Arriba (post-alignment runtime typically ~2 minutes per the Arriba README)
- Downstream callers transIndel and ScanExitron accept the realigned BAM without CIGAR-format errors, confirming BWA-style CIGARs are present

## Limitations

- STAR-produced BAMs carry non-BWA-style CIGARs and cannot be fed directly to transIndel or ScanExitron; the extra 'realign' rule is mandatory for those callers.
- Snakemake version is not pinned in the paper text; the repository README badge indicates Snakemake >= 9.0.0, so older workflows may not run unchanged.
- The chimeric parameters are tied to STAR v2.7.10b behavior; other STAR versions may produce different chimeric junctions for the same data.

## Evidence

- [other] Align each RNA-seq read group with STAR in chimeric-aware mode using chimSegmentMin: 20, chimScoreMin: 10, chimJunctionOverhangMin: 10, chimScoreDropMax: 30: [evidence span withheld — anchor status: not_found]
- [results] the aligned reads are first realigned using STAR (Dobin et al. 2013), and then subjected to Arriba: "the aligned reads are first realigned using STAR (Dobin et al. 2013), and then subjected to Arriba"
- [results] the RNA-seq data are aligned against the reference genome using BWA (Li and Durbin 2009): "the RNA-seq data are aligned against the reference genome using BWA (Li and Durbin 2009)"
- [results] subsequently filtered for mapping quality, duplicates, and discarded of unmapped reads: "subsequently filtered for mapping quality, duplicates, and discarded of unmapped reads"
- [other] since the downstream RNA variant callers (transIndel, ScanExitron) require a BWA-style CIGAR: [evidence span withheld — anchor status: not_found]
