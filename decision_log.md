# POLI-LM Atlas — Research & Engineering Decision Log

This document records major design decisions, rejected approaches, and why the pipeline changed.

The goal is to make the project auditable: a failed model or abandoned method is part of the research process, not something to hide.

---

## 01 — Preserve canonical corpus, exclude only at analysis layer

**Problem**

One Mélenchon document was an explicit Spanish translation rather than an original French-language source.

**Decision**

Keep it in the canonical corpus but exclude it from multilingual analysis.

**Result**

- canonical corpus: 795 documents
- analysis corpus: 794 documents

**Why this matters**

Deleting source material would blur the difference between collection and analysis decisions.

---

## 02 — Long-document embeddings need chunking

**Problem**

Some documents exceeded the semantic model's context window.

**Decision**

Chunk by model tokens, embed chunks independently, use token-weighted mean pooling, then normalize the final document vector.

**Why this matters**

Silent truncation would disproportionately discard material from long speeches.

---

## 03 — Unsupervised topic clustering did not produce the intended construct

**Attempt**

KMeans over multilingual semantic embeddings, testing multiple values of `k`.

**Observed problem**

Clusters were strongly structured by leader/language rather than yielding a stable cross-lingual policy-topic taxonomy.

**Decision**

Reject KMeans as the main topic-measurement strategy.

**Replacement**

Use a fixed 12-topic multilingual anchor taxonomy and cosine similarity to topic centroids.

---

## 04 — Raw topic winner counts were biased by topic-specific cosine baselines

**Observed problem**

Some topic centroids had systematically higher corpus-wide similarity than others.

Top-1 topic frequency was strongly associated with a topic's raw corpus mean.

**Decision**

Do not interpret top-1 winner counts alone.

Add:

- topic-specific pooled-corpus z-scores
- leader means
- bootstrap confidence intervals
- row-centered relative profiles as a sensitivity diagnostic

**Interpretation constraint**

These are corpus-relative semantic measurements, not direct measures of political importance, ideology, or preference.

---

## 05 — Emotion Model A failed because it had no neutral option

**Model**

`MilaNLProc/xlm-emo-t`

**Positive-control result**

Very strong multilingual performance on explicit anger/fear/joy/sadness probes.

**Failure**

Neutral procedural sentences were still forced into emotion classes, often with high confidence.

**Decision**

Reject.

**General lesson**

A forced-choice classifier can look excellent on positive controls and still be unusable when the absence of the target construct matters.

---

## 06 — Emotion Model B passed synthetic QA but failed real political speech

**Model**

`tabularisai/multilingual-emotion-classification`

**Synthetic result**

- explicit emotion probes: strong
- neutral controls: excellent

**Real-speech pilot**

24 usable assistant-assisted examples after excluding co-speaker contamination.

Key results:

- Top-1 hit: 0.25
- median best-gold probability: 0.035
- false-neutral rate on non-neutral examples: 72.7%

**Decision**

Reject.

**General lesson**

Passing synthetic multilingual controls does not establish ecological validity in political speech.

---

## 07 — Speaker contamination must be screened

Two candidate Trump validation chunks included other speakers (`Yair Lapid:` / `The Press:`).

**Decision**

Exclude contaminated samples and replace them.

**General lesson**

Speaker purity is a measurement requirement whenever document sources contain Q&A, interviews, or multi-speaker transcripts.

---

## 08 — Zero-shot multilingual NLI was more promising than fixed emotion heads

**Model**

`MoritzLaurer/mDeBERTa-v3-base-mnli-xnli`

**Approach**

Represent each emotion as an NLI hypothesis.

Example:

`The speaker expresses fear.`

Use independent entailment scores (`multi_label=True`).

**Initial result**

Much better gold-label ranking than Model B, but `surprise` was strongly over-selected.

---

## 09 — Hypothesis wording can create label hubness

**Observed problem**

`surprise` became the top label for 9/24 pilot samples.

**Ablation**

Narrow the surprise wording.

**Result**

- Top-1 improved from 0.50 to 0.75
- Top-2 remained 0.792
- Top-3 remained 0.833
- surprise top count fell from 9 to 1

**Interpretation**

The correct affective signal was often already present, but ranking was distorted by hypothesis wording.

---

## 10 — Neutral should not be treated as an ordinary emotion dimension

A two-stage affect-vs-neutral gate was tested.

**Result**

- overall accuracy: 0.75
- affect recall: 0.818
- neutral recall: 0.0

**Decision**

Do not use a final neutral classifier.

Move toward continuous affect dimensions instead.

---

## 11 — Synthetic-control optimization caused prompt overfitting

More detailed emotion hypotheses reduced neutral baselines dramatically on synthetic controls.

However, on the real-speech pilot:

- non-neutral Top-1 fell to 0.364
- Top-3 remained 0.818

**Decision**

Return to the simpler C1 hypothesis definitions.

**Interpretation**

The revised prompts fit the control set better but generalized worse to the target domain.

---

## 12 — AUC = 1.0 did not imply a clean baseline

Explicit emotion controls often produced scores near 1.0.

Because positives saturated at the top of the scale, every emotion could achieve control AUC = 1.0 even when neutral texts still had large absolute scores.

**Decision**

Inspect both:

- positive-vs-neutral ranking
- absolute neutral baseline activation

**General lesson**

AUC and ranking metrics do not replace baseline calibration.

---

## 13 — Label-wise neutral-baseline correction improved real-speech ranking

Correction:

`(raw_score - neutral_mean) / (1 - neutral_mean)`

**Pilot result**

- Top-1: 0.773
- Top-2: 0.818
- Top-3: 0.955

**Important limitation**

The two neutral pilot examples still received high adjusted affect scores.

**Decision**

Use adjustment for relative continuous affect measurement, not for neutral detection.

---

## 14 — Faster MiniLM model was rejected

**Candidate**

`MoritzLaurer/multilingual-MiniLMv2-L6-mnli-xnli`

**Advantage**

Full-corpus ETA dropped to ~36 minutes.

**Pilot quality**

- Top-1: 0.227
- Top-2: 0.591
- Top-3: 0.727

**Decision**

Reject.

**Reason**

Compute savings were not worth the measurement-quality loss.

---

## 15 — Compute was reduced by sampling chunks, not replacing the validated model

The validated mDeBERTa NLI model was too slow for exhaustive 13,075-chunk scoring on the available MPS hardware.

**Final strategy**

Sample up to three chunks per document near:

- 20% of document
- 50% of document
- 80% of document

For documents with three or fewer chunks, retain all chunks.

**Scale**

- 794 documents
- 2,279 selected chunks
- 76 already available from an earlier checkpoint
- 2,203 remaining model calls at start of final run

**Decision**

Describe the result as a **systematically sampled document-level affect profile**.

Do not describe it as exhaustive full-document emotion scoring.

---

## Final interpretation rules

Topic outputs are corpus-relative semantic similarity measurements.

Affect outputs are exploratory model-detected affective signals.

Neither should be interpreted as:

- a direct measurement of political ideology
- a causal explanation of leader behavior
- a measurement of internal emotional state
- proof that one language or leader is intrinsically more emotional than another

Leader, language, source, genre, and institutional context remain important confounds.

---

## Remaining engineering work

- move stable notebook functions into `src/polilm/`
- add regression tests for corpus counts, exclusions, score shapes, and manifests
- freeze environment dependencies
- build final visualizations
- build public site
- optional project chatbot / retrieval assistant
