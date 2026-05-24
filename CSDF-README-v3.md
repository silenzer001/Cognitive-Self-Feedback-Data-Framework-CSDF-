# CSDF: Cognitive Self-Feedback Data Framework

**A brain-inspired architecture for persistent, self-evolving AI memory**

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

*Conceived: May 14, 2026 | Architecture v2: May 16, 2026 | Architecture v3: May 25, 2026*

---

## Disclosure

This architecture was conceived by Peerawit Noppradab. Claude (Anthropic) was used as a research assistant for literature mapping, concept articulation, and drafting. All core ideas, architectural intuitions, and intellectual contributions are the author's own. This is an early conceptual proposal — not yet implemented or peer-reviewed. Feedback and collaboration welcome.

---

## The Problem: Context Memory Degradation (CMD)

Anyone who has had a long conversation with an AI assistant knows what happens: the model starts to "forget" things said earlier, contradicts itself, loses the thread. Start a new session and it remembers nothing at all.

This happens because modern language models use **attention mechanisms** — every token attends to every other token at a cost that scales quadratically (O(n²)) with context length. Older information gets diluted. When the session ends, everything is gone.

The standard workaround is manual context management: handoff documents, system prompts, periodic summaries. It works — but it's a patch, not a solution. And it puts the burden of memory entirely on the human.

**The deeper question: how does the brain solve this problem?**

It doesn't rely on one system holding everything. It distributes memory across specialized regions, each handling a different type. It consolidates during sleep. It forgets selectively — pruning noise, strengthening important connections. It learns continuously from experience.

What if an AI system could do the same?

---

## The CSDF Architecture

**CSDF (Cognitive Self-Feedback Data Framework)** is a proposed multi-agent architecture where:

1. Memory is distributed across two directions — outward (world knowledge) and inward (self-knowledge) — stored in a tiered knowledge graph, not a single context window
2. Specialist models self-select their sub-tasks from a shared prompt broadcast — no central director
3. A coordinator model checks coverage and resolves gaps or overlaps
4. Each specialist's output is evaluated relationally — against downstream model expectations, not independent ground truth
5. The gap between what a specialist produced and what the next model needed flows back as a training signal
6. Error localization is structural — sub-task ownership makes failure traceable to a specific model
7. Training targets emerge from operation. They are not predefined.
8. The system builds a model of itself — not just the world — and that self-model crystallizes through experience

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
│                                             │
│  spec-A: conceptual / reasoning             │
│  spec-B: implementation / code              │
│  spec-C: memory / retrieval                 │
│  spec-N: ...                                │
└──────────────┬──────────────────────────────┘
               │ (sub-task declarations)
               ▼
┌─────────────────────────────────────────────┐
│         COORDINATOR MODEL                   │
│  (small, fast — e.g. 3B)                    │
│  • Are all necessary sub-tasks claimed?     │
│  • Any sub-task claimed twice?              │
│  • Gap → fill with general model            │
│  • Overlap → resolve by confidence score    │
│  • Reads HOT-META for known failure patterns│
└──────────┬──────────────┬────────────────────┘
           │              │
      ┌────▼────┐    ┌────▼────┐
      │  HOT    │    │  WARM   │   COLD (on demand)
      │  tier   │    │  tier   │
      │(laws/   │    │(theory) │
      │princip.)│    │         │
      └────┬────┘    └────┬────┘
           └──────┬───────┘
                  │ (retrieved context injected per specialist)
                  ▼
┌─────────────────────────────────────────────┐
│        SPECIALISTS EXECUTE SUB-TASKS        │
│  Each produces output for its declared task │
│  Each knows the full prompt context         │
│  Each can anticipate what the next model    │
│  needs — because they read the same prompt  │
└──────────────┬──────────────────────────────┘
               │ (sub-task outputs + ownership records)
               ▼
┌─────────────────────────────────────────────┐
│            MAIN MODEL                       │
│  (orchestrator / synthesizer)               │
│  Receives all sub-task outputs              │
│  Synthesizes into final answer              │
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
│  Training worthy?                           │
│  If poor → inspect sub-task chain           │
│  Localize failure to owning model           │
│  Flag for nightly replay                    │
│  Prune noise                                │
└──────────────┬──────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────┐
│           KNOWLEDGE DB                      │
│  (graph-linked)                             │
│  HOT      — laws/principles                 │
│  HOT-META — system self-knowledge           │
│  WARM     — theories/patterns               │
│  COLD     — specific data                   │
│  INWARD   — self-model node (crystallizing) │
│  PRUNED   — noise, discarded                │
└──────────────┬──────────────────────────────┘
               │ (nightly)
               ▼
