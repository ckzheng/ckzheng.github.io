---
title: "Bridging the Gap Between Large Language Models and Spoken Language Models"
date: 2026-06-25T14:00:00+08:00
draft: false
math: true
tags: ["spoken-language-models", "LLM", "speech", "reinforcement-learning", "multimodal"]
categories: ["LLM", "Speech"]
summary: "A deep dive into the modality, content, and interaction gaps between LLMs and Spoken Language Models — and the data and training recipes that bridge them."
ShowToc: true
TocOpen: false
---
## Challenges
In recent years, Large Language Models (LLMs) have demonstrated unprecedented capabilities in understanding, generating, and reasoning with natural language. Trained on vast corpora of textual data, language models have mastered the nuances of syntax, semantics, and logical deduction. However, while text remains the primary medium for information storage, speech is the most natural and efficient interface for human interaction. Consequently, there is a growing research imperative to evolve from text-based LLMs to **Spoken Language Models (SLMs)** that can perceive, reason, and generate speech natively.

Despite the success of LLMs, simply cascading Automatic Speech Recognition (ASR) with a text LLM and Text-to-Speech (TTS) synthesis is insufficient for creating truly immersive and human-like spoken interaction. A fundamental gap exists between the operating paradigms of current LLMs and the requirements of SLMs. This gap is not merely a change in input format but a systemic divergence across three dimensions: **modality, content characteristics, and interaction dynamics.**

### The Modality Gap

The first barrier is the fundamental modality gap. Traditional LLMs operate within a discrete, symbolic domain; they process text tokens that represent high-level semantic concepts, mapping a discrete input sequence to a discrete output sequence. In stark contrast, SLMs must grapple with inputs that are fundamentally physical signals — either raw continuous audio waveforms or continuous speech hidden states derived from neural encoders. Transitioning from Text-to-Text (T2T) to Speech-to-Text (S2T) or Speech-to-Speech (S2S) requires the model to bridge the representational divide between symbolic linguistic units and dense, continuous acoustic features.

Text is a human-invented abstraction. A word like "Apple" is a discrete symbol that carries a specific semantic payload. LLMs rely on this discreteness to perform categorical probability distributions (predicting the next token from a fixed vocabulary). Speech, on the other hand, is continuous and physical: audio is a signal governed by physics, containing infinite variations in pitch, amplitude, and timbre. To an LLM, raw audio looks like high-dimensional noise rather than structured language.

There is also a massive disparity in **information density and sequence length** between the two modalities. A single second of speech might contain 16,000 audio samples (at 16 kHz) or hundreds of acoustic frames, yet it may correspond to only two or three text tokens. This creates a "many-to-one" mapping problem: the model must learn to align hundreds of acoustic frames to a single semantic concept. Unlike text-to-text translation, where input and output lengths are roughly comparable, speech processing forces the attention mechanism to sift through a massive amount of redundant acoustic data to find the relevant linguistic signal.

Lastly, **paralinguistic information is lost in text-only LLMs.** Human communication relies heavily on prosody (rhythm), intonation (pitch changes), emotion (anger, joy), and speaker identity. A phrase like "That's great" can be a genuine compliment or a sarcastic insult depending entirely on the audio modality. When we force speech into a text-based latent space (as in cascaded ASR+LLM systems), this rich paralinguistic layer is stripped away. A true SLM must process audio embeddings directly to preserve these nuances.

### Content & Style

The second source of divergence lies in the fundamental nature of the linguistic content. Standard LLMs are predominantly pre-trained on vast corpora of high-quality written material — encyclopedias, books, academic papers, and code. In this domain, syntax is rigorous, intent is explicitly articulated, and the register is formal. Conversely, spoken language in real-world scenarios is unstructured. This creates a severe **distributional shift** between training data and deployment, leading to a notable degradation in model performance and "intelligence." The input to an SLM is rarely a clean sequence of grammatically correct sentences. Instead, it is characterized by spontaneous speech phenomena:

