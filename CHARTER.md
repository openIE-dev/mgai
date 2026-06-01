# MGAI Charter

This document distinguishes what the MGAI substrate **commits to forever**
from what it **happens to do today**. Invariants are unconditional. Anything
dated is contingent — a current implementation choice that can change without
breaking the substrate's identity.

---

## Invariants

The substrate does not exist if any of the following is violated.

### I1 — Energy is the unit of account

Every useful decision is priced. The unit is joules — the same unit a watt-hour
meter measures. Cost is not tokens, not dollars, not latency. When a live
energy counter is available on the substrate, it is read. When one is not
available, the cost is reported as an explicitly-labeled
`tdp_estimate + landauer_floor` envelope. The substrate never silently
substitutes one method for the other.

### I2 — Cascade structure: cheap-deterministic before expensive-stochastic

A request is resolved at the lowest tier whose grammar covers it. Tiers run
in order; a higher tier fires only when the lower tier cannot place the
request. The substrate refuses to invert this order, including for latency
or convenience.

### I3 — Replayability classification: V-classes

Every output carries a replayability class:

- **L0-closed** — deterministic compute; identical inputs → identical outputs.
- **L0-retrieved** — exact citation; carries `source_uri` + `retrieval_timestamp`.
- **L0-identifiable** — output of an identifiable encoder (e.g. perceptual hash, HDC item-memory cleanup); replayable given encoder version + input.
- **L1** — citation-backed composition; carries citations to L0-retrieved sources.
- **L2** — model-generated; not bit-replayable; carries epistemic mode + axes.

V-class is structural, not stylistic. A model-generated paragraph cannot be
relabeled L0 by formatting.

### I4 — Claims carry attribution by grammar

A claim that escapes the substrate without attribution is a substrate bug,
not a presentation issue. Attribution is enforced at the *type* level
(`Claim<T>` wraps every claim with `ClaimAttribution { v_class, source, axes,
epistemic_mode, ... }`) so unmarked claims cannot be constructed. Removing
attribution at the audit boundary is a named operation
(`strip_for_internal_use`), never an implicit conversion.

### I5 — Receipts are first-class

Every response carries an energy receipt. Receipts are machine-readable,
serialize stably, and remain valid offline. A response without a receipt is
not a response.

### I6 — Honesty about method

When a primitive's energy is estimated rather than measured, the receipt
labels the method (`tdp_estimate + landauer_floor`) and exposes the impedance
term μ = E_tdp / E_floor. When live counters exist, they are used and labeled
(`rapl`, `powercap`, `nvml`, `powermetrics`). The substrate never elides
which method produced a number.

### I7 — Hard refusal: no unmarked synthetic claims

If a claim cannot be tied to a retrievable source or a deterministic
computation, it is published as L2 — never as L0 or L1. A claim that cannot
be authored with a valid V-class is not authored. The substrate refuses to
emit; it never confabulates a citation.

---

## Optimization objective

Minimize joules per useful decision, subject to `V_delivered ≥ V_required`.

V_required is the replayability class the request demands (read: a benchmark
needs L0-closed; a citation lookup needs L0-retrieved; a synthesis can accept
L2). Substituting a cheaper tier that doesn't meet V_required is a violation,
not an optimization.

---

## Hard refusals (substrate-level, not policy)

The substrate will refuse, structurally:

1. To emit a claim with V_delivered < V_required.
2. To emit an L0-retrieved claim without `source_uri` + `retrieval_timestamp`.
3. To emit a response without an energy receipt.
4. To upgrade a V-class by relabeling. V-classes can only be earned by
   producing the artifact at that class.
5. To silently substitute energy measurement methods.

---

## Contingent (current implementation, dated)

The following are **today's** choices. They are not invariants and may change.

### 2026-05-30 — Cascade tier representatives

The current `mgai-meter` exercises:

- **L0-closed**: `mgai_world_model::point_flow::ArticulatedBody::actor_flow` on a 1-link revolute body.
- **L0-retrieved**: `HashMap<&'static str, &'static str>` lookup.
- **L0-identifiable**: `mgai_hdc::map::ItemMemory::cleanup` with N atoms (default 8).
- **L1**: `HashMap` returning `Cited { value, source }`.
- **L2**: dense `i32` N×N×N matmul (default N=64).

These are representatives chosen for stability under `cargo test`. The
substrate-as-substrate makes no claim that *these specific functions* are
the canonical L0/L1/L2 — only that the *tiers* are real.

### 2026-05-30 — Measured cascade ratios (Apple Silicon, macOS, no live counter)

L2 / L1 ≈ 8 781× on this host as of 2026-05-30. The ratio is the falsifiable
claim, not the absolute joules. Re-running `mgai-meter` on different silicon
will produce different ratios. The shape (L0 < L1 ≪ L2) is the invariant.

### 2026-05-30 — Routing seam-and-cache (observational)

`ask_core::seam::RoutingSeam` records `(query_shape → resolved coordinate)`
after each successful execute. Today it is observational — it does not
bypass the orchestrator. Convergence rate is exposed as a structural metric
on real traffic. Future work may convert it to authoritative routing once
on-traffic convergence is measured.

### 2026-05-30 — Knowledge axes (7-axis)

`KnowledgeAxes` carries `(domain, vocabulary, time, version, locale,
representation, formalism)`. The schema is contingent; a future revision
may add axes without breaking the invariant that *every claim is anchored
to axes*.

### 2026-05-30 — Substrate-level HDC primitive

`mgai_hdc::map` ships bipolar MAP (bind/bundle/permute/cosine), an
`ItemMemory` with cleanup + margin gate, and an analogy operator
(`transform = a ⊙ b; query.bind(transform)`). HDC is the L0-identifiable
representative. The substrate does not require HDC; it requires *some*
identifiable encoder.

---

## What this document is not

- Not a roadmap.
- Not a marketing surface.
- Not a SOTA comparison.
- Not a benchmark commitment beyond "cascade tiers are real and measurable on commodity silicon."

Invariants survive future implementation rewrites. Dated sections are
expected to be edited as the substrate matures.
