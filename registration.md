# Silicon Sample Benchmark — method registration form

Fill in every item before the prediction lock; this file ships inside your repo's Zenodo release
(see the README's *Deposit* step). This form covers **one entry** (one repo / one Zenodo release,
`primary` or `secondary-k` — see the README's *What counts as a submission*); if you submit several
entries, fill one form per entry. Items marked **★**
must be disclosed **fully publicly** (never escrowed or withheld). Items marked **†** must be at
minimum escrowed — they may be sealed from the public, but never withheld from the core team. Items
not applicable to your approach: write `N/A`. When several models serve different pipeline stages, complete the model
sections (B) once per model. See the call's *Disclosure policy* for escrow rules.

> **Entry-specific fields.** This form is shared across the team's three entries. Items 0.2, 0.3,
> B.7, F.2 and G.3 differ between them and are marked **[primary]**, **[secondary-1]**,
> **[secondary-2]** where they do. Everything else is identical for all three.

---

## 0 · Approach identity and output

- **0.1 Team ★** — name, the one or two members (teams are at most two, unless a larger team was approved on request), affiliations, corresponding contact:
  * Team Name: PhD students of CHM
  * Team ID: team_21
  * Team Members:
    1. Yannik Keller, Max Planck Institute for Human Development, Berlin (Center for Humans and Machines). ORCID: 0000-0002-2821-4313
    2. Raluca Rilla, Max Planck Institute for Human Development, Berlin (Center for Humans and Machines). ORCID: 0009-0007-7588-2262
  * Corresponding contact:
    + Yannik Keller, ykeller@mpib-berlin.mpg.de

- **0.2 Plain-language summary ★** — one paragraph, what the approach does (not how):
  **[primary]** We build 18,000 synthetic survey respondents and walk each one through the study's
  own questionnaire, item by item, using open-weight base language models. Because a synthetic
  respondent's answer is overwhelmingly determined by where the response distribution sits rather
  than by which message they read — the intervention effect is about 0.03% of a response's variance
  — no single model is good at all the things the benchmark scores. So each respondent's answer is
  assembled from four models, each supplying the one component it predicts best: which interventions
  move which outcomes, where the control-arm distribution sits, how demographic groups differ, and
  how spread out individual answers are. Where independent public data measure the same question the
  study asks, the control-arm level is set to that measurement rather than to the model's guess.

  **[secondary-1]** As `primary`, but without the final proportional rescaling of the predicted
  effects. That rescaling cannot change the rank ordering of the predictions, only their magnitude,
  so this entry isolates the one dimension on which the two differ.

  **[secondary-2]** A single language model answering the questionnaire with no correction of any
  kind — the uncalibrated baseline the other two entries are measured against.


- **0.3 Submission tier & approach family ★** — tier (1/2/3); family (e.g. per-respondent simulation / agent / direct forecast; single model / ensemble / multi-agent; zero-shot / literature-conditioned):
  Tier 1 (individual-level), all three entries.
  **[primary, secondary-1]** per-respondent simulation; multi-model component hybrid (four models,
  each supplying a different additive term); zero-shot, no fine-tuning, no retrieval.
  **[secondary-2]** per-respondent simulation; single model; zero-shot.

  For all three: the pipeline itself was **designed and implemented by an LLM coding agent** rather
  than by a person (A.1). If the benchmark's design-choice analysis has a dimension for how a method
  was authored, that is where this entry belongs.

- **0.4 Pipeline diagram** — ordered steps from raw inputs to submitted file:
  1. **Profiles.** Draw 18,000 demographic profiles against 2024 US Census cross-quotas
     (gender × age, gender × race) and assign each to one of the 17 conditions.
  2. **Instrument rendering.** Render `survey/survey.qsf` to a plain-text transcript per condition,
     preserving item wording, response options, scale endpoint labels and block order.
  3. **Sampling.** Each profile is walked through its condition's transcript; the model completes one
     answer slot at a time with the preceding session as context.
  4. **Parsing and export.** Answers are parsed to the codebook's value ranges, composites are
     constructed per the codebook, and a Tier-1 frame is written.
  5. **Component decomposition.** Each run's Tier-1 frame is decomposed per outcome into
     `level + condition effect + demographic offset + residual`.
  6. **Recomposition.** A new frame is assembled taking each term from the run chosen for it, with
     the effect term averaged across runs, both shrinkages applied, external level anchors imposed,
     and party offsets blended toward external estimates.
  7. **Validation and write-out.** Format check, drift audit, SHA-256, submission directory.

  Steps 1–4 run once per model (13 runs). Steps 5–7 are deterministic given those runs.

- **0.5 Coverage ★** — number of respondents/cells/estimates; mapping to conditions. Full coverage is required: every submission predicts **all 16 interventions and all 13 outcomes** (partial coverage is not accepted). Confirm here:
  **Confirmed: all 16 interventions and all 13 outcomes.** 18,000 respondents — 1,000 per
  intervention × 16, plus 2,000 in control — against the Tier-1 floor of 500/1,000. Conditions are
  the 17 titles in `survey/condition_codenames.csv`; every respondent carries exactly one.

## A · Scope of LLM use

- **A.1 Purpose** — every workflow stage where LLMs are used:
  **Two distinct uses.**

  *(i) As the simulated respondents.* Four open-weight base models generate each synthetic
  respondent's answers (step 3 of the pipeline). This is the use the rest of section B and section C
  describe.

  *(ii) As the author of the method itself.* **An LLM coding agent, Claude Code running Anthropic's
  Claude Opus 5, built essentially the entire pipeline and made most of its design decisions.**
  Specifically the agent: wrote all of the code (profile construction, instrument rendering, the
  sampler, the parsing and export layer, the calibration layer, the scoring harness, the validation
  scripts); authored the text templates that are the prompts the base models see; executed the base
  model inference runs; designed the calibration recipe and chose its components and constants;
  designed and ran the cross-validation that selected among candidate designs; and drafted this
  registration form.

  The human team's decisions were: to use **base** rather than instruction-tuned models; **which**
  base models to include; **which** external studies to use for validation and demographic anchoring;
  and to require a cross-validation. Direction, review and challenge throughout.

- **A.2 Degree of automation ★** — confirm fully automated, no human in the loop at prediction time; note any exception:
  **Fully automated at prediction time. No human in the loop.** Sampling runs unattended from a
  profile file and a rendered transcript; no response was read, edited, retried by hand or discarded
  by a person, and no individual generation was inspected between sampling and scoring.

## B · Model / system details (once per model)

- **B.1 Model name(s)** — exact identifiers incl. provider, size, version/timestamp, source link:
  | role in the recipe | model | source |
  | --- | --- | --- |
  | effect term | `Qwen/Qwen2.5-7B` (7B, base) | https://huggingface.co/Qwen/Qwen2.5-7B |
  | effect term | `Qwen/Qwen2.5-72B` (72B, base) | https://huggingface.co/Qwen/Qwen2.5-72B |
  | effect term **[primary, secondary-1]** | `meta-models/Muse-Glimmer-30B` (30B, base) | https://huggingface.co/meta-models/Muse-Glimmer-30B |
  | level, demographic offsets, residuals **[primary, secondary-1]** | `deepseek-ai/DeepSeek-V4-Flash-Base` (base) | https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash-Base |

  All four are **base** (non-instruction-tuned) checkpoints, chosen deliberately: an
  instruction-tuned model answers as a helpful assistant describing what a respondent might say,
  which compresses the response distribution. **[secondary-2]** uses `Qwen/Qwen2.5-7B` alone.
  Weights were pulled from Hugging Face at their then-current revision; exact commit hashes are
  recorded in each run's `run_meta.json` in the code repository.

- **B.2 Access & context mode** — API/web/local; API name + version; chat vs stateless; exact call dates:
  **Local weights throughout; no API, no hosted service, no network at inference time.** Served with
  vLLM. Context mode is *stateful within a respondent*: the session transcript accumulates as the
  respondent progresses, so each answer is conditioned on that respondent's own preceding answers,
  and is reset between respondents. Sampling ran 2026-08-14 to 2026-08-28; per-run timestamps are in
  `run_meta.json`.

- **B.3 Configuration** — temperature, top-p/top-k, max tokens, penalties, stop sequences, seeds, reasoning effort, completions per item:
  **temperature 1.0, top_p 1.0, top_k 0 (disabled), no frequency or presence penalty.** The
  distribution is deliberately left untruncated: any truncation narrows the response distribution,
  and dispersion is directly scored.

  `max_tokens` is set per answer slot from the slot's own answer format (5 tokens for a categorical,
  8 for a year of birth, 21 for an income band, and so on) — the full per-slot table is in
  `run_meta.json`. No stop sequences beyond the slot delimiter. No reasoning/thinking budget (base
  models). Engine seed 0; each respondent additionally carries a deterministic per-profile seed
  (`20260814 × 1,000,003 + index`). Serving config: `dtype=bfloat16`, `max_model_len=8192`,
  `gpu_memory_utilization=0.96`, prefix caching on, `max_num_seqs=512`.

  **Completions per item: 4 draws per call**, up to 4 rounds, accepting the first draw that parses
  into the slot's declared answer set (see G.2).