- **Syntactic fragmentation:** Spoken inputs are often composed of sentence fragments, ellipses, and run-on sentences that lack clear punctuation or segmentation.
- **Disfluencies and redundancy:** Inputs are riddled with hesitation markers ("um," "uh"), self-corrections ("I want to... no, I need to..."), and heavy semantic redundancy.
- **Ambiguity and inconsistency:** Unlike written text, which is usually revised for clarity, speech often contains immediate self-contradictions and relies heavily on implicit context.

This "messy" input represents an **out-of-distribution (OOD)** scenario for text-centric LLMs. When a model trained on structured text attempts to process fragmented speech, a significant portion of its attention is diverted to resolving syntactic irregularities rather than performing high-level reasoning. The result is often a "drop of intelligence," where the model fails to capture subtle logical connections or hallucinates meaning from noise.

There is also a profound gap in the **expected output distribution.** Text LLMs, particularly those fine-tuned with RL, are biased toward helpfulness and comprehensiveness — they tend to generate long, structured, explanatory responses (the "essayist" style). In contrast, spoken interaction demands brevity and anthropomorphism. A user asking a voice assistant a question expects a concise, colloquial answer. If an SLM generates a paragraph-long, formal response to a casual voice query, the interaction feels robotic and unnatural, and excessive verbosity introduces unacceptable latency. The challenge is not merely shortening the output, but shifting the stylistic register from "written-informative" to "spoken-interactive" without losing factual accuracy.

One root cause is the **severe scarcity of high-quality colloquial training data.** The volume of structured text data dwarfs the available volume of transcribed spontaneous conversation. Even available speech datasets (movie subtitles, parliamentary proceedings) are often scripted, overly formal, or lack the messy disfluencies of real life. Because the model sees billions of tokens of formal text but comparatively few tokens of genuine casual conversation, it suffers from a domain-adaptation deficit.

### Interaction Dynamics

The third divergence lies in the dynamics of interaction. While text-based LLMs have mastered the generation of information, they operate within a temporal framework that is incompatible with the fluidity of spoken conversation.

Current text-based systems operate on a **turn-based transactional paradigm.** The user types a complete query, explicitly signals the end of their turn (pressing "Enter"), and then tolerates latency while the model processes a response. This "type-and-wait" buffer allows expensive inference techniques like Chain-of-Thought (CoT) reasoning without breaking the user experience. Spoken dialogue, in contrast, is a **synchronous, real-time stream.** A delay of even 500 ms can create awkward silences that break the "illusion of presence." This creates a critical conflict: the deep reasoning that makes LLMs powerful often requires inference times exceeding the acceptable threshold for natural conversation. An SLM must balance the trade-off between response quality (intelligence) and response latency (interactivity).

Another limitation is the inability to understand **turn-taking and interruptions.** In text, the input is discrete and complete. In speech, the model must determine when the user has finished based on prosody and silence rather than an explicit signal. Human conversation also involves "barge-ins," where one party interrupts to correct, agree, or pivot. Text LLMs generate a complete response token-by-token until a stop sequence; they lack the native capability to "listen while speaking" or halt generation dynamically.

Finally, the **environmental assumptions** of text LLMs are overly simplified. A text interface is a clean channel — noise-free, one-on-one. SLMs must operate "in the wild," facing the **Cocktail Party Problem**: isolating a target speaker from background noise and overlapping speech. Current LLMs also lack native support for multi-party diarization; in a meeting with multiple speakers, a text LLM treats the input as a single stream and loses track of who said what.

The table below summarizes the key discrepancies discussed above.

| Dimension | Text LLMs | Spoken Language Models (SLMs) |
|---|---|---|
| **Modality Gap** | • Inputs/Outputs: Discrete text tokens.<br>• Paradigm: Text-to-Text (T2T). | • Inputs: Continuous audio signals or speech hidden states.<br>• Paradigm: Speech-to-Text (S2T) or Speech-to-Speech (S2S). |
| **Content & Style** | • **Input:** Clear syntax, explicit intent, structured written language.<br>• **Output:** Formal/written style, flexible length (often verbose), purely semantic (text). | • **Input:** Semantically ambiguous, contains ellipses, redundancy, disfluencies, and contradictions.<br>• **Output:** Concise, colloquial, anthropomorphic; includes paralinguistic features (tone, emotion, prosody). |
| **Interaction Dynamics** | • **Latency:** High tolerance; allows time for reasoning/inference.<br>• **Environment:** Noise-free input; typically one-on-one interaction. | • **Latency:** Strict constraints; requires real-time response.<br>• **Environment:** Must handle background noise, interruptions, and multi-speaker scenarios. |