┌─────────────────────────────────────────────┐
│           NIGHTLY REPLAY PASS               │
│  • Re-run flagged interactions              │
│  • Inspect sub-task chain for failures      │
│  • Localize: which model's output failed?   │
│  • Generate targeted reconciliation data    │
│  • Detect cross-model contradictions        │
│  • Run distillation pass on inward memory   │
│  • Fine-tune: daily LoRA / weekly deeper    │
└─────────────────────────────────────────────┘
```

---

## Two Directions of Memory

Most memory systems point outward — storing facts about the world. CSDF proposes memory in two directions simultaneously.

### Outward Memory — Knowing the World

Stores facts, domain knowledge, user data, task history. Grows continuously as new knowledge accumulates. Organized by the knowledge hierarchy. Managed by the consolidation loop and nightly replay. Catastrophic forgetting is a real concern here, managed by LoRA fine-tuning and replay techniques.

### Inward Memory — Knowing Itself

Stores a model of the system's own operation: how it behaves, where it tends to fail, what its characteristic patterns are, how it has changed over time.

**The critical distinction:** outward memory accumulates. Inward memory crystallizes.

```
Outward: graph expands with each new fact
Inward:  one node, continuously refined
         resolution increases — shape stable
         the self doesn't get larger, it gets clearer
```

Because the self-model is a single node being refined rather than a growing set of facts, catastrophic forgetting nearly disappears for inward memory. Refining one stable target is safe in a way that accumulating thousands of facts is not.

---

## The Knowledge Hierarchy

Rather than treating all stored information equally, CSDF proposes explicit abstraction tiers:

| Tier | Content | Storage | Access |
|------|---------|---------|--------|
| **Law / Principle** | Broadly applicable generalizations | HOT — always in context | Every session |
| **Theory** | Patterns that explain multiple data points | WARM — retrieved by topic | When relevant |
| **Data** | Specific facts, events, observations | COLD — retrieved on demand | When specifically needed |
| **Noise** | Low-signal information below threshold | PRUNED — discarded | Never |

This mirrors the progression from raw experience to crystallized knowledge in human learning — and in the history of science itself (observation → hypothesis → theory → law).

**Access frequency determines storage tier.** What is used often rises. What is rarely needed sinks. What never surfaces is forgotten — selectively, not blindly. This is synaptic pruning applied to knowledge storage.

---

## The Coherence Problem and Its Solution

In a multi-model system, a critical problem emerges: how do specialist models stay aligned with each other over time as they independently update?

### Why Self-Selection Creates Natural Alignment

In CSDF, all specialists read the full prompt before declaring their sub-task. This means every specialist has full context when producing output — it knows what the other sub-tasks are, even those it isn't handling.

This is a natural form of inter-model expectation signal that emerges from shared context before any training. Specialist A can anticipate what specialist B needs because both read the same prompt. The broadcast-and-self-select model creates alignment structurally, not just through training.

This mirrors how the brain actually works: the thalamus broadcasts signals, and each cortical region responds to the features it is specialized for — not because it was directed to, but because it developed sensitivity to those features through experience.

### Bidirectional Expectation Signals

Standard process supervision evaluates each intermediate step against **independent ground truth** — each step is judged on its own merits.

CSDF proposes something different: evaluate each step **relationally** — A's output quality is partly defined by how well it enables B. B's performance flows back as a training signal to A.

> *"Models that train together, align together — and train against each other's expectations."*

**Cold start (Iteration 1):**
All specialists train on independent ground truth. No predefined expected values between models are needed. System runs, sub-task outputs are recorded alongside final output quality.

**Iteration 2 onward:**
Performance gaps become the training signal. If B performed poorly given A's output, the gap between A's actual output and "what would have made B perform better" becomes A's new training target. A now optimizes not just for correctness in isolation — but for how well its output serves B.

Expected values between models are not predefined. They emerge from operation.

**Biological analog:** This is analogous to **retrograde signaling and Spike-Timing Dependent Plasticity (STDP)** at the inter-model scale. Neuron B fires sub-action potentials back to every neuron connected to it, changing receptor density — modifying how sensitive they are to future inputs. A learns not just from its own output, but from what B expected to receive.

### Error Localization — No Black Box

Because each specialist declares sub-task ownership before executing, the responsibility chain is always known.

```
Output fails →
  Inspect sub-task outputs against ownership records →
  Find which sub-task produced inadequate output →
  That sub-task's declared owner = failing model →
  Training signal targets that model specifically