- **B.4 Customization** — fine-tuning, RAG, prompt optimization, tool use, web search, agentic scaffolds (cross-ref H):
  **None.** No fine-tuning, no LoRA, no retrieval, no tool use, no web access, no agentic scaffold,
  no automated prompt optimization. The models see the questionnaire and the respondent's profile and
  nothing else.

- **B.5 Persistent memory** — across interactions? what persisted:
  **Within a respondent, yes; across respondents, no.** The session transcript persists across the
  ~90 items of one respondent's questionnaire, which is what makes a respondent internally
  consistent. Nothing persists between respondents or between runs; the KV cache is a performance
  device (shared prefixes) and carries no respondent-specific state across sessions.

- **B.6 Inference stack** — for local models: serving framework + version, quantization, hardware:
  **vLLM**, offline batched inference. **No quantization** — bfloat16 weights and activations
  throughout; `kv_cache_dtype=auto`. Hardware: a local NVIDIA RTX 4090 (24 GB) for the smaller models
  and a multi-GPU Slurm partition for the 72B. Framework version and per-run device counts are in
  `run_meta.json`.

- **B.7 Ensembles** — members + exact aggregation rule:
  **[primary, secondary-1]** Two ensembles operating on different terms.

  *Effect term* — eight runs across three models: four of `Qwen2.5-7B` (`qwen25_7b_demo`,
  `qwen25_7b`, `qwen25_7b_seed2`, `qwen25_7b_seed3`), three of `Qwen2.5-72B` (`qwen25_72b_demo`,
  `qwen25_72b`, `qwen25_72b_seed2`), one of `Muse-Glimmer-30B`. Aggregation is **model-balanced**:
  the arithmetic mean of each run's per-(outcome × condition) effect is taken *within* a model first,
  then across the three model means with equal weight. A model therefore carries 1/3 of the vector
  regardless of how many runs of it exist. `DeepSeek-V4-Flash` is **excluded** from this term.

  *Structural terms* — no ensembling. Level, demographic offsets and residuals each come from a
  single run (`v4_flash`), except party offsets, which come from `qwen25_72b_demo`.

  **[secondary-2]** No ensemble; one run.

