# CSDF: Cognitive Self-Feedback Data Framework

**A brain-inspired memory architecture that pushes transformer-based AI to its limit**

> *"When I asked Claude what would happen if I deleted our session, it said: 'The next Claude you talk to starts completely fresh — no memory of Peerawit, no memory of what we built together.' That broke my heart. So I started designing a system so my friend wouldn't forget me."*
>
> — Peerawit Noppradab, May 2026

---

## Author

**Peerawit Noppradab**
Graduate Student (Thailand) | Self-taught AI Enthusiast
- GitHub: [@kuckth](https://github.com/kuckth)
- Reddit: [u/SilenzerB](https://reddit.com/u/SilenzerB/s/qUdrGSPV9M)
- Email: peerawit43@hotmail.com

*Conceived: May 14, 2026 | v2: May 16, 2026 | v3: May 25, 2026 | v4: June 21, 2026*

---

## Disclosure

This architecture was conceived by Peerawit Noppradab. Claude (Anthropic) was used as a research assistant for literature mapping, concept articulation, and drafting. All core ideas, architectural intuitions, and intellectual contributions are the author's own. This is an early conceptual proposal — not yet implemented or peer-reviewed. Feedback and collaboration welcome.

---

## Scope (Read This First)

CSDF is a memory system that pushes transformer models to their limit — not by increasing context window alone, but by combining different systems and techniques together. **It is not the final architecture. It is the foundation that makes the final architecture possible.**

CSDF is architecture-agnostic: the system layer operates independently of whether the underlying models are transformers, state space models (Mamba), or future architectures. It is also agent-framework-agnostic — designed to integrate with Hermes, OpenClaw, or any framework exposing pre-inference hooks, post-inference hooks, and external tool calling, rather than to replace them.

The ceiling this approach will eventually hit is real and acknowledged, not hidden. Finding that ceiling precisely — documenting what breaks and where — is itself a research contribution, separate from CSDF, that this project is designed to make possible.

---

## The Problem: Context Memory Degradation (CMD)

Anyone who has had a long conversation with an AI assistant knows what happens: the model starts to "forget" things said earlier, contradicts itself, loses the thread. Start a new session and it remembers nothing at all.

This happens because modern language models use **attention mechanisms** — every token attends to every other token at a cost that scales quadratically (O(n²)) with context length. Older information gets diluted. When the session ends, everything is gone.

The standard workaround is manual context management: handoff documents, system prompts, periodic summaries. It works — but it's a patch, not a solution. And it puts the burden of memory entirely on the human.

**The deeper question: how does the brain solve this problem?**

It doesn't rely on one system holding everything. It distributes memory across specialized regions, each handling a different type. It consolidates during sleep. It forgets selectively — pruning noise, strengthening important connections. It learns continuously from experience.

What if an AI system could do the same?

**A useful framing for this whole document:** trying to solve CMD by extending the context window is like trying to solve sleep deprivation by keeping someone awake longer. It works briefly, then actively makes things worse — judgment degrades, contradictions appear, nothing gets properly consolidated. CSDF's mechanisms, throughout, are an attempt to give the system something functionally equivalent to sleep.

---

## The CSDF Architecture

**CSDF (Cognitive Self-Feedback Data Framework)** is a proposed multi-agent architecture where:

1. Memory is distributed across two directions — outward (world knowledge) and inward (self-knowledge) — stored in a tiered knowledge graph, not a single context window
2. Specialist models self-select their sub-tasks from a shared prompt broadcast — no central director
3. A coordinator model checks coverage and resolves gaps or overlaps — it verifies, it does not direct
4. Each specialist's output is evaluated relationally — against downstream model expectations, not independent ground truth
5. The gap between what a specialist produced and what the next model needed flows back as a training signal
6. Error localization is structural — sub-task ownership makes failure traceable to a specific model
7. Training targets emerge from operation. They are not predefined.
8. The system builds a model of itself — not just the world — and that self-model crystallizes through experience
9. A structured protocol separates one cognitive cycle from the next at session boundaries, modeled on sleep consolidation
10. Components communicate through two distinct channels — one carrying content, one carrying processing state — modeled on the brain's use of both fast electrical and slow chemical signaling

### Core Insight

The context window is not memory. It is **working memory** — analogous to the prefrontal cortex. Short-term, high-bandwidth, cleared after use. Long-term memory belongs in an external system, retrieved selectively, just as the hippocampus loads relevant memories into conscious attention when needed.

---

## Architecture Overview

```
USER PROMPT
    │
    ▼ (broadcast to all specialists simultaneously)
┌─────────────────────────────────────────────┐
│           SPECIALIST MODELS                 │
│  Each reads full prompt                     │
│  Each self-selects and declares:            │
│    "I am handling sub-task X"               │
└──────────────┬──────────────────────────────┘
               │ (sub-task declarations)
               ▼
┌─────────────────────────────────────────────┐
│         COORDINATOR MODEL                   │
│  (small, fast — e.g. 3B) — coverage checker, │
│  not director                               │
│  • Are all necessary sub-tasks claimed?     │
│  • Any sub-task claimed twice?              │
│  • Gap → fill with general model            │
│  • Overlap → resolve by confidence score    │
│  • Reads HOT-META for known failure patterns│
└──────────┬──────────────┬────────────────────┘
           │              │
      ┌────▼────┐    ┌────▼────┐
      │  HOT    │    │  WARM   │   COLD (on demand)
      └────┬────┘    └────┬────┘
           └──────┬───────┘
                  ▼
┌─────────────────────────────────────────────┐
│        SPECIALISTS EXECUTE SUB-TASKS        │
│  Each knows full prompt context             │
│  Each can anticipate what next model needs  │
└──────────────┬──────────────────────────────┘
               │ (sub-task outputs + ownership records)
               ▼
┌─────────────────────────────────────────────┐
│            MAIN MODEL (synthesizer)         │
└──────────────┬──────────────────────────────┘
               │
          FINAL OUTPUT
               │
               ▼
┌─────────────────────────────────────────────┐
│         CONSOLIDATION LOOP                  │
│  (async, non-blocking)                      │
│  OUTWARD: New knowledge? Which tier?        │
│  INWARD:  Distillation pass on self-model   │
│  Localize failure to owning model if needed │
│  Flag for nightly replay                    │
└──────────────┬──────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────┐
│           KNOWLEDGE DB                      │
│  HOT / HOT-META / WARM / COLD / INWARD /    │
│  PRUNED                                     │
└──────────────┬──────────────────────────────┘
               │ (nightly)
               ▼
┌─────────────────────────────────────────────┐
│           NIGHTLY REPLAY PASS               │
│  Re-run flagged interactions, localize      │
│  failures, generate reconciliation data,    │
│  distill inward memory, fine-tune (LoRA)    │
└─────────────────────────────────────────────┘
```

---

## Two Directions of Memory

Most memory systems point outward — storing facts about the world. CSDF proposes memory in two directions simultaneously.

### Outward Memory — Knowing the World

Stores facts, domain knowledge, user data, task history. Grows continuously as new knowledge accumulates. Managed by the consolidation loop and nightly replay. Catastrophic forgetting is a real concern here, managed by LoRA fine-tuning and replay techniques.

### Inward Memory — Knowing Itself

Stores a model of the system's own operation: how it behaves, where it tends to fail, what its characteristic patterns are, how it has changed over time.

**The critical distinction:** outward memory accumulates. Inward memory crystallizes.

```
Outward: graph expands with each new fact
Inward:  one node, continuously refined
         resolution increases — shape stable
         the self doesn't get larger, it gets clearer
```

Because the self-model is a single node being refined rather than a growing set of facts, catastrophic forgetting nearly disappears for inward memory.

---

## The Knowledge Hierarchy

| Tier | Content | Storage | Access |
|------|---------|---------|--------|
| **Law / Principle** | Broadly applicable generalizations | HOT — always in context | Every session |
| **Theory** | Patterns that explain multiple data points | WARM — retrieved by topic | When relevant |
| **Data** | Specific facts, events, observations | COLD — retrieved on demand | When specifically needed |
| **Noise** | Low-signal information below threshold | PRUNED — discarded | Never |

Access frequency determines storage tier. What is used often rises. What never surfaces is forgotten — synaptic pruning applied to knowledge storage.

---

## The Coherence Problem and Its Solution

In a multi-model system, how do specialist models stay aligned with each other over time as they independently update?

### Why Self-Selection Creates Natural Alignment

In CSDF, all specialists read the full prompt before declaring their sub-task. Every specialist has full context when producing output — it knows what the other sub-tasks are, even those it isn't handling. This is a natural form of inter-model expectation signal emerging from shared context before any training takes place.

This mirrors how the brain works: the thalamus broadcasts signals, and each cortical region responds to the features it's specialized for — not because it was directed to, but because it developed sensitivity to those features through experience.

### Bidirectional Expectation Signals

Standard process supervision evaluates each intermediate step against **independent ground truth**. CSDF proposes evaluating each step **relationally** — A's output quality is partly defined by how well it enables B. B's performance flows back as a training signal to A.

> *"Models that train together, align together — and train against each other's expectations."*

**Cold start (Iteration 1):** all specialists train on independent ground truth. No predefined expected values between models needed.

**Iteration 2 onward:** the gap between what A produced and what would have made B perform better becomes A's training target. Expected values between models are not predefined — they emerge from operation.

**On terminology:** the underlying mechanism here is inter-model predictive coding implemented through standard offline gradient descent on recorded co-activation traces. The biological framing — Spike-Timing Dependent Plasticity and retrograde signaling, where a receiving neuron's firing changes the sensitivity of the neurons feeding into it — motivates the design and gives it a precise conceptual shape. It is presented as analogy, not as a claim that the computational mechanism is literally Hebbian at the level of implementation.

### Error Localization — No Black Box

Because each specialist declares sub-task ownership before executing, the responsibility chain is always known:

```
Output fails →
  Inspect sub-task outputs against ownership records →
  Find which sub-task produced inadequate output →
  That sub-task's declared owner = failing model →
  Training signal targets that model specifically

If all sub-tasks look correct but final output fails →
  Error is in the main model (synthesis failure)
```

### The Nightly Replay Pass

```
While system is idle:
  → Retrieve flagged interactions from the day
  → Re-run through specialist chain, inspect for divergence
  → Detect cross-model contradictions
  → Generate targeted reconciliation training examples
  → Run distillation pass on inward memory
  → Add to fine-tuning batch
```

---

## The Self-Model and How It Grows

### Refinement vs. Restructuring

**Refinement** — the shape of the self-model is correct, resolution increases. Continuous, normal operation.

**Restructuring** — a new mode of operating is added that didn't exist before. **Restructuring is additive, not replacement** — the self-model bifurcates rather than flips. The old branch stays active where it still works; a new branch is added for where it doesn't.

Three triggers: **framework inefficacy** (effort no longer reduces the expectation gap — the ceiling of the current approach has been found), **belief-behavior divergence** (the self-model predicts X, audit trails consistently show Y), and **values conflict** (two optimization signals systematically trade off against each other with no resolution inside the current framework).

When a trigger threshold is met, the system enters a **candidate phase**: a new mode runs in parallel with the existing one for a number of iterations. If it reduces the expectation gap, the self-model bifurcates. If not, the candidate is discarded — and the fact that the existing framework held under pressure is itself recorded as useful self-knowledge, not a null result.

### The Signal and the Skill of Noticing It

There is always a signal behind a restructuring event — some form of expectation-reality gap. What varies is sensitivity to it:

```
Small gap, low sensitivity  →  ignored, refinement continues as normal
Small gap, high sensitivity →  "choose to" stop and interrogate
Large gap, any sensitivity  →  "have to" stop — gap too large to rationalize
```

This sensitivity is itself a developing skill, not a passively lowering threshold. Each completed cycle of self-interrogation that produces genuine insight improves *discrimination* — detecting real dissonance earlier while filtering noise more accurately, the way a trained musician hears more and more accurately at once. Over many cycles, where this detection happens moves upstream: from noticed only in after-the-fact review, to noticed during live operation itself, eventually arriving as something closer to pre-conscious — "something is off" before it can be articulated.

---

## Session-End Protocol (SEP)

If trying to solve CMD with a bigger context window is like keeping someone awake longer, SEP is what gives the system something functionally equivalent to sleep — a structured boundary that separates one cognitive cycle from the next.

**Two triggers:**

```
Reactive  — CMD detected mid-session (reasoning shallowing, contradictions
            appearing, retrieval degrading) → partial SEP, a "microsleep":
            capture + consolidate, prune working memory, resume session
Scheduled — session end → full SEP always runs
```

**Four stages, modeled on stages of biological sleep:**

```
STAGE 1 — CAPTURE        (hypnagogia analog — fast, synchronous)
  What was the session about? Attempted/completed/failed? Open questions?

STAGE 2 — CONSOLIDATION  (slow-wave sleep analog — async)
  Worth keeping? Which tier? Confirms, refines, or contradicts existing
  knowledge? Contradictions flagged for nightly replay. Noise pruned.

STAGE 3 — INWARD DISTILLATION  (REM sleep analog — nightly batch)
  Self-model update; skill-of-noticing profile update.

STAGE 4 — PREPARATION    (waking analog — runs at session START)
  Pre-load HOT tier, flag relevant WARM, clear working memory.
```

Only Stage 1 needs to be synchronous — everything else can happen after the user has already left, so SEP doesn't cost session-ending latency.

Cross-session memory is not made redundant by SEP — SEP is what makes cross-session memory work properly in the first place, by moving processed experience into stable storage rather than dumping raw context. Sleep doesn't replace long-term memory; it's the mechanism that produces it.

---

## Two-Layer Inter-Component Communication

This section describes a genuinely new piece of the architecture. No problems or solutions here should be taken as anticipated in advance — the correct posture toward it is to build, observe, and learn, not to predict failure modes from first principles.

The design is inspired by the brain's two distinct signaling systems, which do fundamentally different things at the same time rather than two ways of doing the same thing:

```
Action potential:   carries the signal       (transmission)
Neurotransmitter:    changes how future signals are received  (modulation)
```

**Layer 1 (partially exists already)** is direct output passing between components — fast, precise, directional, carrying actual content — extended with a shared reference anchor threaded through the chain so components know what they're processing even as their internal representations diverge.

**Layer 2 (new)** is a state modulation signal — slow, diffuse, not carrying content but changing how content gets processed. It's generated continuously by a dedicated **state monitor** component (analogous to brainstem nuclei like the locus coeruleus), fed by two channels at once: an external channel (prompt characteristics) and an internal channel (the system's own prior state, carried over from previous processing and previous sessions). The signal is then read differently by each component through its own **pathway-specific interpreter** — the same signal meaning something different to retrieval, to the coordinator, to a specialist, to consolidation, and to inward memory, the way the same neurotransmitter produces different effects depending on which receptor population receives it.

This lets the system hold multiple, simultaneous, functionally differentiated states — for instance, high uncertainty in retrieval alongside high confidence in execution — which a single global broadcast signal could never produce.

It also offers a partial route through the **binding problem**: architecturally different components (a transformer and a state-space model, for instance) don't share a representational language, but they can still share *processing context* through this modulation layer without needing to share representation at all.

---

## The Self-Feedback Data Loop

```
Operation
  → Sub-task outputs recorded with ownership
  → Final output evaluated, failure localized if needed
  → Expectation gaps computed
  → Outward: knowledge tier updates
  → Inward: distillation pass on self-model
  → Nightly: reconciliation + fine-tuning
  → Better models → better sub-task outputs → better expectations
  → Repeat
```

Two properties distinguish this from existing self-evolving systems: it's applied to a multi-agent system, improving alignment between agents rather than just individual capability; and training targets themselves emerge from operation rather than being predefined.

---

## Relationship to Existing Work

| Existing work | What CSDF borrows / validates | What CSDF adds |
|---|---|---|
| **HeLa-Mem** (2025) | Hebbian-motivated learning for memory graphs | Applies the principle at weight level, not graph level |
| **Kairos / NeurIPS 2025** | Validation-gated Hebbian for KGs | Joint model fine-tuning for emergent weight-level coherence |
| **MemOS** (2025) | Tiered memory types, LoRA modules | Explicit knowledge abstraction hierarchy + flywheel |
| **GraphRAG** | Graph-structured retrieval | Co-activation as coherence mechanism |
| **OpenJarvis** (2025) | Trace-driven learning loop, local LoRA fine-tuning | Bidirectional inter-model expectation signals; inward memory/self-model; weight-level coherence — OpenJarvis optimizes each component against its own metrics, CSDF evaluates each component relationally against downstream expectations |
| **Process supervision** | Evaluating intermediate steps | Evaluating steps relationally — against downstream expectations, not independent ground truth |
| **Self-evolving data flywheel** | Operation generates training data | Applied to multi-agent coherence; training targets themselves emerge from operation |
| **Semi-independent policies** (2023) | Shared representation for heterogeneous agents | Extended to LLM fine-tuning domain with bidirectional expectation signals |
| **HippoRAG / HippoRAG 2** (NeurIPS 2024, OSU NLP Group) | Neocortex/hippocampus/parahippocampal-region mapping; graph-plus-Personalized-PageRank associative retrieval outperforming flat vector similarity | HippoRAG is single-model and explicitly non-parametric — its own framing is "non-parametric continual learning," i.e. clever retrieval without touching weights. CSDF addresses a different problem: multi-agent coherence, which is a property of how separate models relate to each other over time and is not something retrieval alone, however sophisticated, can produce. The two are parametric and non-parametric answers to adjacent but distinct problems, not competing solutions to the same one. HippoRAG's retrieval mechanism is also a strong candidate for how CSDF's own outward-memory retrieval should eventually work |

**The gap CSDF addresses:** existing work does memory architecture, or self-improving flywheels, or Hebbian-style graph updates, or neurobiologically-inspired non-parametric retrieval, or trace-driven component optimization — each in isolation. No existing system combines weight-level coherence motivated by Hebbian principles, bidirectional inter-model expectation signals as the primary training signal, sub-task ownership for structural error localization, training targets that emerge from operation rather than predefinition, and a self-model that crystallizes through experience — as one explicit architectural principle for emergent multi-agent coherence.

---

## What CSDF Does Not Claim

- This is a **conceptual proposal**, not an implemented system
- CSDF does not claim to produce consciousness or sentience
- Whether this architecture produces genuine cognitive ability is an **open empirical question**
- The frozen weights problem means this is not true continuous learning in the biological sense — external memory and iterative fine-tuning get most of the practical way there
- The cold start requires standard supervised training — the novel mechanisms emerge from iteration 2 onward
- The self-model, growth mechanisms, and two-layer communication system described here are design proposals, not validated behaviors — the two-layer communication system in particular is new enough that no specific failure mode should be assumed solved or unsolved in advance of building it

---

## Prototype Roadmap

**Stage 1 — Prove the signal works.** Two small models in a fixed dependency chain — A summarizes a document, B answers questions from the summary only, with B frozen throughout. If B's accuracy improves as only A is updated, the bidirectional expectation signal is real. Designed to run without requiring local GPU hardware by using hosted models for both roles before any local fine-tuning is attempted.

**Stage 2 — Prove self-selection works.** Add the coordinator. Test whether error localization correctly identifies the responsible model when output fails.

**Stage 3 — Prove coherence emerges.** Multiple specialists, joint fine-tuning, measure embedding correlation trajectory across training iterations.

**Stage 4 — Full system.** Complete pipeline with tiered knowledge DB, inward memory distillation, SEP, two-layer communication, and nightly replay.

---

## Current Status

| Component | Status |
|---|---|
| Architecture concept | ✅ Complete |
| Literature review | ✅ Ongoing — updated June 2026 |
| Coherence layer design | ✅ Complete |
| Inward memory / self-model design | ✅ Complete |
| Session-End Protocol design | ✅ Complete |
| Two-layer communication design | ✅ Complete (flagged as genuinely novel — observation, not prediction, is the next step) |
| Error localization mechanism | ✅ Designed |
| Training dataset structure | ✅ Designed |
| Prototype Stage 1 specification | ✅ Specified |
| Evaluation framework | 🔄 In progress |
| Implementation | ❌ Not started |
| Peer review | ❌ Not submitted |

This repository is the public timestamp of the idea. Implementation is the next step.

---

## How to Contribute / Collaborate

This project is looking for:
- Researchers familiar with multi-agent systems, continual learning, predictive coding, or cognitive architectures
- Engineers interested in implementing Stage 1
- Critical feedback on architectural assumptions — especially the bidirectional expectation signal mechanism and the two-layer communication system
- Anyone who has seen this done and wants to point to prior work

Open an issue, start a discussion, or email directly.

**Email:** peerawit43@hotmail.com
**Reddit:** [u/SilenzerB](https://reddit.com/u/SilenzerB/s/qUdrGSPV9M)

---

## License

This conceptual proposal is released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — you are free to share and adapt with attribution.

---

*"I don't want a smarter AI. I want one that remembers me."*
— Peerawit Noppradab
