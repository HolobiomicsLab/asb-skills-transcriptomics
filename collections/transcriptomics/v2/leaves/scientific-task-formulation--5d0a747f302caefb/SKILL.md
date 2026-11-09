---
name: scientific-task-formulation
description: Use when a candidate sub-task must be scoped to verifiable evidence —
  e.g. formulating the question of whether HEDGES-MSA overtakes HEDGES only above
  a specific i.i.d. error rate in the read/write Pareto frontiers.
when_to_use_negative:
- The section text already contains full methods content supporting the sub-task —
  then proceed to analysis rather than formulation/discard review.
- The question asserts universal optimality of a pipeline component (e.g. 'MSA always
  improves storage density') when the article explicitly states the choice 'is not
  always beneficial' — the premise itself must be reformulated.
- The task requires generating new simulation results rather than formulating questions
  answerable from the indexed findings and existing README-documented configurations.
tools:
- name: FrameD
  role: Fault-injection simulation test bed whose 1%/5% i.i.d. error results define
    the evidence base a formulated sub-task must be grounded in
  repo: https://github.com/dna-storage/framed
- name: DNArSim
  role: Nanopore channel simulator used in one of FrameD's two error-model branches
    (examples/DNArSim), relevant when scoping sub-tasks to realistic error models
    rather than i.i.d. errors
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad572
    title: 'FrameD: framework for DNA-based data storage design, verification, and
      validation'
  source_content_hash: sha256:0eef2f92bb38cadb5de3def1494a0012f4cc0aa7e971d40f11504ef196608657
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dna-storage/framed
  tools:
  - DNArSim
  - FrameD
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# scientific_task_formulation

## Summary

Formulates a scoped, evidence-grounded scientific sub-task (research question plus candidate finding) from an article's indexed entities, then validates the sub-task against available section text and discards it if unsupported. Applied here to FrameD's finding that multiple sequence alignment (MSA) in the DNA storage pipeline is beneficial only at certain error rates.

## When to use

Use when a candidate sub-task must be scoped to verifiable evidence — e.g. formulating the question of whether HEDGES-MSA overtakes HEDGES only above a specific i.i.d. error rate in the read/write Pareto frontiers.

## When NOT to use

- The section text already contains full methods content supporting the sub-task — then proceed to analysis rather than formulation/discard review.
- The question asserts universal optimality of a pipeline component (e.g. 'MSA always improves storage density') when the article explicitly states the choice 'is not always beneficial' — the premise itself must be reformulated.
- The task requires generating new simulation results rather than formulating questions answerable from the indexed findings and existing README-documented configurations.

## Inputs

- Candidate scoped sub-task (research question string plus expected finding)
- Enriched article index with tools, workflow steps, filter steps, and traced findings (e.g. the MSA error-rate dependence finding from the FrameD abstract)
- Section text chunk to be checked for supporting methods content
- Repository README describing runnable configurations (examples/iid and examples/DNArSim json files)

## Outputs

- Scoped research question with parameterized thresholds (e.g. 1% vs 5% i.i.d. error rate, HEDGES vs HEDGES-MSA inner code)
- Validated sub-task card with tools (FrameD, manual expert review) and workflow steps
- Discard decision with rationale when the section text lacks supporting methods content

## How to apply

First, state the research question with concrete parameters drawn from the article's entities — e.g. 'at which i.i.d. error rate does the HEDGES-MSA pipeline overtake the HEDGES pipeline for the same inner code configuration in the read/write density Pareto frontiers?'. Second, check the enriched index for a traced finding that directly answers or constrains the question, such as the abstract-level finding that including multiple sequence alignment 'is dependent on the error rate and the type of errors being injected and is not always beneficial'. Third, inspect the candidate section text to confirm it contains methods content (e.g. FrameD fault-injection test bed results at 1% and 5% i.i.d. error) supporting the scoped sub-task. Fourth, apply the reproducibility and evidence-span rules: if no methods content supports the sub-task, discard the candidate sub-task rather than fabricating unsupported claims. The rationale is that MSA's optimality is error-rate dependent (at 5% i.i.d. error HEDGES-MSA outperforms HEDGES, while at 1% HEDGES envelops HEDGES-MSA), so a sub-task claiming universal MSA benefit would be unsupportable.

## Related tools

- **FrameD** (Fault-injection simulation test bed whose 1%/5% i.i.d. error results define the evidence base a formulated sub-task must be grounded in) — https://github.com/dna-storage/framed
- **DNArSim** (Nanopore channel simulator used in one of FrameD's two error-model branches (examples/DNArSim), relevant when scoping sub-tasks to realistic error models rather than i.i.d. errors)

## Examples

```
cd $DNASTORAGE_HOME && tcsh examples/small/run_small.csh
```

## Evaluation signals

- The research question is parameterized with the article's own thresholds (1% and 5% i.i.d. error rates, HEDGES vs HEDGES-MSA inner code, read/write density Pareto frontiers) rather than generic phrasing.
- The expected finding is supported by a verbatim evidence span from the article (e.g. the abstract's MSA error-rate dependence finding).
- Sub-tasks with no supporting section-level methods content are recorded as discarded, not converted into fabricated claims.
- The formulated question is answerable via FrameD's documented fault-injection configurations (6 .json files under examples/iid and examples/DNArSim).
- The workflow log distinguishes formulation, evidence-check, and discard decisions per the reproducibility rules.

## Limitations

- Section-level indexing may lack paper-level metadata (the abstract section carried no DOI/title/authors), limiting how fully a sub-task can be contextualized from a single section.
- A candidate sub-task must be discarded when the provided section text contains no methods content — no claims may be extrapolated beyond indexed evidence.
- The repository README is truncated, so some configuration or analysis details may be unavailable for grounding sub-tasks.

## Evidence

- [abstract] MSA optimality is error-rate dependent, constraining any formulated sub-task about pipeline-component benefit: "We found that the choice to include multiple sequence alignment in the pipeline is dependent on the error rate and the type of errors being injected and is not always beneficial."
- [abstract] FrameD is the fault-injection test bed grounding answerable sub-tasks about DNA storage systems: "it provides a fault-injection-based test bed in which DNA storage systems can be evaluated"
- [readme] Concrete runnable configurations (i.i.d. vs DNArSim error models) exist to test a formulated sub-task: [evidence span withheld — anchor status: external_doc]
- [readme] Compiled results can be analyzed via db_gen.py to evaluate a formulated question: [evidence span withheld — anchor status: external_doc]