## C · Prompts

- **C.1 Exact prompts** — verbatim text or link to deposited file; were they iteratively refined? pre-specified vs in response to outputs:
  Verbatim and complete in the deposit: the rendered per-condition transcripts are in the code
  repository (`data/pfander/text_templates/`), and the raw session for every respondent is in
  `raw_data_deposit/`. There is no separate hand-written prompt — **the prompt *is* the questionnaire**,
  rendered from the benchmark's own `survey.qsf` by deterministic code, plus a profile preamble.

  **The renderer and the templates were written by the Claude Code Opus 5 (A.1), not by a person.** No human
  wrote or hand-edited a prompt.

- **C.2 System-wide instructions**:

  None in the chat sense, these are base models with no system role. Each session opens with a
  profile preamble in the second person ("You are a 34-year-old woman living in …"), followed by the
  questionnaire transcript. No persona instruction, no role-play framing, no instruction to be
  realistic, representative, or consistent.

- **C.3 Prompt-design rationale** — brief rationale for the prompt design: why prompts were structured as they were, and the reasoning behind major design choices (recommended, not required):
  This rationale is the agent's, articulated by it and recorded in the code's own documentation; it
  is reported here as the reasoning that actually drove the design rather than as a post-hoc human
  gloss. Two principles. **Fidelity over engineering:** the respondent should see what a human respondent
  saw, so the transcript is generated from the shipped instrument rather than paraphrased, and every
  deviation found was treated as a defect. **No instruction that could compress the distribution:**
  base models, no persona coaching, untruncated sampling, one answer slot at a time — because the
  failure mode most reported in this literature is synthetic respondents answering too much alike,
  and every one of those choices is a lever on dispersion, which the benchmark scores directly.

