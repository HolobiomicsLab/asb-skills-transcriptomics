---
name: read-abundance-correlation-analysis
description: Use when comparing per-cell pathogen read quantification across multiple
  scRNAseq tools (e.g. SPQ vs Venus/PathogenTrack/ViralTrack) on the same 10x sample
  to assess concordance of read-abundance estimates.
when_to_use_negative:
- Only cell-level infection calls (binary HIV+/HIV-) are of interest and no read-count
  magnitudes are compared — the four-tool overlap analysis is the relevant companion
  check instead
- The 10x sample has not been processed with cellranger count, so no host-aligned
  BAM/unaligned reads exist for SPQ to extract pathogen reads from
- Quantifying multiple distinct viruses in one sample rather than benchmarking against
  comparator tools — SPQ itself handles this natively without a benchmarking correlation
  step
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0781
tools:
- name: scPathoQuant
  role: Produces the reference UMI-based per-cell pathogen read counts (pathogen_al_counts.csv)
    using the default bbmap aligner
  repo: https://github.com/galelab/scPathoQuant
- name: ViralTrack
  role: Comparator tool run independently on the same sample to obtain per-cell HIV+
    calls and read counts for correlation
  repo: https://github.com/PierreBSC/Viral-Track
- name: Venus
  role: Comparator tool whose counts are expected to run higher than UMI-based SPQ
    counts, producing the characteristic upward offset in Fig. 5B-D
  repo: https://github.com/aicb-ZhangLabs/Venus
- name: PathogenTrack
  role: Comparator tool run independently on the same donor 2 sample for read-abundance
    correlation against SPQ
  repo: https://github.com/ncrna/PathogenTrack
- name: cellranger
  role: Generates the 10x output folder that SPQ takes as input; SPQ integrates pathogen
    counts back into these files
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:4ea69b4d6a63d19b4e26597282fc58aef10f8f33b563cee6a1692967988c2ad8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - PathogenTrack
  - Venus
  - ViralTrack
  - bbmap
  - cellranger
  - scPathoQuant
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# read-abundance-correlation-analysis

## Summary

Benchmark scPathoQuant per-cell pathogen read counts against comparator tools (ViralTrack, PathogenTrack, Venus) by correlating read abundance and checking expected count offsets, as done for HIV-infected donor 2 cells in Fig. 5B-D.

## When to use

Use when comparing per-cell pathogen read quantification across multiple scRNAseq tools (e.g. SPQ vs Venus/PathogenTrack/ViralTrack) on the same 10x sample to assess concordance of read-abundance estimates.

## When NOT to use

- Only cell-level infection calls (binary HIV+/HIV-) are of interest and no read-count magnitudes are compared — the four-tool overlap analysis is the relevant companion check instead
- The 10x sample has not been processed with cellranger count, so no host-aligned BAM/unaligned reads exist for SPQ to extract pathogen reads from
- Quantifying multiple distinct viruses in one sample rather than benchmarking against comparator tools — SPQ itself handles this natively without a benchmarking correlation step

## Inputs

- cellranger count output folder for the 10x sample (e.g. HIV-infected donor 2)
- simplified pathogen reference genome fasta (e.g. header '>HIV_virus') from galelab/Whitmore_scPathoQuant_testSets
- per-cell pathogen read counts from each comparator tool (ViralTrack, PathogenTrack, Venus)

## Outputs

- HIV read-abundance correlation values between SPQ counts and each comparator tool (Fig. 5B-D)
- expected offset assessment showing Venus counts run higher than UMI-based SPQ counts
- companion four-tool overlap table of HIV+ cell calls (672 cells, 81.3% overlap, Fig. 5A)

## How to apply

Run scPathoQuant with the default bbmap aligner on the cellranger count output for the sample (e.g. HIV-infected donor 2), producing per-cell pathogen read counts in pathogen_al_counts.csv. Run each comparator tool (ViralTrack, PathogenTrack, Venus) independently on the same sample to obtain its per-cell pathogen+ calls and read counts. Compute correlations between SPQ counts and each comparator tool's counts, as in Fig. 5B-D. Note the expected systematic offset: Venus counts run higher than the UMI-based SPQ counts, so the SPQ-vs-Venus correlation should show an upward offset rather than exact equality. Concordance of counts (not just cell calls) is the target; four-tool overlap of HIV+ cell calls (672 cells, 81.3% on donor 2) is the companion check from Fig. 5A.

## Related tools

- **scPathoQuant** (Produces the reference UMI-based per-cell pathogen read counts (pathogen_al_counts.csv) using the default bbmap aligner) — https://github.com/galelab/scPathoQuant
- **ViralTrack** (Comparator tool run independently on the same sample to obtain per-cell HIV+ calls and read counts for correlation) — https://github.com/PierreBSC/Viral-Track
- **Venus** (Comparator tool whose counts are expected to run higher than UMI-based SPQ counts, producing the characteristic upward offset in Fig. 5B-D) — https://github.com/aicb-ZhangLabs/Venus
- **PathogenTrack** (Comparator tool run independently on the same donor 2 sample for read-abundance correlation against SPQ) — https://github.com/ncrna/PathogenTrack
- **cellranger** (Generates the 10x output folder that SPQ takes as input; SPQ integrates pathogen counts back into these files)

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/pathogen/fastafilefolder
```

## Evaluation signals

- Four-tool overlap table reproduces the reported 672 cells HIV+ in all 4 methods, i.e. 81.3% four-tool overlap on the donor 2 sample (Fig. 5A)
- SPQ-vs-Venus read-abundance correlation shows the expected upward offset, with Venus counts running higher than the UMI-based SPQ counts
- Correlation values across SPQ-vs-ViralTrack, SPQ-vs-PathogenTrack, and SPQ-vs-Venus are consistent with the Fig. 5B-D panel results
- pathogen_al_counts.csv exists and contains non-zero per-cell counts for the expected HIV+ cell population before computing correlations

## Limitations

- Venus and SPQ use different quantification bases (Venus counts run higher than UMI-based SPQ counts), so correlations reflect concordance with a systematic offset, not one-to-one agreement
- The 81.3% four-tool overlap leaves ~18.7% of cells discordantly called across tools, so count correlations should be interpreted alongside cell-call overlap
- Temporary _tmp/ files from SPQ can be large; use --tmp_removal if space is an issue

## Evidence

- [results] SPQ, ViralTrack, PathogenTrack, and Venus show consistent results on the HIV-infected donor 2 sample, with an 81.3% overlap between all 4 tools for identifying HIV-infected cells: [evidence span withheld — anchor status: not_found]
- [results] Venus counts run higher than the UMI-based SPQ counts: [evidence span withheld — anchor status: not_found]
- [readme] Run scPathoQuant with default bbmap aligner producing pathogen_al_counts.csv: "pathogen_al_counts.csv - total number of reads mapping to the pathogen in each cell"
- [readme] Benchmarking codes for ViralTrack, Venus and PathogenTrack provided in test sets repository: [evidence span withheld — anchor status: external_doc]
- [readme] Simplified fasta header recommended for pathogen total read quantification: "it is recommended that if the fasta header is a complicated name it be simplified (i.e. > HIV_virus)"