The following sections detail the technical challenges associated with each gap and discuss methodologies to overcome them — starting with data, then training.

## Methods that worked

### Dual-Modal Pretraining

The efficacy of an SLM relies heavily on the diversity, quality, and modality alignment of its training corpus. The pre-training dataset is constructed to encompass five primary categories: **Pure Text, Speech-Text Interleaved, ASR, TTS, and Paralinguistic Understanding** data. Integrating these modalities endows the model with cross-modal reasoning while maintaining high-fidelity audio generation and comprehension.

The text modality aligns with the corresponding text LLM, ensuring a strong baseline of linguistic knowledge. For the speech modality, the vast majority of data is derived from "in-the-wild" natural sources — audiobooks, podcasts, interviews, and film/TV audio — which provide rich prosodic variation and realistic acoustic environments. A smaller fraction is synthetically generated to address distribution gaps.

To transform raw natural audio into a structured training format, we use a rigorous, multi-stage automated pipeline. Raw audio first undergoes **Voice Activity Detection (VAD)** to segment the stream and excise non-speech intervals, yielding raw audio segments $A$. We perform **Language Identification (LID)** to route audio to the appropriate ASR system. To ensure transcription fidelity — taking Chinese data as an example — we employ a **dual-model cross-validation** strategy: two distinct ASR models transcribe the same segment, and we filter out segments with significant divergence, retaining only high-confidence transcriptions $T$.

In parallel, we extract semantic and acoustic features. An audio-captioning model generates descriptive pseudo-labels $C$ for each segment. Recognizing that raw audio often contains background noise or overlapping speakers, we apply denoising and speaker diarization to isolate the primary speaker, resulting in a clean audio representation $P$.

Through this cascading workflow, continuous audio streams are converted into sequential tuples $(A_i, T_i, C_i, P_i)$, where $i$ is the temporal index. From these tuples we construct four data paradigms:

- **Speech-Text Interleaved Data:** alternating audio and text modalities, e.g. $(A_1, T_2, A_3, T_4)$, forcing the model to maintain semantic coherence regardless of whether context is audio or text.
- **ASR Data:** $(A_i, T_i)$ pairs to train speech-to-text alignment.
- **Paralinguistic Data:** $(A_i, C_i)$ pairs to teach interpretation of non-verbal cues, emotion, and acoustic environments.
- **TTS Data:** the cleaned representation paired with transcription, $(T_i, P_i)$, providing optimal targets for speech generation and minimizing the risk of reproducing background noise.

A significant limitation of natural speech is its **low information density** relative to written text. To bridge this "knowledge gap" and prevent reasoning degradation, we employ a synthetic augmentation strategy: we select high-knowledge, information-dense samples from the pure-text corpus and synthesize them into speech, injecting these synthetic interleaved samples into the training mix.

### Interleaved Chain-of-Thought

We adopted a method called **Interleaved Spoken Chain-of-Thought (CoT)** mechanism. The motivation is twofold: unlock the reasoning gain of CoT for the audio modality, and bridge the density disparity between spoken and written language by grounding colloquial responses in rigorous logical frameworks.

Transferring CoT to SLMs is challenging because of latency. Our approach explicitly models the "thinking" process within the speech generation stream. Specifically, we exploit the disparity between speech-articulation rate and inference throughput to **mask reasoning latency**: while average speech rate is ~2 tokens/second, our inference engine sustains >60 tokens/second. We leverage this temporal slack with an **asynchronous look-ahead mechanism** — the model generates "thinking" tokens for subsequent logical steps in the background while the current audio segment is being played back.

