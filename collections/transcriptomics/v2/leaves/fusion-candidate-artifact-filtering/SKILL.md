---
name: fusion-candidate-artifact-filtering
description: Use when screening raw STAR chimeric alignments from single-cell Smart-Seq
  data to remove artifact-driven fusion candidates and retain true fusions such as
  TCR V(D)J recombinations or recurrent IgH-WHSC1.
when_to_use_negative:
- Input is bulk RNA-seq rather than Smart-Seq scRNA-seq data — scFusion is explicitly
  for single-cell Smart-Seq data rather than 10X or bulk
- The mappability file is missing and the analysis requires the mappability filter
  — scFusion will silently turn the filter off, weakening artifact suppression
- Detecting very rare fusions in highly heterogeneous tumor samples or lowly expressed
  fusions — detection power is limited and can only be improved by sequencing more/deeper
  cells
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scFusion
  role: Pipeline implementing the statistical model, deep-learning model, mappability
    filter, and ad hoc artifact filters on single-cell fusion candidates
  repo: https://github.com/ZijieJin/scFusion
- name: STAR
  role: Aligner (>= 2.7.2d) producing the chimeric-read alignments from which candidate
    fusions are extracted and filtered
  repo: https://github.com/alexdobin/STAR
- name: samtools
  role: BAM processing of single-cell alignments (>= 1.0, tested on 1.10) prior to
    candidate filtering
- name: bedtools
  role: Genomic interval operations (tested on 2.29.2) supporting the mappability
    filter
- name: pyensembl
  role: Python interface to Ensembl gene/transcript annotation used when annotating
    retained fusion candidates
  repo: https://github.com/openvax/pyensembl
- name: pysam
  role: Python BAM parsing module used in the filtering pipeline (tested on version
    0.18.0)
- name: tensorflow/keras
  role: Deep-learning model scoring candidate fusions; turning it off reduces precision
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:a5ca124d8755bc987255b7833a34132cd0bc1966c9270dbcd6e6ac9b30c81b32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZijieJin/scFusion
  tools:
  - Python
  - R
  - STAR
  - bedtools
  - numpy
  - pyensembl
  - pysam
  - samtools
  - scFusion
  - scipy
  - stringr
  - tensorflow/keras
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5238095238095238
n_claims: 21
---

# fusion-candidate-artifact-filtering

## Summary

A filtering stage of the scFusion pipeline that suppresses amplification-artifact chimeric reads in single-cell fusion calls using a mappability filter, a statistical model, a deep-learning (tensorflow/keras) model, and ad hoc filters. It is essential because the heavy Smart-Seq amplification step generates artificial chimeric reads that appear as false-positive fusion candidates.

## When to use

Use when screening raw STAR chimeric alignments from single-cell Smart-Seq data to remove artifact-driven fusion candidates and retain true fusions such as TCR V(D)J recombinations or recurrent IgH-WHSC1.

## When NOT to use

- Input is bulk RNA-seq rather than Smart-Seq scRNA-seq data — scFusion is explicitly for single-cell Smart-Seq data rather than 10X or bulk
- The mappability file is missing and the analysis requires the mappability filter — scFusion will silently turn the filter off, weakening artifact suppression
- Detecting very rare fusions in highly heterogeneous tumor samples or lowly expressed fusions — detection power is limited and can only be improved by sequencing more/deeper cells

## Inputs

- STAR chimeric alignments (BAM) from single-cell Smart-Seq FASTQ pairs (*_1.fastq / *_2.fastq, not *_1.fq)
- hg19mappability 4-column (chr, start, end, value) file placed unzipped in the data folder
- Reference genome FASTA (e.g. hg19.fa, ~3G) and GTF annotation from Ensembl/NCBI/UCSC

## Outputs

- Filtered list of high-confidence single-cell gene fusion calls with per-fusion supporting cell counts
- Fusion candidates rejected by the statistical model, deep-learning model, mappability filter, or ad hoc filters

