---
name: velocyto-spliced-unspliced-count-matrix-generation
description: Use when starting from raw single-cell RNA-seq reads (e.g. Burns et al.
  GSE71982, 157 neonatal mouse inner ear cells) and you need spliced/unspliced count
  layers for RNA velocity estimation and Cytopath trajectory inference.
when_to_use_negative:
- Spliced/unspliced count layers already exist (e.g. a loom or h5ad from velocyto
  is already provided) — go directly to scvelo/Cytopath
- Root and terminal cell states are not clearly identifiable and the underlying RNA
  velocity estimation is expected to be faulty, e.g. features with time-dependent
  degradation rates — the article's authors do not recommend trajectory inference
  in such cases
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: velocyto
  role: Generates the spliced/unspliced/ambiguous count matrices from raw reads for
    the neonatal mouse inner ear dataset (GSE71982)
  repo: https://github.com/velocyto-team/velocyto-notebooks
- name: scvelo
  role: Loads velocyto count matrices into an AnnData object, preprocesses the data,
    and estimates RNA velocity (>=0.1.25); Cytopath depends on scvelo to process the
    data
- name: Cytopath
  role: 'Downstream consumer: uses the RNA velocity field from the velocyto/scvelo
    layers to define a Markov chain model and simulate differentiation trajectories'
  repo: https://github.com/aron0093/cytopath
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:241d2738ae36304ddd71c14ef6ed6246e3a94e3cbe648ce267313dab5cb8bbf1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/velocyto-team/velocyto-notebooks
  tools:
  - Cytopath
  - scvelo
  - velocyto
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# velocyto-spliced-unspliced-count-matrix-generation

## Summary

Generate spliced/unspliced/ambiguous count matrices from raw single-cell reads with velocyto, supplying the transcriptional-activity layers that RNA-velocity-based trajectory inference (scvelo + Cytopath) requires.

## When to use

Use when starting from raw single-cell RNA-seq reads (e.g. Burns et al. GSE71982, 157 neonatal mouse inner ear cells) and you need spliced/unspliced count layers for RNA velocity estimation and Cytopath trajectory inference.

## When NOT to use

- Spliced/unspliced count layers already exist (e.g. a loom or h5ad from velocyto is already provided) — go directly to scvelo/Cytopath
- Root and terminal cell states are not clearly identifiable and the underlying RNA velocity estimation is expected to be faulty, e.g. features with time-dependent degradation rates — the article's authors do not recommend trajectory inference in such cases

## Inputs

- Raw single-cell RNA-seq reads or cellranger output BAM (e.g. GEO accession GSE71982, Burns et al., 157 neonatal mouse inner ear cells)
- Genome annotation file (GTF/GFF) for spliced/unspliced/ambiguous assignment
- Cell barcode metadata

## Outputs

- Spliced count matrix
- Unspliced count matrix
- Ambiguous count matrix (typically a .loom file)
- AnnData (.h5ad) object with spliced/unspliced layers ready for scvelo preprocessing

## How to apply

Run velocyto on the raw reads (alignments plus genome annotation) to produce spliced, unspliced, and ambiguous count matrices, typically exported as a .loom file with cell and gene metadata. Load the count matrices into an AnnData object and use scvelo (>=0.1.25) to preprocess the data and estimate RNA velocity, since Cytopath depends on scvelo to process the data and consumes the velocity field to define its Markov chain model. Velocyto's spliced/unspliced layering is the prerequisite for Cytopath's root/terminal state estimation from the Markov random-walk model and for simulating an ensemble of differentiation trajectories. Correct layering matters: ambiguous reads must be retained rather than discarded, as Cytopath's transition probabilities are derived from the velocity information these layers encode.

## Related tools

- **velocyto** (Generates the spliced/unspliced/ambiguous count matrices from raw reads for the neonatal mouse inner ear dataset (GSE71982)) — https://github.com/velocyto-team/velocyto-notebooks
- **scvelo** (Loads velocyto count matrices into an AnnData object, preprocesses the data, and estimates RNA velocity (>=0.1.25); Cytopath depends on scvelo to process the data)
- **Cytopath** (Downstream consumer: uses the RNA velocity field from the velocyto/scvelo layers to define a Markov chain model and simulate differentiation trajectories) — https://github.com/aron0093/cytopath

## Examples

```
velocyto run -o output.loom -b barcodes.tsv aligned.bam genes.gtf  # then: scv.read('output.loom') for scvelo (>=0.1.25) preprocessing
```

## Evaluation signals

- Output contains all three count layers — spliced, unspliced, and ambiguous — with consistent cell x gene dimensions
- For the Burns et al. neonatal mouse inner ear dataset, the matrices must cover the expected 157 cells
- The loom/h5ad loads cleanly into an AnnData object under scvelo (>=0.1.25) preprocessing without missing-layer errors
- Downstream Cytopath runs identify clear root and terminal states and yield high Spearman correlation between inferred pseudotime and known cell type ordering, robust across multiple runs
- Robustness check: pseudotime is robust to root and terminal probability thresholds within a range of [0.85, 1]

## Limitations

- RNA velocity estimation from spliced/unspliced ratios can be unrepresentative of true transcriptional dynamics due to simplifying assumptions of time-invariant rates of transcription, splicing, and degradation and a single regulatory regime per gene
- If the velocity estimation itself is faulty (e.g. spurious root points in the erythroid gastrulation dataset), trajectory inference downstream is not meaningful
- Correctness of the velocity field cannot be verified in every scenario, even when marker-based information is available

## Evidence

- [abstract] Cytopath takes advantage of transcriptional activity information from RNA velocity: "We report Cytopath, a method for trajectory inference that takes advantage of transcriptional activity information from the RNA velocity of single cells"
- [readme] README: RNA velocity defines the Markov chain model: "Transcriptional activity information from RNA velocity of single cells is used to define a Markov chain model; simulation of this model yields an ensemble of possible differentiation trajectories"
- [discussion] Discussion: faulty velocity breaks downstream inference: "We do not expect that trajectory inference with Cytopath is meaningful when the underlying RNA velocity estimation itself is faulty."
- [discussion] Discussion: velocity assumptions: "These issues arise from the simplifying assumptions of time-invariant rates of transcription, splicing, and degradation as well as the assumption of each gene operating under a single regulatory"