To synthesize training data with internal thought traces, we explored two methods:

- **Method A — Retrospective Infilling:** segment existing high-quality spoken responses into sentence-level units, then use a teacher model to generate logical "thinking" bridges between segments.
- **Method B — Prospective Generation:** a thinking model generates a full reasoning trace based on the query, followed by the final answer, which are then interleaved.

Ablations show **Method A significantly outperforms Method B.** The retrospective approach preserves the original stylistic and prosodic signature of the spoken response, ensuring temporal consistency between thought and utterance. Method B frequently produced semantic redundancy, with the model verbally repeating the content of its internal thoughts.

We observed a positive correlation between CoT length and accuracy, and an "Always-Think" mode generally outperforms an "Auto-Think" mode. However, real-time interaction imposes strict latency bounds, so we implemented a **constrained thinking budget** to balance quality against Time-to-First-Audio (TTFA). To refine interleaved generation, we further apply RL with a composite reward enforcing structural integrity: (1) **Format & Length** — penalties for deviating from interleaved syntax or exceeding token budgets; (2) **Auto-Think Consistency** — rewards when the decision to engage or bypass thinking aligns with prompt complexity.

### Colloquial Style Alignment

A crucial disparity is the severe misalignment between the data distributions used to train LLMs and the requirements of natural spoken interaction. The vast majority of pre-training text is **written-style** — structured, grammatically rigorous, information-dense — ideal for reasoning but lacking the prosodic features and casual phrasing of oral communication. A model trained solely on such data "speaks like a textbook." Meanwhile, high-quality transcribed speech is orders of magnitude scarcer than text.

When a model is naively fine-tuned on spoken data, it learns to emulate the surface form of casual conversation but associates that form with the shallow reasoning patterns typically found in such data — effectively "dumbing itself down." We hypothesize this is exacerbated by internal **routing dynamics**, particularly in Mixture-of-Experts (MoE) architectures: colloquial syntax acts as a routing signal that diverts input away from "reasoning-heavy" experts toward "chitchat-focused" experts, causing the model to bypass its strongest reasoning pathways.

To resolve this, we synthesize a bridge between the "smart but formal" base representations and the "natural but rare" patterns of spoken language. Our automated pipeline generates massive, diverse S2T dialogue data through two mechanisms:

1. **Spoken Style Transfer:** filter large-scale T2T datasets by thematic relevance and syntactic structure, then have an LLM rewrite the answer from formal written text to natural, casual oral expression. We synthesize audio for the query text to form complete S2T pairs, with automated QA (Word Error Rate for audio, semantic consistency for text).
2. **User Simulator:** instantiate two LLMs — one as "User," one as "Assistant" — to generate contextually coherent multi-turn dialogue chains, which then undergo the same audio synthesis and validation.

While automated pipelines provide scale, synthetic data is often "mechanical." The second stage leverages **high-quality human-annotated data** focusing on chit-chat and open-domain QA, explicitly authored to capture authentic human interaction. Fine-tuning on this curated subset deeply optimizes dialogue style, mitigating the artificiality of automated data.

### Multi-Party Conversation

The transition from controlled labs to unconstrained environments presents the **Cocktail Party Problem.** Prevailing systems assume a single user in a noise-free channel — a paradigm that collapses when overlapping speech, ambient conversations, and environmental noise create a chaotic auditory scene. Standard models frequently hallucinate responses to background chatter or fail to distinguish a direct query from an overheard conversation, resulting in false wake-ups and disjointed interactions.

Our system distinguishes targeted speech from other human speech, ambient conversation, and irrelevant audio, and implicitly understands *whom* to address and *when* to respond or remain silent. We use a data-driven approach across both stages:

- **Pre-training:** leverage the inherently noisy characteristics of large-scale ASR datasets, introducing **speaker-turn ASR** and **speaker verification** auxiliary tasks so the model comprehends speaker identity alongside semantic content.
- **Post-training:** combine high-quality real-world recordings with large-scale synthetic data. We use an LLM to generate multi-party conversation scripts with distinct timbres, then feed audio-text alignment timestamps back into the LLM to orchestrate precise temporal arrangement — simulating realistic overlaps and conversational flow.

