# POLI-LM Atlas

A multilingual computational study of political language, semantic emphasis, and model-detected affective signals across six political leaders.
Status: analytical core complete / near-complete. Remaining work is final figure generation, repository cleanup, reusable code consolidation, and optional chatbot integration.

Project overview
POLI-LM Atlas compares patterns in public political communication across six leaders and six languages:
- Donald Trump — English
- Recep Tayyip Erdoğan — Turkish
- Xi Jinping — Chinese
- Vladimir Putin — Russian
- Giorgia Meloni — Italian
- Jean-Luc Mélenchon — French
The canonical corpus contains 795 documents. One explicitly translated Spanish Mélenchon text is retained in the canonical corpus but excluded from cross-language analysis, leaving an analysis corpus of 794 documents.
The project focuses on three analytical layers:
1. linguistic features
2. multilingual semantic / topic structure
3. exploratory affective-signal measurement
The project does not infer internal emotional states, political motives, or causal ideological explanations from model outputs.
Why this project is structured around validation
The main engineering lesson of the project is that a model producing a score is not enough.
At every stage, I asked whether the measurement behaved sensibly across languages, document lengths, neutral text, and real political speech.
This led to a workflow where:
- controlled QA happens before full-corpus inference
- failed models are documented rather than hidden
- model baselines are inspected before interpretation
- language / leader / source confounds are kept explicit
- raw and adjusted measurements are preserved separately
Single research notebook
The public repository uses one integrated notebook:
notebooks/
└── poli_lm_atlas.ipynb
The notebook contains the full analytical narrative in sequence:
1. corpus audit
2. linguistic features
3. multilingual embeddings
4. topic analysis
5. affect / emotion measurement
6. final leader-level summaries and figures
The reusable implementation code should live under src/polilm/; the notebook is the research narrative and reproducible analysis entry point.
Pipeline summary
Corpus audit
The corpus is checked for:
- document counts
- language consistency
- missing values
- duplicates
- source metadata
- genre / setting metadata
- explicit analysis exclusions
The canonical corpus is preserved; exclusions occur only at the analysis layer.
Linguistic features
Document-level descriptive features are extracted for all canonical documents.
These include lexical and surface statistics, sentence-level measures where reliable, and language-aware fallbacks such as Han-character proxies for Chinese.
Multilingual semantic embeddings
Semantic representations use BAAI/bge-m3.
Long documents are split into model-token chunks, embedded independently, then aggregated using token-weighted mean pooling and final vector normalization.
A multilingual same-topic / different-topic QA probe is used before downstream analysis.
Topic measurement
An initial KMeans approach was tested and rejected because clusters were largely structured by leader/language rather than yielding a stable cross-lingual policy taxonomy.
The project instead uses a fixed multilingual anchor taxonomy with 12 topics:
- economy & jobs
- housing & cost of living
- immigration & borders
- climate & environment
- energy
- health & social welfare
- education
- AI & digital policy
- foreign policy & diplomacy
- defense & security
- democracy, institutions & law
- trade & industry
Topic centroids are created from multilingual anchors embedded with BGE-M3.
Documents are chunked, compared to topic centroids with cosine similarity, and aggregated with token-weighted means.
Because some topic centroids have higher corpus-wide similarity baselines than others, the project also stores:
- pooled-corpus topic z-scores
- leader-level mean z-scores
- bootstrap confidence intervals
- row-centered relative topic profiles as a sensitivity diagnostic
These are corpus-relative semantic measurements, not direct measures of ideology or policy preference.
Emotion / affect measurement
Emotion measurement required several model-selection rounds.
Model A — rejected
MilaNLProc/xlm-emo-t
It performed strongly on explicit multilingual emotion probes, but it had no neutral class and forced neutral procedural text into emotion labels.
Decision: reject.
Model B — rejected
tabularisai/multilingual-emotion-classification
It handled neutral controls well, but failed on real political speech.
Key pilot diagnostics:
- Top-1 hit rate: 0.25
- median best-gold probability: 0.035
- false-neutral rate on non-neutral samples: 72.7%
Decision: reject.
Model C — multilingual zero-shot NLI
MoritzLaurer/mDeBERTa-v3-base-mnli-xnli
Instead of using a fixed emotion classification head, each emotion is represented as an NLI hypothesis.
Example:
The speaker expresses fear.
The model scores each hypothesis independently.
The initial version over-selected surprise; narrowing that hypothesis improved real-speech ranking.
After label-wise neutral-baseline correction, the assistant-assisted pilot reached:
- Top-1: 0.773
- Top-2: 0.818
- Top-3: 0.955
The project therefore uses continuous model-detected affective signal dimensions rather than a final neutral/emotional classifier.
Final affect dimensions:
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
Scores are not treated as calibrated emotion probabilities.
Compute-aware final scoring
Exhaustive scoring of all 13,075 emotion chunks with the validated mDeBERTa NLI model was too slow on the available Apple MPS hardware.
A faster multilingual MiniLM NLI model reduced runtime substantially but degraded pilot quality:
- Top-1: 0.227
- Top-2: 0.591
- Top-3: 0.727
The faster model was rejected.
The final strategy keeps the validated mDeBERTa model and reduces compute using systematic within-document chunk sampling at approximately 20%, 50%, and 80% of each document.
This preserves beginning / middle / end coverage for every document while reducing total inference cost.
The resulting output is described as a sampled document-level affect profile, not exhaustive full-document emotion scoring.
Repository structure
poli-lm-atlas/
├── README.md
├── pyproject.toml
├── data/
│   ├── manifests/
│   ├── processed/
│   └── raw/
├── notebooks/
│   └── poli_lm_atlas.ipynb
├── src/
│   └── polilm/
├── docs/
│   ├── methodology.md
│   ├── decision_log.md
│   ├── results.md
│   └── limitations.md
├── figures/
├── site/
└── tests/
Reproducibility
Target environment:
- Python 3.11
- PyTorch
- Apple MPS where available
- Transformers
- SentenceTransformers
- pandas
- NumPy
- PyArrow
- scikit-learn
- Plotly
The final public release should freeze exact dependencies in pyproject.toml.
Interpretation rules
The project uses deliberately conservative language:
- topic results are semantic similarity measurements in the collected corpus
- affect results are model-detected affective signals
- leader differences may reflect language, source, genre, institutional setting, or collection effects
- model outputs do not establish internal emotional states
- row-centered and standardized profiles are diagnostics, not causal explanations
Remaining work
The analytical core is nearly complete.
Remaining work:
- merge the research workflow into notebooks/poli_lm_atlas.ipynb
- move stable reusable functions into src/polilm/
- add regression tests
- freeze environment dependencies
- generate final figures and tables
- build the public data-storytelling site
- optionally add a project chatbot
Optional chatbot
A project chatbot could answer questions about:
- corpus composition
- methodology
- topic measurement
- model-selection decisions
- validation failures
- final figures
- limitations
It should retrieve project evidence rather than generate unsupported political interpretations.
License / citation
Add final license, source notes, and citation metadata before public release.
