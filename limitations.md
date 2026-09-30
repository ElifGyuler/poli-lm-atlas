# Limitations

## Corpus selection

The corpus is observational and source-selected rather than randomized.

Differences across leaders may reflect:

- source availability
- speech genre
- institutional setting
- historical period
- collection strategy
- document length

---

## Leader-language confounding

Each leader is primarily associated with one language in the project.

Therefore, leader and language effects cannot be cleanly separated.

A pattern observed for a leader cannot automatically be attributed to the leader rather than language or corpus construction.

---

## Topic measurement

Topic scores are semantic similarity measurements to multilingual anchor centroids.

They are not direct probabilities that a document "belongs" to a topic.

Raw topic centroids also differ in baseline similarity, which is why topic-specific z-scores are used as a diagnostic transformation.

---

## Emotion / affect measurement

The affect pipeline measures model-detected textual signals, not speakers' internal emotions.

The real-speech pilot is assistant-assisted rather than independently annotated human gold.

Neutral / no-affect detection was not validated strongly enough to support a final neutral classifier.

The final affect dimensions are therefore continuous exploratory signals.

---

## Prompt sensitivity

Zero-shot NLI scores depend on hypothesis wording.

The project observed strong over-selection of `surprise` under one hypothesis formulation.

This makes hypothesis wording part of the measurement instrument.

---

## Baseline correction

Label-wise neutral-baseline correction improves relative ranking but does not turn model scores into calibrated probabilities.

Adjusted scores may be negative and should be interpreted relative to the measurement pipeline, not as universal emotion intensity scales.

---

## Compute-driven sampling

The final affect stage uses systematic within-document chunk sampling rather than exhaustive scoring.

Up to three chunks are selected near the 20%, 50%, and 80% positions of each document.

This reduces compute while preserving broad document coverage, but it may miss brief affective passages between sampled regions.

The final affect profiles are therefore approximate sampled summaries.

---

## Causal interpretation

The project is descriptive.

It does not establish that:

- a leader caused a linguistic pattern
- a topic profile reflects ideological preference
- an affect score reflects internal emotion
- differences across leaders are language-independent
- model-derived associations are causal
