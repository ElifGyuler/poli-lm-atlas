# Methodology

## 1. Research design

POLI-LM Atlas is a multilingual computational text-analysis project examining patterns in public political communication.

The analysis corpus contains 794 documents from six political leaders, each primarily associated with one project language.

Because leader and language are structurally confounded, cross-leader comparisons are treated as descriptive corpus patterns rather than clean causal estimates.

---

## 2. Corpus construction

The canonical corpus contains 795 documents.

One Mélenchon text is an explicit Spanish translation and is retained in the canonical collection while excluded from cross-language analysis.

This separation preserves provenance and prevents analysis decisions from silently changing the source corpus.

Core metadata include:

- leader
- language
- date
- title
- source URL
- genre / setting where available

---

## 3. Linguistic features

Document-level linguistic features are extracted as descriptive measures.

The feature design is language-aware: features that depend on whitespace tokenization or sentence segmentation are not assumed to be equally reliable across all six languages.

Chinese receives separate character-aware handling where appropriate.

---

## 4. Multilingual embeddings

Model:

`BAAI/bge-m3`

Long documents are split into model-token chunks.

For each document:

1. tokenize
2. split into context-safe chunks
3. embed each chunk
4. compute token-weighted mean embedding
5. L2-normalize final document representation

This avoids silent truncation of long speeches.

The semantic layer is validated using same-topic and different-topic multilingual probes before downstream use.

---

## 5. Topic measurement

### 5.1 Rejected unsupervised approach

KMeans clustering was tested across multiple values of `k`.

The resulting clusters were strongly structured by leader/language rather than yielding a stable cross-lingual substantive policy taxonomy.

KMeans was therefore rejected as the primary topic-measurement strategy.

### 5.2 Anchor-centroid approach

A fixed 12-topic taxonomy is used.

Each topic is represented by multilingual anchor texts embedded with BGE-M3.

For each document chunk, cosine similarity is computed against all topic centroids.

Document-level scores are token-weighted means across chunks.

### 5.3 Topic baseline correction

Some topic centroids have higher corpus-wide cosine similarity than others.

To avoid interpreting raw winner frequencies as directly comparable topic prevalence, the project also computes topic-specific pooled-corpus z-scores.

For topic `t`:

```text
z_it = (score_it - corpus_mean_t) / corpus_sd_t
```

This standardizes each topic relative to its own corpus-wide score distribution.

### 5.4 Leader-level summaries

For each leader and topic:

- mean z-score
- bootstrap confidence interval
- CI relation to zero
- row-centered relative topic profile

Bootstrap intervals resample documents within leader while keeping the pooled topic baseline fixed.

These intervals describe conditional corpus uncertainty; they do not eliminate language, source, genre, or collection confounding.

---

## 6. Affect / emotion measurement

### 6.1 Measurement target

The project does not attempt to infer a leader's internal emotional state.

The target is **model-detected affective signal in public communication**.

### 6.2 Validation sequence

Candidate models are evaluated in stages:

1. explicit multilingual emotion probes
2. neutral procedural controls
3. assistant-assisted real-speech pilot
4. label-specific baseline diagnostics
5. compute / quality trade-off checks

### 6.3 Final model family

The final retained model is:

`MoritzLaurer/mDeBERTa-v3-base-mnli-xnli`

Each emotion dimension is represented as an NLI hypothesis.

The model scores entailment independently for each dimension.

### 6.4 Final dimensions

- anger
- contempt
- disgust
- fear
- frustration
- gratitude
- joy
- love
- sadness
- surprise

`neutral` is not treated as a final emotion dimension.

### 6.5 Label-wise neutral baseline adjustment

Each emotion has a multilingual neutral-control baseline.

Adjusted score:

```text
adjusted = (raw_score - neutral_mean) / (1 - neutral_mean)
```

This corrects label-specific baseline activation.

The adjusted score is not a calibrated probability.

### 6.6 Compute-aware chunk sampling

Exhaustive affect scoring is computationally expensive.

The final pipeline samples up to three chunks per document at approximately:

- 20%
- 50%
- 80%

of the chunk sequence.

Documents with three or fewer chunks retain all chunks.

Document-level affect profiles are token-weighted means across sampled chunks.

The result is therefore a **sampled document-level affect profile**.

---

## 7. Interpretation constraints

All final analysis should preserve the following distinctions:

- semantic similarity is not topic probability
- topic z-score is not political importance
- affect score is not internal emotion
- model score is not calibrated probability
- leader differences are not automatically language-independent
- source / genre / setting differences may contribute to observed patterns
