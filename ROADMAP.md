# Test Lab Roadmap

## Goal

Test Lab explores whether a software project can retain an arbitrarily rich body of executable evidence while keeping ordinary change-time validation fast.

The central hypothesis is:

> Preserve abundant evidence, but consume only the evidence that is worth consulting for the current decision.

The project does not try to minimize the number of tests. It experiments with minimizing the amount of evidence executed synchronously while preserving stronger eventual validation.

## Core model

Traditional test level remains one independent axis:

- unit
- integration
- end-to-end

Test purpose is a second axis:

- development / TDD
- feature acceptance
- regression ratchet
- invariant
- compatibility
- benchmark
- diagnostic
- exploratory

Execution policy is a third axis:

- mandatory
- selected
- diagnostic
- sampled
- audit
- stress

A test's level does not determine its purpose or execution frequency.

## Validation relations

Start with explicit relationships rather than inferring them:

- `impacts`: code or behavior whose changes make a test relevant.
- `dominates`: after one test succeeds, another test has too little marginal validation value to justify synchronous execution under the current policy.
- `diagnoses`: a test is useful for localizing another test's failure.

`dominates` is intentionally pragmatic rather than logical subsumption. A dominated test can retain unique evidence and remain valuable for diagnosis, sampling, or audit.

## Execution modes

### DEV

Fast feedback around behavior currently under development. TDD-oriented tests may be especially active here.

### FRONTIER

Default change-time validation:

1. identify impacted evidence;
2. keep mandatory evidence;
3. remove tests dominated for this policy;
4. execute the resulting validation frontier.

### DIAGNOSE

Triggered after a failure. Expand into narrow ratchets, lower-level checks, sibling evidence, and other tests that help localize the failure.

### SAMPLE

Run the mandatory/frontier set plus a reproducibly sampled subset of the remaining corpus.

### AUDIT

Run the broad or complete corpus. Initially this is the reference oracle used to evaluate selector quality.

### STRESS

Run deliberately expensive evidence such as fuzzing, large property campaigns, race/sanitizer checks, parameter sweeps, long simulations, broad browser matrices, and heavy performance experiments.

## Milestone 1 — Adaptive validation prototype

Build a small synthetic application with enough structure to support unit, integration, and end-to-end tests.

Target roughly 30 initial tests with metadata for:

- level;
- purpose;
- impacts;
- explicit dominance;
- diagnostic relationships.

Implement `frontier`, `diagnose`, and `audit`.

Initially execute both FRONTIER and AUDIT for experiments. AUDIT is the ground truth; FRONTIER is only a candidate optimization.

Success criteria:

- tests remain independently runnable;
- FRONTIER selection is deterministic for the same declared inputs;
- failure expansion is reproducible;
- reports explain why each test ran or was skipped;
- FRONTIER and AUDIT decisions can be compared automatically.

## Milestone 2 — Make selection observable and self-correcting

Record enough evidence to explain selector behavior.

Track at least:

- total corpus size;
- selected and skipped counts;
- frontier and audit runtime/cost;
- failures found by FRONTIER;
- failures found only by AUDIT;
- diagnostic expansion size;
- delayed detection time;
- reason each test was selected, skipped, or expanded.

Treat an audit-only failure as two defects:

1. an implementation defect;
2. a validation-selection defect.

Classify selector misses, for example:

- missing impact edge;
- incorrect dominance relation;
- unexpected cross-component dependency;
- unique oracle information;
- stale historical relevance.

Add regression protection for selector mistakes so the validation system ratchets along with the software.

## Milestone 3 — Automatic impact analysis and empirical dominance

Gradually replace hand-maintained impact metadata with experiments using:

- file and module dependencies;
- call graphs;
- runtime coverage;
- checked/assertion coverage;
- changed public APIs;
- dependency traversal;
- historical co-failure data.

Keep explicit metadata as an override.

For dominance, collect pairwise evidence such as:

- executions together;
- failures of each test;
- overlapping failures;
- unique failures;
- execution cost;
- diagnostic usefulness.

Start with recommendations such as “B appears to dominate A for FRONTIER”; do not automatically discard tests or make irreversible decisions from weak evidence.

## Milestone 4 — Probabilistic validation

Add SAMPLE mode.

Start with uniform sampling only as a baseline, then compare weighted policies.

Random selection must be reproducible, for example by deriving the seed from the source revision.

Candidate weighting signals:

- impact relevance;
- dependency proximity;
- historical failure usefulness;
- time since last execution;
- novelty of changed code;
- uniqueness of the test's evidence;
- recent instability;
- previous selector misses;
- execution cost;
- deliberate random exploration.