## How to apply

After scFusion.py aligns each cell's *_1.fastq/*_2.fastq pairs with STAR (>= 2.7.2d) and processes alignments with samtools (>= 1.0) and bedtools (2.29.2), apply the mappability filter using the hg19mappability file in 4-column chr/start/end/value format; scFusion turns this filter off if the file is not supplied. Then filter candidate fusions through scFusion's statistical model and deep-learning model (tensorflow, keras, numpy, with optional scipy), using pyensembl and pysam for annotation handling. Finally apply ad hoc filters that suppress amplification-artifact chimeric reads. The multi-cell design is the rationale: multiple cells from a sample should contain the same fusion, so candidates supported across cells (e.g. TRAJ33-TRAV1-2 in 126 cells) survive while single-cell artifacts are discarded.

## Related tools

- **scFusion** (Pipeline implementing the statistical model, deep-learning model, mappability filter, and ad hoc artifact filters on single-cell fusion candidates) — https://github.com/ZijieJin/scFusion
- **STAR** (Aligner (>= 2.7.2d) producing the chimeric-read alignments from which candidate fusions are extracted and filtered) — https://github.com/alexdobin/STAR
- **samtools** (BAM processing of single-cell alignments (>= 1.0, tested on 1.10) prior to candidate filtering)
- **bedtools** (Genomic interval operations (tested on 2.29.2) supporting the mappability filter)
- **pyensembl** (Python interface to Ensembl gene/transcript annotation used when annotating retained fusion candidates) — https://github.com/openvax/pyensembl
- **pysam** (Python BAM parsing module used in the filtering pipeline (tested on version 0.18.0))
- **tensorflow/keras** (Deep-learning model scoring candidate fusions; turning it off reduces precision)

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- The T cell dataset run retains 8 TCR V(D)J recombinations, including TRAJ33-TRAV1-2 supported by 126 cells and TRAJ12-TRAV1-2 by 20 cells
- The multi-cell support criterion holds: fusions detected in many single cells are kept while single-cell amplification artifacts are discarded
- Only 1.8% of STAR-Fusion candidate fusions correspond to V(D)J recombinations, while scFusion's filtered calls detect them — bulk-method candidates confirm the artifact background being removed
- Disabling the deep-learning model visibly reduces precision, so precision on simulated/known fusions should be higher with the model enabled
- Rare true events may be missed (TRAJ20-TRAV1-2 not identified, possibly because it was rare in MAIT cells), so absence of a low-count fusion alone does not indicate filter failure

## Limitations

- Turning off the deep-learning model reduces the precision of scFusion
- Many potential artifacts are only removed by the ad hoc filters, so filter configuration affects results
- scFusion has reduced sensitivity for fusions with low expression, as is the case for all RNA-based methods
- Detection power is limited for rare fusions in highly heterogeneous tumor samples
- Choosing appropriate thresholds to define distinct cell populations based on fusion status is a difficult issue

## Evidence

- [intro] Amplification artifacts rationale: "the heavy amplification step may generate artificial chimeric reads, leading to false-positive fusion candidates"
- [intro] Two-model false-positive control: "scFusion employs a statistical model and a deep-learning model to control for false positives."
- [discussion] Multi-cell rationale: "scFusion takes advantage of the fact that multiple cells from a sample should contain the same fusion."
- [discussion] Deep-learning contribution: "When deep-learning model was turned off, the precisions of scFusion reduced"
- [discussion] Ad hoc filters: "We also noticed that many potential artifacts were also filtered by the ad hoc filters"
- [methods] Mappability filter optionality: "If you do not provide this file, scFusion will turn off the mappability filter."
- [readme] Tooling versions: [evidence span withheld — anchor status: not_found]
- [discussion] Rare-fusion miss: "The TRAJ20-TRAV1-2 was not identified possibly because it was rare in MAIT cells"