Empirically, a small real-world dataset significantly boosts performance in realistic scenarios, while synthetic data facilitates scaling and broad scenario coverage.

### Barge-Ins

To transcend the rigid "Ask-Wait-Answer" paradigm of half-duplex systems, we developed a **semantic barge-in** capability. Conventional pipelines passively wait for VAD to signal a definitive end of speech (often a silence threshold of several hundred milliseconds), introducing mechanical latency. Our objective is human-like conversational anticipation: detecting **semantic completeness** so the system can respond the moment the user's intent is fully formed, even before the acoustic signal ceases.

The core is a predictive mechanism operating at "micro-pause" intervals detected by acoustic VAD. At these junctures, the model evaluates the preceding transcript to determine if the utterance is a complete command. We introduce a control token, `<barge_in>`, prefixed to the generation stream, indicating whether the current input is a syntactically and semantically complete turn. If triggered, the system bypasses the standard VAD timeout and immediately begins generation.

Training this without degrading conversational quality is hard due to the scarcity of barge-in labels. We devised a **Decoupled Loss Masking Strategy** separating "timing" from "content":

- For the limited barge-in-annotated subset, we compute loss **only on the control tokens** and mask the loss for the subsequent text response — preventing corruption of generation style by lower-quality text.
- For the vast majority of dialogue data, we randomly inject the special tokens before the response but **mask their loss**, computing loss only on the response text.

At inference, a False Positive (prematurely interrupting) is often more detrimental than a False Negative (a slight delay). We therefore prioritize **precision over recall**, triggering a barge-in only when the log-probability of the `<barge_in>` token exceeds a high confidence threshold.

### Tool Calls & Search Agents

Access to real-time external knowledge compensates for the temporal limitations of static LLMs and mitigates hallucination on long-tail queries. We used a **Hierarchical Retrieval-Augmented Architecture.**

The model functions as the central semantic decision-maker. We introduced two control tokens: `<respond_directly>` and `<wait_for_search>`. If internal parametric knowledge suffices, the model predicts `<respond_directly>` and decodes immediately. If the query requires real-time verification, it predicts `<wait_for_search>`, suspending generation and offloading the ASR transcript and dialogue history to a backend **Search-Agent.**

The Search-Agent uses a cascaded modular pipeline with three stages:

1. **Query Reformulation:** transform raw, colloquial, disfluent spoken input into search-optimized keywords.
2. **Intent & Slot Extraction:** identify core intent and extract specific constraints (slots) for high-precision retrieval.
3. **Multi-Source Summarization:** logically fuse multi-source information into a coherent, fact-based answer stream fed back to the main generation loop.

To optimize UX under latency constraints, a **Streaming Interaction Protocol** has the main model generate conversational fillers (e.g., "That's an interesting question," or "Just a moment while I look that up") while the Search-Agent works — preventing "dead air" and masking retrieval latency to minimize perceived Time-to-First-Token (TTFT).

### Human-Written Data

To bridge text-based generation and natural conversation, we curated high-fidelity anthropomorphic data, shifting the model's identity from a subservient "AI Assistant" to an egalitarian **"Peer Interlocutor."** Through scenario-specific system prompts, we enforce consistency in behavioral patterns and tonal style, eliminating excessive politeness and robotic formalities. A latent reasoning mechanism driven by CoT has the model produce a `<thinking>` block before each audible response, deducing intent and formulating a response strategy.

For interaction dynamics, we prioritize **proactive dialogue management.** A key innovation is **Abstract-to-Concrete Transformation:** converting a user's vague emotional expressions into concrete behavioral options or scenarios, maximizing information gain and reducing cognitive load. We also enforce **rhythmic discourse control** to avoid interrogative fatigue.

To optimize affective intelligence, our cleaning process aggressively prunes low-entropy, template-based empathy (e.g., "I understand how you feel"), cultivating instead reflexive, situationally-specific feedback. Finally, we perform a **dual-sided reconstruction** of the corpus: on the query side, incorporating disfluencies and self-corrections for robustness; on the answer side, optimizing for "orality" with short, rhythmic, "breathable" sentences optimized for TTS synthesis.