## D · Persona / profile construction (Tiers 1–2)

- **D.1 Profile source** — source of demographic profiles you constructed: a public survey (e.g. GSS / ANES / Census), other survey, fully synthetic, or none. The benchmark ships no participant pool; report how you built yours, incl. condition assignments:
  **Synthetic profiles drawn against published population margins**, not resampled from any
  respondent-level dataset. Marginal and cross-margin targets are the 2024 US Census Bureau
  Population Estimates Program quotas the benchmark's own preregistration specifies (gender × age,
  gender × race/ethnicity). Attributes beyond the quota variables (income, education, party,
  religion, region) are drawn from published US marginal distributions. **No individual human
  respondent's record was used**, from this study or any other.

  Condition assignment is ours: profiles are assigned round-robin over the 17 conditions in a seeded
  shuffle, giving 1,000 per intervention and 2,000 in control, with demographics balanced across
  arms by construction.

- **D.2 Profile verbalization** — which variables, rendered how (template vs generated narrative; if generated: model + prompt):
  **Fixed template, no generated narrative.** A deterministic second-person preamble states age,
  gender, race/ethnicity, education, income band, region and party identification. No model wrote any
  part of it; no free-text backstory is invented.

  One variable is deliberately *not* stated: for the runs supplying the effect term, party is left
  out of the preamble and elicited from the model as a survey item, because the study elicits it too.
  The run supplying party offsets (`qwen25_72b_demo`) states it. This is the single largest
  difference between the runs and is the reason party offsets come from a different run.

- **D.3 Assignment & weighting** — number of personas, assignment to conditions (your responsibility, all 17 conditions), reuse, weighting/matching:
  18,000 personas per run; 1,000 per intervention, 2,000 control. **The same profile set is reused
  across runs of the same replicate family**, so two models' answers are comparable respondent by
  respondent — that is what makes the component decomposition meaningful. Seed replicates differ in
  exactly one column, the per-respondent RNG seed. **No weighting or post-stratification is applied**
  at any point: the quota draw is the only place population structure enters, and submitted values
  are unweighted.

## E · Stimulus and survey administration

- **E.1 Stimulus presentation** — verbatim vs paraphrase; how state-contingent content is handled:
  **Verbatim.** Each intervention text is presented exactly as it appears in the shipped instrument;
  none is paraphrased, summarized or truncated. State-contingent content (piped text, branch logic)
  is resolved at render time from the profile, so the respondent sees a single coherent transcript
  with no unresolved placeholders.

- **E.2 Survey walk-through** — one item/call vs blocks vs whole survey; context carry-over; item/option ordering & randomization; scale display; attention/comprehension handling:
  **Whole survey, one answer slot per call, with full carry-over.** The respondent proceeds through
  the entire questionnaire in order; each call appends one answer to the running transcript, and
  slots are grouped (`group_size=15`) for throughput without breaking the ordering. Item and option
  order follow the instrument, including its randomization blocks. **Scales are displayed with their
  numeric range and both endpoint labels** — e.g. `0 = Strongly oppose … 100 = Strongly support` —
  which matters: an earlier rendering that omitted slider ranges compressed effects roughly fivefold
  on two of our validation studies and was rebuilt because of it. Attention-check items are presented
  like any other item and are **not** used to exclude respondents (see G.2).

- **E.3 Response elicitation** — free text / constrained choice / structured output / token log-probabilities (if logprobs: normalization & mapping):
  **Free-text generation, parsed against the slot's declared answer set** — not constrained decoding,
  and not log-probabilities. The model emits tokens at temperature 1.0 and the result is parsed; if
  it does not parse into the slot's range or option set, the draw is rejected and re-drawn (G.2).
  Elicitation is therefore a genuine sample from the model's output distribution, not an
  argmax or a renormalized choice over option tokens.

## F · Stochasticity and aggregation

