# Connectome OS

**A debugging and control layer for embodied graph systems whose structure is *knowable* — the connectome — rather than learned.**

"OS" in the Linux sense: infrastructure for introspection and intervention, not a mystical claim about emergent mind. Not mind upload. Not digital consciousness. Not AGI. Connectome OS is the graph-native runtime that mounts on top of a connectome + a spiking engine and lets you **probe, perturb, and reason about the structure — cut the wiring, measure the fracture, ask what substructure carried the failure.**

> ⚠️ **Alpha / research preview.** Connectome OS is in active development on the [`research/connectome-ruvector`](https://github.com/ruvnet/RuVector/tree/research/connectome-ruvector) branch. The Tier-1 fly-scale demonstrator (`examples/connectome-fly/`) is shipped and all 73 tests pass; the production `ruvector-connectome` / `ruvector-lif` / `ruvector-embodiment` crates, real FlyWire v783 data ingest, MuJoCo-based embodiment, and Tier-2 mouse-scale substrate are **named follow-ups, not shipped**. APIs may change. Measured numbers, missed SOTA targets, and reverted follow-up levers are documented honestly in [ADR-154 §17](https://github.com/ruvnet/RuVector/blob/research/connectome-ruvector/docs/adr/ADR-154-connectome-embodied-brain-example.md) (13 measurement-driven discoveries preserved). Use in research and internal tooling; hold on anything that expects a stable API contract.

- Source code: [ruvnet/RuVector @ `research/connectome-ruvector`](https://github.com/ruvnet/RuVector/tree/research/connectome-ruvector)
- Working example (Tier-1 fruit fly): [`examples/connectome-fly/`](https://github.com/ruvnet/RuVector/tree/research/connectome-ruvector/examples/connectome-fly)
- Architecture Decision Record: [ADR-154](https://github.com/ruvnet/RuVector/blob/research/connectome-ruvector/docs/adr/ADR-154-connectome-embodied-brain-example.md)
- Research gist (public summary): [29be261d41ebd66dcdb9e389e9393458](https://gist.github.com/ruvnet/29be261d41ebd66dcdb9e389e9393458)

---

## Table of contents

- [Applications — practical to exotic](#applications--practical-to-exotic)
- [Introduction](#introduction)
- [Features](#features)
- [Capabilities & comparison](#capabilities--comparison)
- [Quick start](#quick-start)
- [User guide](#user-guide)
- [Fly-AI guide](#fly-ai-guide)
- [Measurement-driven findings](#measurement-driven-findings)
- [What Connectome OS is *not*](#what-connectome-os-is-not)
- [Contributing](#contributing)

---

## Applications — practical to exotic

Connectome OS is a substrate. What you build *with* it depends on what connectome you load, what stimulus you drive, and what structural questions you ask. The applications below are organized by distance from the shipped Tier-1 demonstrator — the first group works today, the middle group needs API wrappers or corpus preparation but no new research, and the last group needs one or more Phase-2 / Phase-3 items from the implementation plan.

### Part 1 — Practical (works today on the shipped demo)

Each of these runs against `examples/connectome-fly/` as it stands. No external data, no missing phase, no research pivot required.

| Application | Who / why | How Connectome OS enables it |
|---|---|---|
| **FlyWire exploration & audit** | Computational neuroscientists studying *Drosophila*; teaching labs | Load real wiring via `load_flywire_streaming`, drive with deterministic stimulus, watch live partitions + motifs — all without a Python stack |
| **Community-detection ground truth** | Graph-algorithm researchers evaluating Leiden / Louvain variants | SBM + hub-module ground truth, mincut, level-1 greedy, and multi-level Louvain baselines with published ARI numbers — drop a new algorithm in and get a paired comparison row |
| **Reference LIF for spiking-simulator benchmarks** | Anyone building a spiking simulator or neuromorphic chip who needs a measured, deterministic, open-source baseline | `lif_throughput` bench + bit-exact AC-1 contract; ~7.6 M spikes/sec sparse per-step, single-threaded Rust reference |
| **Connectome-constrained coursework** | Grad students, structured-AI courses | Single `cargo test` reproduces all 5 acceptance criteria on the default SBM; no MuJoCo / PyTorch / CUDA dependencies |
| **Fragility monitoring on recorded spike data** | Groups with experimental ephys recordings | The Fiedler detector is substrate-agnostic — feed it a spike stream from a biological recording, a trained RNN, or a custom simulator, and the live `λ₂` coherence signal + precursor events come out the same |

### Part 2 — Semi-practical (needs API wrappers or corpus preparation, but no new research)

These reuse the shipped substrate on a different kind of input. No new algorithms, no Phase-2/3 work — just thin wrappers and a clear problem statement.

| Application | Who / why | How Connectome OS enables it |
|---|---|---|
| **Causal-perturbation tests on trained networks** | Interpretability / alignment researchers studying trained recurrent models | Re-interpret the "connectome" as an RNN weight graph; the mincut + σ-separation test identifies which weight subsets are load-bearing for behaviour X |
| **Graph-constrained RL policy priors** | RL researchers using biological wiring as inductive bias | Initialize policy-network connectivity from a real connectome, measure how much behaviour depends on structure vs learned weights via the same causal-perturbation gate |
| **Neural-architecture pruning with proof** | ML systems groups pruning networks without losing capability | `ruvector-mincut` produces *certified* cuts; AC-5-style σ-separation proves which prunings change behaviour and which don't |
| **Neuromorphic hardware verification** | Chip designers validating spiking silicon against a known-deterministic reference | The bit-exact AC-1 contract gives you a gold-standard spike trace — run the same stimulus on hardware and Connectome OS, diff the traces |

### Part 3 — Exotic (needs Phase-2 or Phase-3 scaffolding)

These are what the substrate is ultimately *for*. Each depends on at least one deferred item from `docs/research/connectome-ruvector/08-implementation-plan.md` (real-data ingest, MuJoCo body, Tier-2 mouse substrate) — but **public APIs + stubs for 4 of the 5 are now in-tree** (commit [`07cbb8d8c`](https://github.com/ruvnet/RuVector/commit/07cbb8d8c)), so downstream code can type against them today against the deterministic default implementation and swap the production piece in later.

| Application | Who / why | How Connectome OS enables it | Scaffolding status | Blocker to full |
|---|---|---|---|---|
| **Embodied fly navigation in VR** | HRI + embodied-AI research requiring a tractable, fully-inspectable brain | Tier-1 fly scale (~139 k neurons) simulated at > real-time on a workstation, driving a virtual fly through visual / olfactory stimuli | ✅ `BodySimulator` trait in `src/embodiment.rs`; `StubBody` (deterministic open-loop) works today; `MujocoBody` panic-stub documents the FFI shape | Phase-3 MuJoCo + NeuroMechFly `cxx` bridge |
| **In-silico circuit-lesion studies** | Computational psychiatry exploring focal-lesion hypotheses without animal work | The σ-separation protocol turns "we cut this and behaviour Y changed" into a falsifiable engineering claim with paired controls | ✅ `LesionStudy` + `CandidateCut` + `LesionReport` in `src/lesion.rs`. Paired-trial loop + σ distance against a nominated reference cut; deterministic. Productizes AC-5. | Real FlyWire data for clinical plausibility |
| **Cross-species connectome transfer tests** | Comparative neuroscience asking "does a motif that matters in fly also matter in mouse?" | Same runtime, two different connectomes, same motif-retrieval index; measure shared behavioural vocabulary | ⏳ Not yet — needs a second heterogeneous connectome loaded into the same runtime | Tier-2 mouse substrate |
| **Connectome-grounded AI safety auditing** | Alignment research: can a system's behaviour be explained by substructure, and is that substructure stable under perturbation? | A connectome-constrained system is uniquely auditable — the structure is *knowable* rather than learned, so AC-5-style "remove this, see what breaks" is meaningful | ✅ `StructuralAudit::new(&conn, stim).run()` in `src/audit.rs` — one call produces a `StructuralAuditReport` with coherence events, both mincut partitions, motif-corpus size, and σ-separation on auto-generated boundary vs interior cuts | None — the tooling *is* the shipped scaffolding |
| **Substrate for structural-intelligence research papers** | Anyone pursuing the "cut the brain, measure the fracture" research program as a publishable line | The 13 measurement-driven discoveries are reproducible from the one-liner `cargo bench -p connectome-fly`; the substrate *is* the paper's methods section | ✅ Already open — no scaffolding needed | None |

### Rule of thumb

If your application wants a deterministic spiking simulation paired with a live structural-analysis loop on a wiring diagram you trust, Connectome OS is the substrate. If your application needs general-purpose AI reasoning, an LLM, or statistical learning from raw data, Connectome OS is not the right tool — it's specifically for systems where the *structure is the thing you want to reason about*.

---

## Introduction

Connectome OS is a Rust runtime that runs a connectome as a live spiking system, watches its structural coherence every 5 ms of simulated time, and lets you **cut load-bearing edges on demand and measure — in σ of a paired control — exactly what breaks**. It is the only tool in its class that treats structural perturbation as a first-class acceptance criterion, not an afterthought.

### Who this is for

- **Computational neuroscientists** with a connectome and a question that looks like *"does this substructure matter for behaviour X?"*
- **Interpretability / alignment researchers** who want a falsifiable, σ-gated definition of "causal component" for recurrent systems.
- **Graph-algorithm researchers** who need a deterministic, labelled, paired-baseline substrate to evaluate community-detection algorithms against.
- **Systems engineers** building spiking simulators, neuromorphic chips, or graph-native runtimes who need a measured, bit-exact Rust reference.
- **Teachers** who need a single `cargo test` that reproduces every claim of a Tier-1 brain simulation on a commodity workstation, with no Python / CUDA / MuJoCo stack.

### The gap this closes

Brain-scale simulation today has two shapes. You either **run a spiking simulator** (Brian2, NEST, Auryn, GeNN) and get behaviour traces with no structural explanation, or **run a graph analysis** (clusters, motifs, degree distributions) and never touch dynamics. When the simulated system works-then-stops-working, neither tool can tell you why.

The 2024 *Nature* whole-fly-brain paper broke through on the simulation half — a leaky integrate-and-fire model built straight from the [FlyWire connectome](https://flywire.ai) reproduced grooming, feeding, and other sensorimotor behaviours with zero learned parameters. What it did not ship is the **explanatory layer**: a system that, while the simulation runs, keeps telling you which substructure of the graph is carrying the current behaviour, and what happens if you remove it.

Connectome OS is that explanatory layer, implemented against the same LIF primitives.

### What's different about it

- **Live Fiedler coherence detection.** The second-smallest eigenvalue of the co-firing graph's Laplacian is recomputed every 5 ms simulated and emitted as a `CoherenceEvent` when the graph is about to fragment. Measured: ≥ 50 ms lead on ≥ 70 % of constructed-collapse trials. **No other spiking simulator ships this signal live.**
- **σ-separation as a gate test.** Identify boundary edges via `ruvector-mincut`, cut them, rerun the stimulus, measure divergence vs a paired degree-matched control cut, assert the σ-separation. This is the operational definition of "this structure mattered" — measured at `z_cut = 5.55σ` (hits the 5σ SOTA target) on the Tier-1 demo.
- **Bit-exact determinism within path.** Same seeds produce bit-identical spike traces. Repeat runs are a contract, not a hope. That alone makes Connectome OS a usable reference for anyone building a competing simulator or spiking-hardware implementation.
- **Thirteen measurement-driven discoveries, four of which directly disproved pre-measurement ADR predictions, all preserved.** No hidden gaps, no threshold relaxations to force green tests. The ADR §17 table of findings is the methodology section.
- **Shipped Leiden community-detection baseline.** The three-phase Traag-et-al.-2019 refinement algorithm is in-tree (`src/analysis/leiden.rs`) with **perfect ARI = 1.000 on a hand-crafted 2-community planted SBM** where Louvain collapses to ARI = 0.000 — direct vindication that refinement-then-aggregate is what hub-heavy SBM community detection needs.

### The positioning, in one paragraph

Connectome OS is **not** a model of consciousness, **not** mind upload, and **not** a substrate-independent intelligence claim. It is a *structurally grounded, partially biological, causal simulation system* — infrastructure for probing intelligence as a system whose wiring is *knowable* (the connectome) rather than learned. The edge is not scale. The edge is **control**: once the substrate is in place, the interesting questions stop being "what did the system do?" and start being "what structure made that inevitable, and what happens when I remove that structure?" If that framing matches what you're trying to build, Connectome OS is the right tool. If your project is general-purpose AI reasoning, an LLM, or statistical learning from raw data, Connectome OS is not the right tool — and the [Applications](#applications--practical-to-exotic) table above already sorts the match-quality question.

### Why now

Three things converge:

1. **Connectomes exist at fly scale.** The full adult *Drosophila* connectome (~139 k neurons, ~50 M synapses) is publicly available through FlyWire / Janelia. Mouse cortical barrels are a few years out. The data bottleneck that killed prior attempts is gone at Tier 1.
2. **CPU-only event-driven LIF is fast enough in the realistic (sparse) regime.** ~7.6 M spikes/sec per step at N=1024, single-threaded, pure Rust — 38× to 150× the published Brian2 + C++-codegen range.
3. **The graph primitives already exist.** `ruvector-mincut` ships subpolynomial dynamic cuts with audit certificates; `ruvector-sparsifier` ships spectral sparsification; `ruvector-attention` ships SDPA. Those weren't built for neuroscience — they were built for graph systems — and that turns out to be exactly what a connectome runtime needs.

Any one of those conditions in isolation has been true for years. The three together is a new window.

---

## Features

Connectome OS ships six first-class capabilities. Each has a test and a publicly measured number. **All 87 tests pass / 0 fail** on the reference host at [commit `f58f0c98f`](https://github.com/ruvnet/RuVector/commit/f58f0c98f) — the Leiden-refinement merge.

| Feature | What it does | Measured |
|---|---|---|
| **Event-driven LIF engine** | Simulates spike dynamics on a connectome via timing-wheel dispatch, SoA layout, f32x8 SIMD, opt-in delay-sorted CSR | Saturated N=1024: **~4× speedup over scalar baseline** (commit 10) — first measurement to clear the ≥ 2× ADR target |
| **Live Fiedler coherence detector** | Computes `λ₂` of the co-firing-window Laplacian every 5 ms simulated; fires a `CoherenceEvent` when the graph is about to fragment | AC-4 strict: ≥ 50 ms lead on ≥ 70 % of 30 trials, **PASS** |
| **σ-separation causal perturbation** | Identifies load-bearing edges via mincut, cuts them, measures how much behavior diverges vs a paired control cut; the *operational definition* of "this structure mattered" | AC-5: `z_cut = 5.55σ` (hits SOTA 5σ target); `z_rand = 1.57σ` (honest gap on null tightness) |
| **Structural + functional partition** | Two mincut paths — `structural_partition` on the static graph (recovers SBM modules) + `functional_partition` on coactivation (moves with stimulus) | AC-3a + AC-3b both **PASS** |
| **Spike-window motif index** | SDPA-embedded 100 ms spike rasters, indexed in-memory for nearest-neighbour retrieval of repeated patterns | AC-2: precision@5 = 0.60 distance-proxy (honest gap vs SOTA 0.80; see findings below) |
| **Deterministic reproducibility** | Every run keyed by `(connectome_seed, stimulus_seed, engine_seed)` produces bit-identical spike traces within-path | AC-1 bit-exact on 194,784 spikes across repeat runs |

Additional infrastructure:

- **FlyWire v783 ingest** (`src/connectome/flywire/`) — fixture-tested TSV parser; streaming variant drops the ~2 GB intermediate `Vec<SynapseRecord>` buffer on real-scale data
- **Sparse-Fiedler dispatch** (`src/observer/sparse_fiedler.rs`) — `O(n + nnz)` memory path at n > 1024; 40× memory reduction at N=10K vs dense
- **Degree-stratified null sampler** (`src/connectome/stratified_null.rs`) — decile-matched random-cut baseline for AC-5 at FlyWire scale
- **Multi-level Louvain baseline** (`src/analysis/structural.rs::louvain_labels`) — the stepping stone Leiden's refinement phase will beat; kept with a docstring warning because it over-merges on hub-heavy SBMs
- **Opt D delay-sorted CSR** (`src/lif/delay_csr.rs`) — per-row CSR sorted by synaptic delay; opt-in behind `EngineConfig.use_delay_sorted_csr`
- **GPU SDPA scaffold** (`src/analysis/gpu.rs`) — cudarc-gated; CPU fallback; panics with an actionable diagnostic if the toolkit isn't linked

---

## Capabilities & comparison

### Connectome OS vs other spiking simulators

| System | Language | Focus | Structural analysis during simulation? | Causal-perturbation gate test? | Verdict |
|---|---|---|---|---|---|
| **Connectome OS** | Rust | Connectome runtime + structural OS | ✅ live Fiedler + mincut + SDPA motif + stratified null | ✅ AC-5 σ-separation shipped | What this repo is |
| **Brian2** | Python + C++ codegen | LIF simulation for research | ❌ (external post-processing) | ❌ | Fast-enough simulator; no structural layer |
| **NEST** | C++ (+ MPI) | Scale-out spiking simulation | ❌ | ❌ | Widely cited; scale-first |
| **Auryn** | C++ | Hand-tuned single-node event-driven | ❌ | ❌ | Aspirational throughput ref |
| **GeNN** | C++ / CUDA | GPU code generation | ❌ | ❌ | Throughput via GPU |
| **Human Connectome Project tools** | varied | Static connectome visualisation | Static only | ❌ | Viewer, not runtime |
| **CEBRA** | Python / ML | Learned spike-train embedding | Embedding only | ❌ | Downstream analysis tool |

The differentiator isn't throughput (Brian2 + GeNN win that on their preferred axes). It's the category: *Connectome OS is the only system in the list that runs a connectome simulation, **watches its structural coherence live, cuts load-bearing edges on demand, and measures σ-separation against paired controls** as a first-class acceptance criterion.*

### Performance (Ryzen-class CPU, single thread)

| Regime | Connectome OS | Brian2 (published) | Auryn (published) | NEST (published) |
|---|---|---|---|---|
| Sparse per-step (N=1024, 10 ms simulated) | **~7.6 M spikes/sec** | ~50–200 K | ~300–500 K | ~100–300 K |
| Saturated (N=1024, 120 ms simulated) | ~29 K wallclock (slower; detector dominates — see ADR §16) | same published range | same | same |

The Brian2/Auryn/NEST numbers are *published* ranges. We have not re-run them in the same sandbox against the same stimulus — documented caveat in [`BASELINES.md`](https://github.com/ruvnet/RuVector/blob/research/connectome-ruvector/examples/connectome-fly/BASELINES.md).

### Feasibility tiers

| Tier | Scope | Neurons | Status |
|---|---|---|---|
| **Tier 1** | Fruit fly (`Drosophila`), partial mouse cortex | 10⁴ – 10⁵ | **Shipped.** `examples/connectome-fly/` is the Tier-1 demonstrator; FlyWire v783 ingest is fixture-tested. |
| **Tier 2** | Larger mouse regions, multi-region | 10⁵ – 10⁶ | Feasible; ~29 engineer-weeks in the implementation plan. Memory dominated by synapses; requires SSD-backed graph + aggressive sparsification. |
| **Tier 3** | Full mammalian / human brain | 10⁹ – 10¹¹ | **Not feasible at any horizon in this project.** Compute insufficient *and* biological parameters don't exist *and* the connectome doesn't exist. Explicit non-goal. |

---

## Quick start

```bash
# Clone the code repo (Connectome OS lives on a research branch of RuVector).
git clone -b research/connectome-ruvector https://github.com/ruvnet/RuVector.git
cd RuVector

# Build + run all tests (single thread, release profile).
cargo test -p connectome-fly --release

# Run the demonstrator — synthetic 1024-neuron fly-like connectome,
# 500 ms simulation, JSON report on stdout.
cargo run --release -p connectome-fly --bin run_demo

# Write the report to a file instead.
cargo run --release -p connectome-fly --bin run_demo -- /tmp/demo.json

# Run the benchmarks.
cargo bench -p connectome-fly --bench lif_throughput
cargo bench -p connectome-fly --bench sim_step
cargo bench -p connectome-fly --bench motif_search
cargo bench -p connectome-fly --bench opt_d_isolation
```

Expected output from `run_demo`: a ~1 KB JSON document with `connectome` stats, `simulation` spike totals, `coherence.events_total`, a `partition` with class histograms, and `motifs[]` with top-k repeated spike-window patterns.

**Hardware requirements:** any modern x86_64 CPU with AVX2 (single-threaded; no GPU required). Memory: ~4 GB peak for the Tier-1 demonstrator; ~2 GB for the connectome itself.

---

## User guide

### Running a simulation

```rust
use connectome_fly::{Connectome, ConnectomeConfig, Engine, EngineConfig, Observer, Stimulus};

// 1. Build (or load) a connectome.
let conn = Connectome::generate(&ConnectomeConfig::default());
// Alternatively: let conn = connectome_fly::connectome::flywire::load_flywire(path)?;

// 2. Set up an engine. Default config = wheel+SoA+SIMD optimized path.
let mut eng = Engine::new(&conn, EngineConfig::default());

// 3. Build a stimulus — deterministic current injection into sensory neurons.
let stim = Stimulus::pulse_train(
    conn.sensory_neurons(),
    /* start_ms */ 100.0,
    /* duration_ms */ 200.0,
    /* charge_pa */ 85.0,
    /* period_ms */ 120.0,
);

// 4. Run, collecting spikes + coherence events via the observer.
let mut obs = Observer::new(conn.num_neurons());
eng.run_with(&stim, &mut obs, /* t_end_ms */ 500.0);

let report = obs.finalize();
println!("spikes: {}", report.total_spikes);
println!("coherence events: {}", report.coherence_events.len());
```

### Analyzing live spikes

```rust
use connectome_fly::{Analysis, AnalysisConfig};

let an = Analysis::new(AnalysisConfig::default());

// Structural partition — mincut on the static connectome.
let structural = an.structural_partition(&conn);

// Functional partition — mincut on the coactivation-weighted connectome.
let functional = an.functional_partition(&conn, obs.spikes());

// Motif retrieval — SDPA-embedded 100 ms windows, top-5 repeated.
let (index, hits) = an.retrieve_motifs(&conn, obs.spikes(), /* k */ 5);
```

### Running a causal perturbation

```rust
// Identify boundary edges between the two halves of the functional partition.
let side_a: std::collections::HashSet<u32> = functional.side_a.iter().copied().collect();
let mut boundary: Vec<usize> = Vec::new();
let row_ptr = conn.row_ptr();
for pre in 0..conn.num_neurons() {
    for flat in row_ptr[pre] as usize..row_ptr[pre+1] as usize {
        let post = conn.synapses()[flat].post.idx();
        if side_a.contains(&(pre as u32)) != side_a.contains(&(post as u32)) {
            boundary.push(flat);
        }
    }
}

// Cut the top 100 boundary edges.
let cut = conn.with_synapse_weights_zeroed(&boundary[..100]);

// Re-run the same stimulus against the cut connectome and compare to the
// baseline. Population-rate divergence = the structural-causality signal.
// See `tests/acceptance_causal.rs::ac_5_causal_perturbation` for the full
// paired-trial protocol with σ-separation gate.
```

### Adaptive detect cadence

The Fiedler coherence detector automatically backs off from 5 ms to 20 ms cadence when the co-firing window density exceeds 100 Hz per neuron. **This is always-on** — no config required. It is what delivers the 4× saturated-regime throughput win (commit 10). See ADR-154 §16.

### Feature flags

| Flag | Default | What it does |
|---|---|---|
| `simd` | ✅ on | `wide::f32x8` vectorized subthreshold LIF. Falls back to scalar on hosts without AVX. |
| `gpu-cuda` | off | Opt-in GPU SDPA path via `cudarc`. Panics with an actionable diagnostic if the CUDA toolkit isn't linked. |

---

## Fly-AI guide

This section is the workflow for building a **fly-AI system** — a complete pipeline from real FlyWire connectome to a running, probe-able simulation. It's what the architecture is *for*.

### Step 1 — ingest the wiring

```rust
use connectome_fly::connectome::flywire::load_flywire;

// Point at a directory of FlyWire v783 TSVs:
//   neurons.tsv         (~139 k rows)
//   connections.tsv     (~50 M rows)
//   classification.tsv  (cell types)
let conn = load_flywire(std::path::Path::new("/data/flywire-v783/"))?;
println!("loaded {} neurons, {} synapses", conn.num_neurons(), conn.synapses().len());
```

Real FlyWire v783 is ~2 GB unpacked. For memory-efficient ingest, use `load_flywire_streaming` (same API, 4.5 GB → 1.7 GB peak). For development without downloading, see `src/connectome/flywire/fixture.rs` — a 100-neuron hand-authored fixture that exercises the full parse path.

Schema: [ADR-154 §6](https://github.com/ruvnet/RuVector/blob/research/connectome-ruvector/docs/adr/ADR-154-connectome-embodied-brain-example.md) + `src/connectome/flywire/schema.rs`.

### Step 2 — drive it with a stimulus

Out of the box you get [`Stimulus::pulse_train`](https://github.com/ruvnet/RuVector/blob/research/connectome-ruvector/examples/connectome-fly/src/stimulus.rs). For more specific protocols — visual stimuli, olfactory pulses, optogenetic activation — compose `CurrentInjection` events directly.

For a **closed-loop embodied setup** (real proprioception + vision + motor output), you need a body simulator. The architecture reserves this slot for MuJoCo 3 + NeuroMechFly v2 via a thin `cxx` bridge. This is Phase 3 of the implementation plan ([`docs/research/connectome-ruvector/04-embodiment.md`](https://github.com/ruvnet/RuVector/blob/research/connectome-ruvector/docs/research/connectome-ruvector/04-embodiment.md)) and is not shipped yet — today's Tier-1 demonstrator stubs embodiment with deterministic current injection.

### Step 3 — watch the structural signal

The Fiedler coherence-collapse detector runs every 5 ms of simulated time (backed off to 20 ms when the network saturates). When the `λ₂` of the co-firing graph drops below a rolling baseline, the detector emits a `CoherenceEvent`. In the Tier-1 demonstrator, this signal precedes constructed behavioural failures by ≥ 50 ms on 70 %+ of trials (AC-4-strict).

```rust
for event in &report.coherence_events {
    println!(
        "t={:.1}ms  fiedler={:.4}  pop_rate={:.1} Hz",
        event.t_ms, event.fiedler, event.population_rate_hz
    );
}
```

### Step 4 — probe the structure

The load-bearing operation: identify mincut-surfaced edges, cut them, measure what breaks. This is the **operational definition of structural causality** — AC-5 in ADR-154. The σ-separation test requires:

1. Baseline run — no cuts, record late-window population rate.
2. Targeted cut — remove top-K mincut boundary edges, re-run, record divergence.
3. Control cut — same K edges but degree-stratified random, re-run, record divergence.
4. Assert `z_cut ≥ 5σ` and `z_rand ≤ 1σ`.

The `src/connectome/stratified_null.rs` module gives you the degree-stratified sampler out of the box. On the Tier-1 SBM at N=1024 the stratified null collapses (the functional boundary overlaps the high-degree hubs at this scale — documented finding). At FlyWire v783 scale (~139 k neurons with a heavier non-hub tail) it is expected to separate; that rerun is the natural Day-1 validation for a fly-AI system and is the **sole remaining lever for AC-2** after the encoder, corpus-size, and label axes were all empirically ruled out on the synthetic SBM (findings #10, #12, #13 in the ADR).

### Step 5 — retrieve behavioural motifs

Repeated spike-raster patterns in 100 ms windows get embedded via SDPA and indexed for nearest-neighbour retrieval. On a heterogeneous substrate (real FlyWire, not synthetic SBM) the retrieval cross-references behavioural motifs with structural boundaries — which motifs precede which cuts, which modules generate which motifs, does lesioning module X eliminate motif M. This is the full closed loop: structure → dynamics → motif vocabulary → targeted perturbation → structural claim.

### What's missing from a complete fly-AI

| Deliverable | Status |
|---|---|
| FlyWire v783 TSV ingest | ✅ fixture-tested; real 2 GB data fetch is left to the user |
| Streaming ingest (memory-efficient) | ✅ in-tree |
| Event-driven LIF engine | ✅ with SIMD + delay-CSR + adaptive detection |
| Fiedler coherence detector | ✅ Jacobi (n ≤ 96) + dense power (n ≤ 1024) + sparse (n > 1024) |
| Mincut structural partition | ✅ via `ruvector-mincut::canonical::exact` |
| SDPA motif retrieval | ✅ brute-force kNN; DiskANN upgrade documented as follow-up |
| σ-separation causal test | ✅ AC-5 shipped at 5.55σ on SBM |
| Degree-stratified null at FlyWire scale | ⏳ sampler shipped; needs real data |
| **MuJoCo 3 + NeuroMechFly body** | ❌ Phase 3 of the implementation plan; separate ADR |
| Visual-system stimulus module | ❌ Phase 3 |
| Cross-path bit-exact determinism | ⏳ named follow-up |
| Leiden refinement phase for AC-3a | ⏳ multi-level Louvain baseline shipped; refinement is future work |

The code for the checkmarks is in [`examples/connectome-fly/`](https://github.com/ruvnet/RuVector/tree/research/connectome-ruvector/examples/connectome-fly). The deferred items have their engineering estimates in [`docs/research/connectome-ruvector/08-implementation-plan.md`](https://github.com/ruvnet/RuVector/blob/research/connectome-ruvector/docs/research/connectome-ruvector/08-implementation-plan.md).

---

## Measurement-driven findings

Connectome OS ships with **13 measurement-driven discoveries** preserved in ADR-154 §17. Each is attached to the commit that produced it and the lesson it encoded. The pattern is deliberate: every "next lever named in the ADR" was empirically tested; six of the ten pre-measurement diagnoses were disproven when measured.

| # | Finding | Lesson |
|---|---|---|
| 1 | Degree-stratified AC-5 null collapses at N=1024 SBM | Null formulation that matches structure too closely collapses the signal |
| 2 | SIMD saturated gain = 1.013×, not ≥ 2× target | Adding lanes to a loop that rarely executes gives nothing |
| 3 | Observer buffer-reuse is 3 % slower than calloc | OS-zeroed calloc pages beat explicit-loop zeroing for cold allocations |
| 4 | Fiedler detector dominates saturated bench by ~450:1 | Pre-measurement "kernel-bound" diagnosis was wrong — detector was the actual cost |
| 5 | Sparse-Fiedler threshold drop 1024 → 96 is a 3× regression | The sparse path is a scale win at n ≥ 10 000, not a demo-size speed win |
| 6 | **Adaptive detect cadence hits 4.29× — first ≥ 2× win** | **Change *when* the detector runs, not *what* it does.** The 14-LOC heuristic beat every structural change attempted. |
| 7 | Standard full-reorthog Lanczos converges on `λ_max`, not `λ₂` | "Use Lanczos" is not a drop-in replacement without shift-and-invert or deflation |
| 8 | DiskANN at 605-window corpus scored 0.551 vs brute-force's 0.60 | The AC-2 gap is not index-algorithmic; it's corpus structure |
| 9 | BTreeMap incremental Fiedler is 5.8× slower at top-line | Algorithmic complexity doesn't beat constant factors at this scale |
| 10 | Expanded 8-protocol labeled corpus still sub-random precision@5 | The encoder-substrate pairing is protocol-blind; encoder / substrate / labels all need testing |
| 11 | Multi-level Louvain collapses to one community on hub-heavy SBMs | This is exactly what Leiden's refinement phase was introduced to fix |
| 12 | Rate-histogram and SDPA tie at sub-random — **encoder axis ruled out** | The crudest encoder that preserves all raster info ties the engineered one; encoder isn't the bottleneck |
| 13 | Raster-regime labels collapse to 92 % monoculture — **labels axis ruled out** | The substrate saturates every window into the same raster regime; labels can't rescue precision |

The deepest generalisable insight: **when several structurally-different remediations all miss the same target, the target is on a different axis than the ones being searched.** That insight validated item 6 (adaptive cadence — orthogonal axis of *when*) and is the reason items 12 and 13 could triangulate AC-2's real problem (substrate, not encoder / corpus / labels).

Of the three AC-2 axes in the §13 framing, encoder (item 12), corpus (item 10), and labels (item 13) are empirically ruled out. **Substrate is the sole remaining lever** — real FlyWire v783 ingest replacing the synthetic SBM. That's a data-ingest engineering task, not a research pivot.

---

## What Connectome OS is *not*

Connectome OS's positioning rubric holds across every commit on the branch. These claims are explicit non-goals:

- **Not a model of consciousness.** The system produces spike trains, population rates, coherence events, and partition summaries. Nothing more is claimed.
- **Not mind upload.** Running the wiring is not running the mind.
- **Not AGI.** Connectome-constrained biological simulation is orthogonal to general-purpose reasoning systems.
- **Not a substrate-independent intelligence claim.** The system runs on the connectome given to it. Swap the connectome and you run a different system.
- **Not a FlyWire viewer.** The code operates on connectome data; it does not visualise it.
- **Not Tier-3 scale.** Full mammalian / human brain is explicitly out of scope at any horizon. Compute is insufficient, biological parameters don't exist, and the connectome doesn't exist.

See [`07-positioning.md`](https://github.com/ruvnet/RuVector/blob/research/connectome-ruvector/docs/research/connectome-ruvector/07-positioning.md) for the full hype-avoidance rubric.

---

## Contributing

- **Issues & design discussion:** [ruvnet/RuVector issues](https://github.com/ruvnet/RuVector/issues), tag `connectome`
- **Architecture decisions:** ADR-154 on the research branch. New scope additions require a new ADR, not an edit to 154.
- **Positioning guard:** every PR passes the §07-positioning.md rubric — no consciousness / upload / AGI language, anywhere.
- **Threshold discipline:** ADR §14 forbids weakening test thresholds to force a green. If a SOTA target is missed, the gap is recorded in BENCHMARK.md, not relaxed in the test.
- **Honest numbers beat hyped numbers every time.** Thirteen measurement-driven discoveries on this branch, six of which directly disproved pre-measurement pitches, preserved precisely because of this rule.

---

## License

Rust source in [ruvnet/RuVector](https://github.com/ruvnet/RuVector) is under the project's existing license; see the parent repo for the canonical terms.

---

*Connectome OS — cut the wiring, measure the fracture, ask what structure made the failure inevitable.*