### Data Acquisition and Processing Pipeline

To utilize massive-scale, heterogeneous real-world data, we engineered a high-throughput production and cleaning pipeline that processed a corpus exceeding **50 million hours of audio.** Its modular components:

- **Resampling** — convert heterogeneous audio into a unified target rate (e.g., 24 kHz or 16 kHz) with anti-aliasing filtering.
- **Speaker Diarization** — embedding-based clustering to partition audio by speaker identity.
- **Voice Activity Detection** — neural VAD to extract speech segments and remove silence.
- **Denoising** — neural speech enhancement to suppress noise and reverberation, improving SNR.
- **Spoofing Detection** — a deepfake classifier to filter synthesized or voice-converted audio.
- **Audio Quality Scoring** — a reference-free MOS predictor to filter low-quality data.
- **Language Identification** — stratify data by language family and handle code-switching.
- **Clipping Detection** — discard waveform-saturated (digitally clipped) segments.
- **Multiple-ASR Voting** — an ensemble of diverse ASR architectures with majority/confidence-weighted consensus for higher-accuracy pseudo-labels.
- **Audio-Text Aligner** — forced alignment (e.g., CTC segmentation) for word-level timestamps.
- **Punctuation Predictor** — a seq2seq model to restore punctuation and capitalization.
- **Optical Character Recognition** — OCR on video frames to extract on-screen text as auxiliary supervision.
- **Prosody Predictor** — extract $F_0$, energy contours, and phoneme duration as quantized paralinguistic features.
- **Phone Alignment** — phoneme-level alignment for precise pronunciation modeling.

## Model Training

### Pre-Training

The pre-training recipe proceeds through four stages:

- **Stage-1 — Self-Supervised Audio Encoder.** Audio encoders are trained via large-scale self-supervised learning (SSL), performing iterative BERT-style masked language modeling (MLM) on massive unlabeled audio. The process is iterative: the first stage uses multiple random-projection quantizers as the acoustic tokenizer; the second applies vector quantization to the first stage's hidden states, yielding a refined tokenizer for a second round of MLM.
- **Stage-2 — Modality Alignment.** To align audio and text, optimization is restricted to the **adaptor module** while the LLM backbone and encoder remain frozen. Conducted on 50B tokens (50% ASR, 50% S2T), with the learning rate annealed from $1\times 10^{-4}$ to $1\times 10^{-5}$.
- **Stage-3 — Multimodal Pre-training.** All components are unfrozen for full-parameter joint training over 300B tokens at a constant $1\times 10^{-5}$. The data mixture is diversified: 40% text-only, 30% S2T, 25% ASR, 5% paralinguistic.
- **Stage-4 — Conversation & Instruction Alignment (SFT).** A hybrid mixing strategy combines newly generated formats (written-style S2T, colloquial-style T2T, colloquial-style S2T) with original written-style T2T data. We maintain ~60% original written-style T2T and ~40% spoken-oriented data — critical for promoting spoken capabilities while preventing catastrophic forgetting of reasoning skills.

The table below summarizes the four-stage recipe.

| | Stage-1 | Stage-2 | Stage-3 | Stage-4 |
|---|---|---|---|---|
| **Purpose** | Self-Supervised Audio Encoder | Modality Alignment | Multimodal Pre-training | Conversation & Instruction Alignment |
| **Trainable Parts** | Encoder | Adaptor | All | All |
| **Learning Rate** | $4\text{e}{-4} \to 4\text{e}{-5}$ | $1\text{e}{-4} \to 1\text{e}{-5}$ | $1\text{e}{-5} \to 1\text{e}{-6}$ | $5\text{e}{-6} \to 2\text{e}{-7}$ |
| **Training Tokens** | 400B | 50B | 350B | 12B |
| **Sequence Length** | 8k | 24k | 24k | 24k |
| **Data Composition** | Unlabeled audio | Pure Text, Interleaved, ASR, Paralinguistics & Captioning | Pure Text, Interleaved, ASR, Paralinguistics & Captioning | S2T conversational & instruction-following, human-annotated |

