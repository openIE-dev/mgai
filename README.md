# Math-Ground AI

**Physical AI that tells you what it just cost.**

A library where every call returns a picojoule receipt. Closed-form physics
where it exists. Lyapunov-traced controllers. Derived safety envelopes.
Anytime-valid verification. A typed actuation boundary.

This is the public face of the library. Source is private during the build-out;
this repository ships the **receipt meter** (`mgai-meter`) as a single
self-contained binary plus a written description. The interactive exhibits live
at **[mathground.ai](https://mathground.ai)**.

---

## What this repository contains

```
mgai/
├── README.md     # This document
├── CHARTER.md    # The seven invariants the substrate commits to forever
└── LICENSE       # BSL-1.1 (binary), CC-BY-4.0 (CHARTER.md and receipts)
```

Binaries are attached to **[Releases](https://github.com/openIE-dev/mgai/releases)**.
The first release ships `mgai-meter` for `aarch64-apple-darwin`. Other targets
follow as the substrate is exercised on more silicon.

## What `mgai-meter` does

`mgai-meter` measures the wall-time of one representative operation at each
tier of the cognitive cascade, computes the **Landauer-floor lower bound** for
the energy of that operation (`kT·ln 2 × bit-ops`, exact at 300 K), and the
**TDP-estimate upper bound** (`wall_time × assumed_watts`, declared as an
estimate). The ratio of the upper bound to the lower bound is the **apparent
impedance** μ — how far real silicon sits above physics' floor.

Why this exists: the cascade's energy story turns on a falsifiable claim — that
deterministic primitives at the low tiers cost orders of magnitude less than
model-generated primitives at the high tiers. `mgai-meter` makes that claim
measurable on whichever computer you run it on. If the ratios disappear on
your silicon, the thesis is wrong; the meter is how you check.

Sample output, on an Apple Silicon laptop with no live energy counter
(`tdp_estimate + landauer_floor` method):

```
primitive                          Vclass     wall_ns/op     E_floor/op       E_tdp/op   μ_apparent
------------------------------------------------------------------------------------------------
l0_closed::fk_rollout              l0_closed         79.217    5.878e-6 pJ     633.738 nJ      1.08e11
l0_retrieved::hashmap_lookup       l0_retrieved       7.973    1.469e-6 pJ      63.781 nJ      4.34e10
l0_identifiable::hdc_cleanup       l0_identifiable 2785.000    4.592e-4 pJ      22.280 µJ      4.85e10
l1::cited_lookup                       l1            13.386    2.939e-6 pJ     107.086 nJ      3.64e10
l2::matmul_proxy                       l2         14343.750    1.881e-4 pJ     114.750 µJ      6.10e11

cascade ratios (wall-time):
  L1              / L0-closed   =       0.17×
  L0-identifiable / L0-closed   =      35.16×
  L2              / L0-closed   =     181.07×
  L2              / L1          =    1071.57×

E_measured: null   (no live counter on this substrate)
```

The cascade ratios are **measured wall-time** — real numbers on real silicon.
Absolute joules are bracketed by the Landauer floor (a physical lower bound)
and the TDP envelope (a labelled upper estimate); a live energy counter
(RAPL / powercap / NVML / `powermetrics`) tightens the bracket but is not
required to read the ratios.

## How to read the receipt

A receipt is more honest than a benchmark. The four shapes the library
publishes:

1. **Joule receipt** — every metered call returns its joules, computed at the
   call site against a silicon-specific cost model.
2. **Epistemic-mode receipt** — every claim carries its V-class
   (`L0-closed`, `L0-retrieved`, `L0-identifiable`, `L1`, `L1.5`, `L2`),
   `source`, `axes`, and `epistemic_mode`. A claim that cannot be authored at
   the right V-class is not authored. Unmarked claims cannot be constructed at
   the type level.
3. **Evidence receipt** — anytime-valid e-values + prediction-powered inference
   for any quantity the library asserts.
4. **Routing checkpoint** — byte-reproducible. Two replicas of the cascade
   serving the same command stream land on the same FNV-1a transcript digest;
   mutation, drop, or reorder is detected by digest divergence alone.

See [CHARTER.md](./CHARTER.md) for the seven invariants the substrate commits
to forever, and the dated contingents that are current implementation choices
rather than load-bearing claims.

## The cognitive cascade

A request is resolved at the lowest tier whose grammar covers it. Higher tiers
fire only when the lower tier cannot place the request. The shape that survives
across implementations:

| tier               | example primitive                      | V-class        | μ_apparent (this Mac) |
| ------------------ | -------------------------------------- | -------------- | --------------------- |
| L0-closed          | `point_flow::actor_flow` (1-link body) | L0-closed      | ~10¹¹                 |
| L0-retrieved       | `HashMap<&str,&str>` lookup            | L0-retrieved   | ~10¹⁰                 |
| L0-identifiable    | HDC item-memory cleanup                | L0-identifiable| ~10¹⁰                 |
| L1                 | citation-backed `HashMap<&str,Cited>`  | L1             | ~10¹⁰                 |
| L1.5 (vector tier) | Matryoshka short-circuit (in cascade)  | L1             | —                     |
| L2                 | dense matmul proxy                     | L2             | ~10¹¹                 |

The receipt's two non-negotiables: the **method label** (`measured` vs
`tdp_estimate + landauer_floor`) is always present; the **μ** column shows the
gap to physics. A receipt that hides either is a substrate bug.

## Live, in the browser: mathground.ai

Nine interactive exhibits run the same accounting in WebAssembly on your
device:

| exhibit                                           | what it measures                  |
| ------------------------------------------------- | --------------------------------- |
| [/exhibits/fft](https://mathground.ai/exhibits/fft)         | Cooley–Tukey radix-2 FFT          |
| [/exhibits/fft2d](https://mathground.ai/exhibits/fft2d)     | 2-D FFT via row-then-column passes |
| [/exhibits/lorenz](https://mathground.ai/exhibits/lorenz)   | Lorenz '63 RK4 integrator         |
| [/exhibits/heat](https://mathground.ai/exhibits/heat)       | 2-D heat equation, explicit Euler |
| [/exhibits/grad](https://mathground.ai/exhibits/grad)       | Heavy-ball gradient descent       |
| [/exhibits/pfilter](https://mathground.ai/exhibits/pfilter) | Bootstrap particle filter         |
| [/exhibits/sdf](https://mathground.ai/exhibits/sdf)         | Marching squares on a 2-D SDF     |
| [/exhibits/sdf3d](https://mathground.ai/exhibits/sdf3d)     | 3-D SDF, sphere-traced (WebGL2)   |
| [/exhibits/sudoku](https://mathground.ai/exhibits/sudoku)   | CSP receipts (AC-3 vs Z3 vs brute)|

Companion pages:

- [/scale](https://mathground.ai/scale) — your last frame plotted on a 29-decade log axis between the Landauer floor and a 1 km drive in an EV.
- [/receipts](https://mathground.ai/receipts) — the four receipt shapes, plus a per-device session aggregator that reads `localStorage`.
- [/lineage](https://mathground.ai/lineage) — what mathground inherits from Deep Space 1 Remote Agent and adjacent literature.

## Where we sit — substrate vs vendor

On **1 June 2026**, NVIDIA shipped a coordinated triple drop — new silicon
(RTX Spark), an open world foundation model (Cosmos 3), and an open AV
reasoning VLA (Alpamayo 2 Super) — across Computex and GTC Taipei. Each
demonstrates a capability shape. Each already lives at picojoule grain inside
this site. Four pairings:

### 01 · Alpamayo 2 Super → chain-of-thought trace

| | |
|---|---|
| **The proof** | 32 B-parameter open reasoning VLA for L4 robotaxi development; produces a chain-of-thought trace before acting. |
| **The substrate version** | [`/exhibits/reason`](https://mathground.ai/exhibits/reason) — a small cascade walks each query through five tiers and emits the trace as the chain of thought. Every hop is a deterministic function with its own picojoule receipt. |
| **SWAP-C note** | Zero new compute. The transcript IS the chain of thought. |

### 02 · Cosmos 3 → modality routers, one shape-key space

| | |
|---|---|
| **The proof** | Open omnimodel for physical AI. Mixture-of-transformers; trained on 20 trillion multimodal tokens. Single model handles text/image/video/audio/action. |
| **The substrate version** | [`/exhibits/omnimodal`](https://mathground.ai/exhibits/omnimodal) — four deterministic encoders project text, image, audio, and video into the same HDC shape-key space. A live 4×4 cosine matrix is the proof. No joint training; no shared neural encoder. |
| **SWAP-C note** | ~150 LOC per encoder. No model weights. The omnimodal property is substrate, not learned. |

### 03 · Cosmos 3 (architecture) → MoT cascade split

| | |
|---|---|
| **The proof** | Mixture-of-transformers: a reasoning transformer and an expert generation transformer share a common embedding. Separation of "classify the form" from "synthesise the answer". |
| **The substrate version** | [`/exhibits/reason`](https://mathground.ai/exhibits/reason) — every cascade tier is split into a reasoning sub-expert (recognise the form) and a generation sub-expert (produce the answer). Generation only fires when reasoning matches. Both sub-hops are individually accountable. |
| **SWAP-C note** | A refactor, not new compute. The capability shape transfers directly. |

### 04 · RTX Spark → lazy-loaded L2 leaf

| | |
|---|---|
| **The proof** | NVIDIA's first Windows-PC processor. 20-core Grace + Blackwell GPU + NPU + 128 GB unified. 1 petaFLOP AI; runs 120-billion-parameter models locally. |
| **The substrate version** | [`/exhibits/escalate`](https://mathground.ai/exhibits/escalate) — the cascade resolves most queries at L0/L1 for picojoules. When a query escapes the spine, the page dynamic-imports a separate WASM bundle (the model tier) and pays its byte cost plus per-token inference. Two bundles visible in the browser's Network panel. |
| **SWAP-C note** | L2 leaf only paid for on escalation. Receipt parameterised for a [PrismML 1-bit Bonsai 1.7 B](https://prismml.com/news/ternary-bonsai)-class model. |

### Why this is the only lane left to take

Vendor capability growth is **anti-SWAP-C**: each new generation ships more
power per chip, not less. The hardware-software-joule loop nets out to absolute
energy growth even under exponential compute scaling. The low-SWAP-C envelope
(drones, OpenMV cameras, STM32 sensor nodes, wearables, medical implants)
remains the unsolved joule problem, and it can only be solved at the substrate
level. The receipt is the discriminator: incumbents pay tokens for "show
thinking" verbosity; the substrate emits the trace as a free byproduct of the
cascade.

The full pairings page with the live exhibits is at
[mathground.ai/lane](https://mathground.ai/lane).

## Status

This is a research substrate under active development. The seven invariants in
[CHARTER.md](./CHARTER.md) are commitments; everything else is dated. APIs and
crate boundaries may change without notice. The public binary is the surface
that's intended to stay stable.

## License

The binary is distributed under **BSL-1.1** with conversion to Apache-2.0 at a
date specified per-release. Source remains private during the build-out.
Receipts and `CHARTER.md` are CC-BY-4.0.

## Contact

Open Interface Engineering, Inc.
Delaware B-Corp · Sarasota, FL
[contact@openie.dev](mailto:contact@openie.dev)