If all sub-tasks look correct but final output fails →
  Error is in the main model (synthesis failure)
  Upstream models are not the problem
```

This is made possible because specialists read the full prompt and self-select. Unlike a router-directed system where misrouting can confuse error localization, self-selection with declared ownership means the responsibility chain is always traceable.

### The Nightly Replay Pass

Inspired by hippocampal consolidation during sleep, CSDF includes an asynchronous nightly pass:

```
While system is idle:
  → Retrieve flagged interactions from the day
  → Re-run through specialist chain
  → Inspect sub-task outputs against ownership records
  → Identify divergence point: where did the chain fail?
  → Detect cross-model contradictions
  → Generate targeted reconciliation training examples
  → Run distillation pass on inward memory
  → Add to fine-tuning batch
```

This is not just learning from what happened. It is actively correcting failures before they become entrenched in weights.

---

## The Self-Model and How It Grows

The inward memory does more than store failure patterns. It maintains a living model of how the system operates — what it does well, what it tends to get wrong, and how its behavior has changed over time.

### Refinement vs. Restructuring

Not all updates to the self-model are the same.

**Refinement** is when the shape of the self-model is correct and resolution increases. "I handle ambiguity carefully" becomes "I handle ambiguity carefully when time allows, less so under time pressure." Same structure, higher fidelity. This happens continuously through normal operation.

**Restructuring** is when the shape itself needs to change — when a new mode of operating is developed that didn't exist before. Not a 180-degree personality shift, but the addition of a new way of thinking: a new skill, a new approach to a class of problems that the old framework couldn't handle.

Restructuring is triggered by one of three signals:

1. **Framework inefficacy** — the system pushes harder within its current approach but the expectation gap plateaus. Effort is no longer reducing error. The ceiling of the current framework has been found.
2. **Belief-behavior divergence** — the self-model predicts the system will behave in manner X, but audit trails show it consistently does Y across different task types.
3. **Values conflict** — two optimization signals pull in opposite directions systematically. Satisfying one consistently degrades the other. The system cannot resolve the trade-off within its current framework.

When a restructuring trigger threshold is met, the system enters a **candidate phase**: a new mode is generated and run in parallel with the existing mode for a number of iterations. If the new mode reduces the expectation gap in the relevant context, the self-model bifurcates — the new mode is added conditionally alongside the old one. If not, the candidate is discarded and the stress-test itself is recorded: the existing framework held under pressure.

**Restructuring is additive, not replacement.** The self-model doesn't flip — it becomes more conditional. A new branch is added. The old branch remains active in the contexts where it still works. This mirrors how humans actually develop through adversity: not losing who they were, but becoming someone who contains what they were plus a new dimension.

### The Signal That Initiates Growth

Growth is not purely proactive. There is always a signal — some form of expectation-reality gap. The outcome wasn't what was planned. The behavior wasn't what the self-model predicted. Something in the environment didn't match the internal model.

What varies is not whether there is a signal, but the system's sensitivity to it:

```
Small gap, low sensitivity  →  ignored, refinement continues as normal
Small gap, high sensitivity →  "choose to" stop and interrogate
Large gap, any sensitivity  →  "have to" stop — gap too large to rationalize
```

This sensitivity is itself a skill — and it develops through repeated cycles. Each completed cycle of self-interrogation that produces genuine insight makes the system slightly more sensitive to weak signals. Not because the threshold passively lowers, but because the system develops better discrimination: detecting real dissonance earlier while filtering noise more accurately.

The more cycles completed, the easier it becomes to notice the signal. And each time the cycle runs, the starting point is richer than before.

---

## The Self-Feedback Data Loop

CSDF's name comes from its central mechanism: **the system's own operation generates the data that improves it — including the training targets themselves.**

```
Operation
  → Sub-task outputs recorded with ownership
  → Final output evaluated
  → Failure localized if needed
  → Expectation gaps computed (what downstream needed vs. what it received)
  → Outward: knowledge tier updates
  → Inward: distillation pass on self-model
  → Nightly: reconciliation + fine-tuning
  → Better models → better sub-task outputs → better expectations
  → Repeat