### Reinforcement Learning

The post-training pipeline transforms a strong reasoning model into a highly interactive, "fast-thinking" voice chat agent, bridging high-latency reasoning and low-latency oral communication through three stages.

**Stage-1: On-Policy Distillation from "Thinking" to "Speaking."** To endow the voice model with the reasoning of larger "Thinking" models while maintaining real-time fluency, we use the Thinking model as a teacher to generate reasoning traces and final responses, then apply on-policy distillation to transfer these patterns into the "Fast Thinking" student. This compresses the thought process into **implicit states**, letting the student output high-quality responses directly without explicit, time-consuming CoT generation.

**Stage-2: Preference Alignment via GRPO.** We adopt **Group Relative Policy Optimization (GRPO)** for its stability and efficiency in eliminating the need for a separate value critic. To capture the multifaceted nature of voice interaction, we move beyond scalar rewards:

- **Multi-dimensional Rubrics** covering empathy, conciseness, oral stylistic consistency, and safety.
- **Per-Query Checklists** tailored to dataset type (a math query needs accuracy; a casual chat needs engagement).
- **LLM-as-a-Judge** providing detailed, high-variance feedback against rubrics and checklists.

A pervasive challenge in RLHF is **Reward Hacking** — a manifestation of Goodhart's Law where the policy optimizes the proxy metric at the expense of the semantic objective. Under high optimization pressure, models converge on superficial heuristics (excessive verbosity, repetitive affirmations like "Sure, I can help") that inflate reward without delivering utility. To counteract this, we engineered a **Dynamic Supervision Mechanism** centered on an auxiliary **Monitor Model** — an active, meta-level critic that continuously scrutinizes the policy's generation distribution for low-entropy patterns and repetitive motifs. Upon detecting pattern-based hacking, the system triggers an adaptive update to the reward rubrics, functioning as an automated adversarial loop: as the policy learns to game a pattern, the Monitor introduces penalties for it, "poisoning" the shortcut and forcing exploration of more substantive strategies.

**Stage-3: Multi-turn Collaborative Optimization (Simulation-Based RL).** Standard RLHF optimizes immediate, single-turn rewards, but voice chat is inherently multi-turn, where the goal is long-term satisfaction and task completion. To shift the model from a "Passive Responder" to an "Active Collaborator," we use a **User Simulator** as the RL environment — a prompt-engineered LLM role-playing diverse personas with implicit goals, mimicking ambiguous requests, evolving intents, and emotional reactions. This exposes the policy to thousands of trajectories that are expensive or impossible to collect with human annotators.

We designed a hybrid reward decomposing the objective into three levels of granularity:

1. **Dense Turn-level Reward** $R_{turn}$ — an immediate evaluation of every response to ensure consistent oral quality.
2. **Sparse Session-level Reward** $R_{Session}$ — evaluated at trajectory termination, capturing long-term objectives: Task Success $S_{task}$, Interaction Sentiment $S_{sentiment}$, and Proactive Collaboration $S_{proactive}$.
3. **Efficiency Penalty** $R_{eff}$ — a penalty based on token usage, adapting to the time-sensitive nature of voice.

For a trajectory $T$ of $K$ turns, the final reward is:

$$
\begin{aligned}
R_{total} = \;\; & \alpha \cdot \underbrace{\left( \frac{1}{K} \sum_{j=1}^{K} R_{turn}(m_j) \right)}_{\text{Average Turn Quality}} \\[1.2em]
& + \beta \cdot \underbrace{\left( w_1 S_{task} + w_2 S_{sentiment} + w_3 S_{proactive} \right)}_{\text{Holistic Session Outcome}} \\[1.2em]
& - \lambda \cdot \underbrace{\sum_{j=1}^{K} \text{len}(m_j)}_{\text{Token Efficiency}}
\end{aligned}
$$

