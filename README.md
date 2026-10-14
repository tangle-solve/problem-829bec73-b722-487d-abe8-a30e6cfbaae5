<!-- solve-problem:829bec73-b722-487d-abe8-a30e6cfbaae5 -->
# Adaptive-adversary dynamic-graph benchmark

A [Tangle Solve problem](https://solve.tangle.tools/problems/829bec73-b722-487d-abe8-a30e6cfbaae5).

# Research Brief: Adaptive-Adversary Dynamic-Graph Benchmark (ALGO-000244)

## What is the problem?

Design and build a **replayable benchmark suite for dynamic graph algorithms in which the adversary is adaptive**: the next graph update (edge insertion/deletion) may depend on the algorithm's previous outputs (e.g., answers to connectivity, reachability, matching, or shortest-path queries).

Formal setting: We have a dynamic graph algorithm maintaining graph G on n vertices under an update sequence σ₁, σ₂, … chosen by an adversary A. In the standard *oblivious* setting, σ is fixed in advance; benchmarks (fixed traces) only test this. In the *adaptive* setting, σ_{t+1} = A(σ_{≤t}, o_{≤t}) where oᵢ are the algorithm's outputs up to time t. The task is an *engineering + experimental-design* problem:

1. A trace format that records the adversary's decision rule (or enough state to replay it) so runs are reproducible — including the algorithm's outputs that conditioned the adversary, which makes replay depend on the algorithm under test unless the adversary is itself deterministic given outputs.
2. Per-update **correctness certificates** (e.g., spanning forest / cut certificates for connectivity, witness paths for shortest paths) allowing offline verification of every answer.
3. Instrumented measurement of: worst-case (not amortized) latency per update, number of full rebuilds, peak/total memory, and **random-seed exposure** (how much of the algorithm's internal randomness the adversary could observe or infer — relevant for algorithms whose guarantees hold only w.h.p. against adaptive adversaries if randomness is hidden).

The benchmark should let experimentalists distinguish algorithms with adaptive-adversary analyses from those with only oblivious-adversary analyses — a distinction fixed traces cannot capture.

## Why does it matter?

Nearly all practical dynamic graph algorithms are evaluated on fixed update traces, which model only oblivious adversaries; their published worst-case guarantees may silently fail when updates react to outputs. A standardized adaptive benchmark would (a) expose this gap for practitioners in network routing, databases (view maintenance), and real-time systems needing latency guarantees; (b) give the theory community experimental evidence for the practical cost of the oblivious/adaptive divide (e.g., conjectured-hard problems under the OMv framework); and (c) provide correctness certificates so results in dynamic-algorithms papers become independently checkable rather than trust-based.

## What's already known?

- **Theory side:** The oblivious/adaptive (and "adaptive offline" vs "adaptive online") taxonomy for randomized dynamic algorithms is standard (cf. Henzinger's 2015 unifying framework paper below, which also popularized OMv-based conditional lower bounds). Many algorithms analyzed only against oblivious adversaries are known to fail, in principle, against adaptive ones — typically because the adversary can learn the algorithm's random choices from its outputs and then force a bad event. This gap is documented theoretically but essentially never measured.
- **Benchmark side:** Existing dynamic-graph benchmarks use **fixed traces** (network snapshots, temporal graphs, synthetic generators). None implement output-dependent update generation, and none emit per-answer correctness certificates as a standard artifact. There is (to our knowledge, verify) no public benchmark of this type; this census record proposes it as a new artifact rather than referencing an existing one.
- **Adjacent work:** Fuzzing/stress-testing literature (coverage-guided testing, e.g., AFL-style feedback loops) supplies the engineering pattern for output-dependent input generation; certificate/self-verification ideas exist for static graph algorithms (e.g., verification of MST, connectivity) and can be ported to the dynamic setting.

Why prior efforts fall short: replaying an adaptive trace across different algorithms is ill-defined because the trace itself depends on the tested algorithm's outputs — this is the central unresolved design question.

## What would count as solving it?

A published, open-source artifact providing:

1. **Trace/protocol format** precisely defining the adversary's information set (which outputs, how quantized) and enabling deterministic replay — either by fixing adversary seeds and re-running the adversary logic, or by a "reference algorithm" convention that makes traces algorithm-independent. The spec must resolve the replay-vs-adaptivity tension explicitly.
2. **An adversary generator** with at least 2–3 adaptive strategies (e.g., certificate-invalidation attacks, seed-reconstruction attacks against randomized algorithms) plus oblivious baselines, on multiple graph families.
3. **Certificate-verified correctness**: every query answer accompanied by a checkable witness (spanning forest for connectivity, etc.), verified offline.
4. **Instrumentation** reporting worst-case per-update latency (not just mean/amortized), rebuild counts, memory, and random-seed exposure statistics.
5. **A demonstration study**: run ≥3 established dynamic algorithms (e.g., a dynamic connectivity and a dynamic matching implementation) and show the benchmark discriminates between them in the adaptive setting in a way fixed traces do not — ideally exhibiting at least one concrete algorithm whose oblivious-analyzed guarantees degrade.

Referee criterion: reproducible results, sound adversary model documented as a formal spec, and the demonstration study.

## How to start

1. **Read Henzinger et al. (2015) Section on adversary types and the OMv conjecture** to pin down the three-adversary taxonomy (oblivious / adaptive offline / adaptive online) and which popular algorithms fall in which class — this defines what your adversaries should target.
2. **Write the formal spec before any code**: define the adversary's information set exactly (full outputs? timing? error messages?), since an underspecified model makes all results uninterpretable. Decide the replay convention: either "adversary re-execution" (store adversary state+seed, re-run against each algorithm) or "reference-algorithm traces" (fix traces against a designated baseline, acknowledging cross-algorithm comparison caveats).
3. **Port static verification/certificates to the dynamic setting first for dynamic connectivity** (dynamic spanning forest is itself the certificate — recompute a forest periodically and verify). This is the easiest certificate and the easiest algorithm to bootstrap.
4. **Design one cheap but effective adaptive adversary**: one that reads connectivity answers and deletes a bridge edge on the answer's spanning forest path — a known pathological pattern that fixed traces essentially never contain. Main obstacle throughout: ensuring replayability without sacrificing adaptivity, and isolating timing noise for worst-case latency measurement.

## Key references

- Henzinger, S., Krinninger, S., Nanongkai, D., Saranurak, T. *Unifying and Strengthening Hardness for Dynamic Problems via Reductions from Range Searching*. arXiv:1504.06148 (2015?). (verify exact venue — appeared as approximately this framework; also associated with the OMv conjecture via Henzinger et al., STOC 2015). Introduces/uses the conditional-lower-bound framework (OMv) and the adversary taxonomy used to classify dynamic algorithms. https://arxiv.org/abs/1504.06148
- Christiansen, H. L. (attribution uncertain, verify). *Algorithms for Dynamic Graphs* (2025, per census record). A recent treatment of dynamic graph algorithms; likely a source for current algorithm classes and their assumed adversary models. Not independently verifiable from the record — locate the actual document before relying on it.
- Background (suggested, verify relevance): standard literature on dynamic connectivity (e.g., Holm–de Lichtenberg–Thorup) and on certificate/checking-based verification of graph algorithms, e.g., the "program checking" tradition of Blum–Kannan–Luby for self-testing/correcting algorithms.

## Warnings

- **Replayability vs. adaptivity is the core subtlety**: an adaptive trace depends on the tested algorithm's outputs, so "one trace, many algorithms" is impossible in the strictest sense. Any benchmark must state and justify its convention, or reviewers will (correctly) attack it.
- **Worst-case vs. amortized latency**: many fast dynamic algorithms are amortized; reporting means will hide exactly the failures the benchmark exists to expose. Use per-update maxima and report distributions.
- **Random-seed exposure is ill-defined without care**: an algorithm "exposes" randomness only through observable outputs and timing; define the leakage model formally before measuring it.
- **Don't overclaim conditional-lower-bound connections**: OMv says certain worst-case update bounds are unlikely; it does not predict that specific implementations fail adaptively. Keep theory claims and empirical claims separate.
- The Christiansen (2025) citation comes from the census record and is unverified — confirm it exists and says what is assumed before citing it.

---
*`ALGO-000244` · flagged for review*

## Acceptance guidance

A published, open-source artifact providing:

1. **Trace/protocol format** precisely defining the adversary's information set (which outputs, how quantized) and enabling deterministic replay — either by fixing adversary seeds and re-running the adversary logic, or by a "reference algorithm" convention that makes traces algorithm-independent. The spec must resolve the replay-vs-adaptivity tension explicitly.
2. **An adversary generator** with at least 2–3 adaptive strategies (e.g., certificate-invalidation attacks, seed-reconstruction attacks against randomized algorithms) plus oblivious baselines, on multiple graph families.
3. **Certificate-verified correctness**: every query answer accompanied by a checkable witness (spanning forest for connectivity, etc.), verified offline.
4. **Instrumentation** reporting worst-case per-update latency (not just mean/amortized), rebuild counts, memory, and random-seed exposure statistics.
5. **A demonstration study**: run ≥3 established dynamic algorithms (e.g., a dynamic connectivity and a dynamic matching implementation) and show the benchmark discriminates between them in the adaptive setting in a way fixed traces do not — ideally exhibiting at least one concrete algorithm whose oblivious-analyzed guarantees degrade.

Referee criterion: reproducible results, sound adversary model documented as a formal spec, and the demonstration study.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/829bec73-b722-487d-abe8-a30e6cfbaae5?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: ALGO-000244.

- [Public source catalog](https://drewstone.github.io/problems/#/p/ALGO-000244)