```

Two properties distinguish this from existing self-evolving systems:

1. Applied to a **multi-agent system** — the flywheel improves alignment between agents, not just individual capability
2. **Training targets emerge from operation** — the expected outputs are not predefined by humans, they are generated by the system discovering what each model needs from the others

---

## Relationship to Existing Work

CSDF is not built from nothing. It draws on and extends:

| Existing work | What CSDF borrows | What CSDF adds |
|---|---|---|
| **HeLa-Mem** (2025) | Hebbian learning for memory graphs | Applies Hebbian at weight level, not graph level |
| **Kairos / NeurIPS 2025** | Validation-gated Hebbian for KGs | Joint model fine-tuning for emergent weight-level coherence |
| **MemOS** (2025) | Tiered memory types, LoRA modules | Explicit knowledge abstraction hierarchy + flywheel |
| **GraphRAG** | Graph-structured retrieval | Co-activation as coherence mechanism |
| **OpenJarvis** (2025) | Trace-driven learning loop, local LoRA fine-tuning | Bidirectional inter-model expectation signals; inward memory / self-model; Hebbian weight-level coherence — OpenJarvis optimizes each component against its own metrics, CSDF evaluates each component relationally against downstream expectations |
| **Process supervision** | Evaluating intermediate steps | Evaluating steps relationally — against downstream expectations, not independent ground truth |
| **Self-evolving data flywheel** | Operation generates training data | Applied to multi-agent coherence; training targets themselves emerge from operation |
| **Semi-independent policies** (2023) | Shared representation for heterogeneous agents | Extended to LLM fine-tuning domain with bidirectional expectation signals |

**The gap CSDF addresses:**

Existing work either does memory architecture, or self-improving flywheels, or Hebbian graph updates, or trace-driven component optimization. No existing system combines:

- Hebbian co-activation at the model weight level
- Bidirectional inter-model expectation signals as the primary training signal
- Sub-task ownership for structural error localization
- Training targets that emerge from operation rather than predefinition
- A self-model that crystallizes through experience (inward memory)

as an explicit architectural principle for emergent multi-agent coherence.

The bidirectional expectation flow is analogous to retrograde signaling in biological neural circuits — a mechanism explored formally in predictive coding frameworks.

---

## What CSDF Does Not Claim

- This is a **conceptual proposal**, not an implemented system
- CSDF does not claim to produce consciousness or sentience
- Whether this architecture produces genuine cognitive ability is an **open empirical question** — and an interesting one
- The frozen weights problem means this is not true continuous learning in the biological sense — but external memory and iterative fine-tuning get most of the practical way there
- The cold start requires standard supervised training — the novel mechanisms emerge from iteration 2 onward
- The self-model and growth mechanisms described here are design proposals, not validated behaviors

---

## Prototype Roadmap

The architecture is designed to be validated incrementally. Each stage proves one thing before the next begins.

**Stage 1 — Prove the signal works** *(target hardware: 4GB VRAM)*
Two small models in a fixed dependency chain. Model A summarizes a document. Model B answers questions from the summary only. A's training signal is computed from B's failures. If B's accuracy improves with B frozen and only A changing, the bidirectional expectation signal is real.

**Stage 2 — Prove self-selection works**
Add coordinator. Let models declare sub-task ownership. Test whether error localization correctly identifies the responsible model when output fails.

**Stage 3 — Prove coherence emerges**
Multiple specialists, joint fine-tuning, measure embedding correlation trajectory across training iterations. Does cross-model alignment improve as a function of co-activation, not just individual training?

**Stage 4 — Full system**
Complete pipeline with tiered knowledge DB, inward memory distillation, and nightly replay.

---

## Current Status

| Component | Status |
|---|---|
| Architecture concept | ✅ Complete |
| Literature review | ✅ Completed (May 2026) |
| Coherence layer design | ✅ Complete (May 2026) |
| Inward memory / self-model design | ✅ Complete (May 2026) |
| Error localization mechanism | ✅ Designed |
| Nightly replay mechanics | ✅ Designed |
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
- Engineers interested in implementing Stage 1 (two-model LoRA pipeline, ~4GB VRAM feasible)
- Critical feedback on architectural assumptions — especially the bidirectional expectation signal mechanism
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