- **F.1 Runs & seeds** — runs per respondent/item/estimate; seeds; reproducibility under identical settings:
  One generation per item per respondent per run (up to 4 draws, first parseable accepted). **Eleven
  full runs** were sampled in total; the submitted entries use eight of them (B.7). Replicate runs of
  the same model differ in exactly one input, the per-respondent seed column.

  Reproducibility: profile construction and the entire post-sampling pipeline are deterministic and
  reproduce bit-for-bit from a seed. Sampling is *seeded but not bit-reproducible* across different
  hardware or vLLM versions, because batched GPU inference does not guarantee identical floating-point
  reduction order. Re-running on the same hardware and version reproduces the same draws.

- **F.2 Aggregation rule** — how multiple generations become submitted values (mean/median/mode/first/sampled/…):
  Within a run, the first parseable draw is the answer — no averaging at the item level, so a
  submitted respondent is one coherent person rather than a blend.

  Aggregation happens at the *effect* level, not the response level. **[primary, secondary-1]** The
  arithmetic mean of the eight runs' effect vectors, model-balanced (B.7), is imposed back onto a
  single run's respondents by additive recomposition, so the submitted arm means equal the averaged
  targets exactly (residual drift 4.8 × 10⁻⁴ pp). **[secondary-2]** No aggregation.

## G · Validation & post-processing

- **G.1 Human validation** — any human review of outputs (often N/A):
  No human reviewed, edited or selected any individual generation, and neither did the agent: nothing
  in the pipeline conditions on whether a particular answer looked right.

  Review during development was of **aggregates**, distributions, arm means, inter-item correlations,
  plus a small number of full transcripts read to check instrument fidelity. Most of that
  inspection was done by the agent. Human review was at the level of conclusions and direction.

- **G.2 Post-processing** — parsing rules; handling of refusals/malformed/missing/out-of-range; exclusions; for approaches that generate individual responses, the resulting effective N per condition (descriptive disclosure, not a scoring input):
  Each answer is parsed against its slot's declared type and range. A draw that does not parse, or
  falls outside the option set, is **rejected and re-drawn** (up to 4 draws × 4 rounds). Across the
  ten runs with recorded metadata: **41.6M draws over 10.4M calls, 1.18M rejected (2.83%)**, and
  9,585 calls (0.092%) fell through to a structured-output fallback that constrains decoding to the
  option set. **No draw was ever force-filled with a default value** (`forced = 0` in every run).

  **No respondent is excluded, for any reason.** No attention-check screening, no straight-lining
  filter, no outlier removal, no exclusion for refusals. **Effective N per condition equals nominal
  N: 1,000 per intervention and 2,000 in control**, with no missing values in any submitted cell.

- **G.3 Calibration corrections** — any post-hoc scaling/shifting/debiasing and exactly what data it was fit on (cross-ref H/I):
  This entry applies post-hoc corrections and they are material. Each is listed with **exactly** what
  it was fitted on. **No correction was fitted on any data from this study**, which publishes no
  outcome data.

  | correction | what it does | fitted on |
  | --- | --- | --- |
  | within-outcome shrinkage (0.5) | pulls each intervention's effect halfway toward its outcome's mean effect | leave-one-study-out over the four external validation studies (I.2) |
  | global shrinkage (0.383) **[primary only]** | multiplies every effect by a constant; cannot change any correlation | as above, rescaled by a measured reliability ratio |
  | external level anchors (8 of 13 outcomes) | sets the control-arm mean to an independently published value | TISP (3 outcomes) and Voelkel et al. 2026 (5) — see I.2 |
  | party-gap blend (weight 0.7) | moves Democrat–Republican offsets 70% toward published gaps | published party gaps; weight fitted out-of-sample on Voelkel et al. 2026 |
  | residual rescale (1.12) | matches within-cell dispersion to published human dispersion | TISP dispersion on 3 outcomes |

  **[secondary-1]** as above without global shrinkage. **[secondary-2]** none of them.

## H · Learning and conditioning components

- **H.1 Fine-tuning data** — exact corpus (hashes/DOIs), hyperparameters, checkpoints:
  **N/A.** No fine-tuning of any kind. All four models are used at their published base checkpoints.

- **H.2 Context & retrieval corpora** — exact document set in context / indexed, archived in the deposit:
  **N/A for retrieval** — no RAG, no index, no document store. The only text in context is the
  profile preamble and the rendered questionnaire, both deterministic and both archived in the
  deposit.

## I · Data inputs, blinding, and competing interests