Prevent starvation by increasing selection pressure with test age or by defining a maximum time between executions.

The system should report measurable residual risk rather than pretending that a sample is a proof.

## Milestone 5 — Validation budgets and risk classes

Explore choosing evidence under an explicit budget instead of a fixed suite.

Possible interfaces:

- runtime budget;
- maximum work budget;
- confidence/risk profile;
- explicit mode.

The optimization target is conceptually:

> maximize expected validation value subject to the available budget.

Not all evidence is eligible for eventual validation. Define risk classes such as:

- must-hold — always checked;
- high-risk — strong affected-change frontier;
- ordinary — adaptive frontier;
- low-risk or expensive — sample/audit/stress.

Compiler/type correctness, cheap schema checks, security-critical rules, protocol compatibility, and similarly cheap/high-risk invariants are candidates for synchronous enforcement.

## Milestone 6 — Evidence lifecycle, diversity, and performance

Treat test origin and lifecycle as first-class data.

A TDD test may be highly active during implementation and later become mostly diagnostic or audit evidence. A bug-derived regression test may be long-lived historical evidence even when rarely selected synchronously.

Retain provenance such as the issue or failure that motivated a test.

Model oracle diversity separately from test count. Candidate oracle kinds include:

- exact expected output;
- property/invariant;
- reference implementation;
- differential comparison;
- snapshot/golden;
- browser/system observation;
- protocol round-trip;
- external implementation;
- deterministic work counter;
- performance baseline.

Prefer independent observers over many tests that encode the same misunderstanding.

Treat correctness and performance evidence uniformly where useful. Historical issues may leave behind correctness ratchets, benchmark scenarios, allocation/work counters, browser scenarios, and compatibility checks with different execution policies.

## Milestone 7 — Historical replay and usefulness scoring

Use repository history as experimental data.

For historical changes:

1. replay the change;
2. run FULL/AUDIT;
3. run candidate selectors;
4. compare detection, cost, and diagnosis.

Compare policies such as:

- impact only;
- impact + dominance;
- impact + uniform sampling;
- impact + weighted sampling;
- historical-failure weighting;
- different budgets and risk profiles.

Estimate marginal usefulness for tests from signals such as:

- execution cost;
- unique failures found;
- diagnostic usefulness;
- overlap with other evidence;
- relevance to changed code;
- age;
- flakiness;
- oracle independence.

Low usefulness should primarily influence execution frequency, not cause automatic deletion.

## Milestone 8 — Large-corpus experiment

Deliberately create a large evidence corpus, including agent-generated evidence.

Scale through at least:

- 1,000 tests;
- 10,000 tests;
- 100,000 tests where practical.

For synthetic issues, allow multiple evidence artifacts such as:

- unit regression test;
- integration scenario;
- end-to-end scenario;
- property/invariant;
- benchmark ratchet;
- diagnostic probe;
- compatibility test.

Measure whether ordinary validation latency remains primarily a function of the change surface rather than total corpus size.

## Milestone 9 — Compare validation architectures

Compare at least:

1. run everything;
2. conventional static suites/layers;
3. evidence corpus + adaptive validation.

Measure:

- synchronous CI latency;
- total compute;
- regressions detected before merge;
- regressions detected eventually;
- mean detection delay;
- diagnosis cost;
- selector complexity;
- maintenance burden;
- agent intervention required.

The goal is to identify the cost/risk frontier rather than assume that one architecture is universally best.

## Milestone 10 — Extract only proven concepts

Do not turn Test Lab into a general testing framework prematurely.

Only extract concepts after experiments demonstrate that they are stable and reusable. Candidate outputs include:

- test metadata schema;
- impact graph;
- validation-frontier selector;
- diagnostic expansion;
- sampling policy;
- audit scheduling;
- evidence metrics;
- reusable historical replay harness.

## Guiding principles

- Do not optimize for a small test corpus; optimize evidence consumed per decision.
- Keep test level, test purpose, and execution policy independent.
- Prefer deterministic and reproducible selection; seeded randomness is still reproducible.
- Keep mandatory correctness properties synchronous.
- Use broader eventual validation where detection latency is an acceptable trade-off.
- A delayed failure should improve both the implementation and the selector.
- Preserve useful issue-derived ratchets and evidence instead of deleting them merely because they overlap.
- Keep simple semantic/reference oracles independent from optimized production mechanics when practical.
- Measure selector misses explicitly; FRONTIER is not allowed to silently redefine truth.