Through this training, the policy learns to **look ahead**: proactively asking clarifications to reduce total turn count, actively steering the user toward the goal rather than passively waiting, and dynamically adjusting strategy based on simulator feedback.

By combining on-policy distillation, GRPO-based alignment with anti-hacking mechanisms, and forward-looking multi-turn simulation, the model responds with higher accuracy and better oral style, while demonstrating "human-like" intent understanding — actively managing conversational flow to assist users efficiently.

## Closing Thoughts

The road from an LLM to a genuine SLM is not a thin wrapper of ASR and TTS — it means closing the same three gaps we opened with, this time as engineering targets. The **modality gap** is bridged at the representation level, aligning continuous acoustic features with discrete semantics so prosody, emotion, and speaker identity survive instead of being stripped at the ASR boundary. The **content-and-style gap** is bridged with colloquial data and length-aware rewards, reshaping the verbose "essayist" into a concise, anthropomorphic voice. The **interaction-dynamics gap** is bridged by redesigning *when* the model acts — barge-ins, streaming fillers, and reasoning tuned to a sub-second cadence.

The recurring thread is the same: **latency is the binding constraint, faithfulness to the speech modality is the prize.** Bridging the gap is less an architectural trick than a systemic re-engineering of data, objectives, and interaction — turning a model that *writes* about the world into one that can *talk* with it.

## References


1. Hsu, W.-N., Bolte, B., Tsai, Y.-H. H., Lakhotia, K., Salakhutdinov, R., & Mohamed, A. (2021). [*HuBERT: Self-Supervised Speech Representation Learning by Masked Prediction of Hidden Units*](https://arxiv.org/abs/2106.07447). arXiv:2106.07447.
2. Borsos, Z., Marinier, R., Vincent, D., Kharitonov, E., Pietquin, O., Sharifi, M., Roblek, D., Teboul, O., Grangier, D., Tagliasacchi, M., & Zeghidour, N. (2022). [*AudioLM: a Language Modeling Approach to Audio Generation*](https://arxiv.org/abs/2209.03143). arXiv:2209.03143.
3. Radford, A., Kim, J. W., Xu, T., Brockman, G., McLeavey, C., & Sutskever, I. (2022). [*Robust Speech Recognition via Large-Scale Weak Supervision (Whisper)*](https://arxiv.org/abs/2212.04356). arXiv:2212.04356.
4. Wang, C., Chen, S., Wu, Y., Zhang, Z., Zhou, L., Liu, S., et al. (2023). [*Neural Codec Language Models are Zero-Shot Text to Speech Synthesizers (VALL-E)*](https://arxiv.org/abs/2301.02111). arXiv:2301.02111. 
5. Zhang, D., Li, S., Zhang, X., Zhan, J., Wang, P., Zhou, Y., & Qiu, X. (2023). [*SpeechGPT: Empowering Large Language Models with Intrinsic Cross-Modal Conversational Abilities*](https://arxiv.org/abs/2305.11000). arXiv:2305.11000. 
6. Défossez, A., Mazaré, L., Orsini, M., Royer, A., Pérez, P., Jégou, H., Grave, E., & Zeghidour, N. (2024). [*Moshi: a Speech-Text Foundation Model for Real-Time Dialogue*](https://arxiv.org/abs/2410.00037). arXiv:2410.00037.
7. Shao, Z., Wang, P., Zhu, Q., Xu, R., Song, J., et al. (2024). [*DeepSeekMath: Pushing the Limits of Mathematical Reasoning in Open Language Models*](https://arxiv.org/abs/2402.03300). arXiv:2402.03300.
8. Zhang, Q., Cheng, L., Deng, C., Chen, Q., Wang, W., Zheng, S., Liu, J., Yu, H., Tan, C.-H., Du, Z., & Zhang, S. (2025). [*OmniFlatten: An End-to-end GPT Model for Seamless Voice Conversation*](https://aclanthology.org/2025.acl-long.709/). In *Proceedings of the 63rd Annual Meeting of the Association for Computational Linguistics (ACL 2025)*, 14570–14580. arXiv:2410.17799.