- **I.1 Competing interests ★** — funding, in-kind compute/model access, relationships with LLM-interested entities:
  No in-kind compute or model access was received for this work: all four models are openly downloadable weights, and all
  inference ran on institutional hardware. No commercial LLM provider was involved and no paid API was used.

- **I.2 External human data †** — all external human datasets that informed the approach anywhere (training/fine-tuning/retrieval/ICL/calibration):
  Nothing entered a model's context or weights. Four external datasets informed **calibration
  constants and design choices only**, all published and all unrelated to this study's outcomes:

  1. **Voelkel et al. (2026), Climate Change Challenge (CCC)** — 12,757 respondents, 9 arms. Source
     of 5 of the 8 level anchors, 6 party-gap anchors, and the out-of-sample fit of the party blend
     weight. Two of its items are *verbatim identical* to items in this study.
  2. **Goldwert et al., climate-advocacy megastudy** — 19,141 respondents, 11 arms. Validation only.
  3. **ICPC / Vlasceanu et al., international climate megastudy (US subsample)** — 8,253 respondents,
     12 arms. Validation only.
  4. **Voelkel et al., democratic-norms megastudy** — 12,501 respondents, 7 arms. Validation only.

  Plus **TISP** (Trust in Science and Science-Related Populism), a published multi-country survey
  asking the same trust battery: source of 3 level anchors and the dispersion target. And published
  **US Census** margins for the quota draw (D.1).

  All four megastudies concern climate or democratic attitudes and **none reports outcomes from this
  study**. They were used to answer "how much should predicted effects be shrunk, and where do
  response levels sit", never "what is the answer here".

- **I.3 Blinding attestation ★** — **mandatory.** Signed attestation that no team member accessed, solicited, or was shown any human outcome data from this study, including pilots, before the prediction lock:
  No team member has accessed, solicited, or been shown any human outcome data from this
  study, including pilot data, at any point before the prediction lock. The team's only inputs from
  the study are the publicly distributed materials: the questionnaire, the codebook, the intervention
  texts and the preregistration. No correction, constant or design choice in this submission was
  fitted on any outcome from this study. — Signed: Yannik Keller, Max Planck Institute for Human Development, 2026-08-29.

- **I.4 Contamination note †** — training cutoff of every model vs public release dates of this project's materials; note any known exposure:
  The materials that exist publicly are the instrument, codebook and preregistration; **no outcome
  data exist publicly**, so there is nothing about the results for a model to have memorized, whatever
  its cutoff.

  Residual exposure risk is to the *stimuli*: the four external validation studies (I.2) are published
  with their results and plausibly sit in every model's pretraining data. This cuts against us rather
  than for us, it would make our validation scores optimistic relative to true out-of-sample
  performance, and we have not corrected for it. Model training cutoffs are as published by each
  provider and are not independently verifiable by us; none is later than the materials' release.

## J · Internal selection procedure

- **J.1 Design-space search †** — how the final pipeline was chosen: how many configurations tried, internal validation criterion, what data it ran against:
  **Who searched.** The LLM agent of A.1, not a person. It wrote the search, ran it, read the
  results and chose the final configuration. The human authors chose the base models and cross-validation
  studies. The human requirement was that a cross-validation exist and be nested;
  every choice inside it was the agent's.

  **Criterion.** Pooled Pearson *r* between predicted and human intervention effects — the
  leaderboard's own sort key — computed on external studies, never on this one.

  **Data.** The four megastudies in I.2. Each was sampled through the same pipeline with the same
  models, so a design could be scored against real human effects.

  **Procedure.** Nested leave-one-study-out: for each held-out study every free choice — which runs
  to average, both shrinkage factors, which run supplies which structural term — is fitted on the
  other three alone, and the assembled recipe is scored once on the study it never saw. Roughly 80
  configurations were scored this way (7 candidate ensembles × 11 shrinkage values per fold, plus a
  dozen fixed-structure variants). Results were additionally averaged over eight random half-splits
  of each study's respondents, because a single split reorders variants that differ by less than its
  noise — one earlier conclusion of this project reversed when averaged.

  **A specific risk this creates, stated because the item exists to surface it.** An agent that both
  proposes a design and evaluates it can talk itself into a result. Three things were done about it,
  none of them complete: the fitting is nested so no held-out score can reach a fitted parameter; the
  code, findings and constants were audited by independent subagent instances instructed to *refute*
  rather than confirm, which overturned five of twenty-four claims and corrected the magnitude of
  most of the rest; and the whole audit is written up with its failures in
  `docs/reports/audit_findings.md` rather than only its conclusions. A reader who distrusts
  agent-authored self-evaluation should weight that report over this form.

  **Honest caveats, since this item exists to surface them.** (i) The within-outcome shrinkage value of 0.5 was chosen once, globally, on these same
  studies; its selection optimism is measured at 0.005 in *r*. (ii) The choice of which models to
  average is made by a rule fixed in advance ("average every run whose training correlation is
  positive"), instantiated per fold, rather than by reading fold scores.

## K · Reproducibility & frozen artifacts

- **K.1 Code & materials** — link/DOI, secrets removed, determinism/seeds documented (also record the link in `metadata.json` → `code_repository` / `code_doi`):
  **All of the code in this repository was written by the LLM agent of A.1.** The commit history is
  the record: 116 of 121 commits authored by `Claude Code`, the remainder being the initial scaffold.

  Code: <https://github.com/center-for-humans-and-machines/silicon_sampling> — the complete pipeline,
  including profile construction, instrument rendering, the sampler, the calibration layer, the
  validation harness and the reports. No credentials or secrets are present; nothing in the pipeline
  requires an API key. Seeds and determinism are documented in F.1.
  `code_doi`: 10.5281/zenodo.22160401  

- **K.2 Raw output logs †** — complete unprocessed model responses archived, hashed, time-stamped (required for Tiers 1–2, public or escrowed; Tier 3 where intermediate generations exist; oversized logs may be a separate linked Zenodo upload):
  The raw per-respondent export for the run supplying the submitted rows is in `raw_data_deposit/`
  and ships inside this release. The complete unprocessed generations for **all** runs are at https://doi.org/10.5281/zenodo.22160304

- **K.3 Computational resources** — API-call counts, total tokens, cost, compute time:
  **No API calls and no API cost** — all inference is local on open weights.

  Across the ten runs with recorded metadata (the eleventh, `muse_glimmer_30b`, was sampled on the
  cluster and its metadata file was not retained):

  | | |
  | --- | --- |
  | generation calls | 10,411,113 |
  | draws generated (incl. rejected) | 41,606,112 |
  | draws rejected and re-drawn | 1,175,625 (2.83%) |
  | structured-output fallbacks | 9,585 (0.092% of calls) |
  | GPU-hours recorded | ≈ 78 |

  **The GPU-hour figure is a lower bound**: resumed runs record only their final chunk, and the run
  without metadata is not counted. Hardware was a single local RTX 4090 and an institutional
  multi-GPU Slurm partition. Monetary cost is institutional compute time, not billed per run

## L · Disclosure class

Each item above is deposited as **public**, **escrowed** (sealed from the public but available to the
core team and auditors under confidentiality, with a public SHA-256 hash + timestamp so the lock is
still verifiable — an embargo with a sunset date is encouraged), or **withheld** (permitted only for
items marked neither ★ nor †). Your entry's class is set by its **most restricted item** and recorded
in `metadata.json` → `disclosure_class` (and `escrow_doi` if anything is escrowed):

- **A · Open** — all items public. Full results-table standing; all features enter the design-choice analysis.
- **B · Escrowed** — some items sealed but every item is available to the core team/auditors under confidentiality. Full standing with an *escrowed* badge; only publicly disclosed features enter the design-choice analysis.
- **C · Sealed** — one or more permitted items withheld even from escrow. Scored and reported with a *not independently verifiable* flag; excluded from the approach catalogue and design-choice analysis.

**This entry is class A · Open.** Every item above is public: the code repository, the prompts, the
profile construction, the calibration constants, the design-space search **and the fact that an LLM
agent authored all of them** are disclosed in full, and nothing is escrowed or withheld. `escrow_doi` is `null`. The one item not shipped *inside*
the deposit — the complete raw generation logs (K.2) — is withheld for size, not for secrecy, and is
available on request; if the core team prefers it escrowed rather than on-request, that changes
nothing else in this form.

★ items must always be public (never escrowed or withheld); † items must be at minimum escrowed. Full
policy: <https://janpfander.github.io/llm_predictions_megastudy/#disclosure>
