## Comment / Rebuttal: Concerns on “Unified Signal & Metrology Standards for Analog AI Hardware”

### Purpose

This submission supports the committee’s objective—**interoperability, repeatability, and comparability**—but raises concerns about *how* unification is being pursued. In brief:

* We should **not standardize a single encoding, format, or representation** prematurely.
* We should standardize **how characteristics are specified, measured, and reported**: operating envelopes, distributions, uncertainty, and environmental dependencies.
* “Conformance” should emphasize **completeness and traceability of characterization**, not pass/fail compliance against a narrow digital-style profile.

---

## 1) Core Concern: Premature “Format Selection” Will Freeze Innovation

Analog AI components are still an active research frontier. If a standard selects (explicitly or implicitly) a particular encoding or “preferred” signal form, the standard becomes a **technology bet**, not a neutrality layer.

**Recommendation:** Unify *descriptions and measurement*, not *implementations*.
Treat the standard as a **descriptor + metrology framework** that can support multiple signal/encoding families without privileging one.

---

## 2) Core Concern: Discrete Types (“A/B”) Are the Wrong Abstraction

Many analog behaviors are not naturally categorical. They are:

* **Continuous** (curves, envelopes)
* **Stochastic** (spread, drift, tails)
* **Contextual** (depends on temperature, supply, history, loading, time)

A digital-style taxonomy (“Type A / Type B compliant”) will erase the engineering truth and incentivize vendors to optimize toward a checkbox.

**Recommendation:** Component “ratings” should resemble mature materials / tube-era datasheets:

* **Transfer curves** with bounds
* **Histograms / distributions**
* **Worst-case envelopes**
* **Parameter sensitivity surfaces** (vs environment and time)
* **Measurement uncertainty and repeatability**

---

## 3) Metrology Must Be Statistical and Time-Aware

Analog AI performance is dominated by effects that only appear when you measure:

* **Population variance** (device-to-device, array-to-array)
* **Temporal drift** (minutes → months)
* **History dependence** (cycling, prior states, conditioning)
* **Tail risk** (rare failures matter in deployment)

A standard that reports only means/typicals will produce irreproducible comparisons.

**Recommendation:** Require reporting of:

* **Distributions (not just averages)**
* **Quantiles** (P50/P90/P99) and/or confidence intervals
* **Sample sizes, sampling method, and selection criteria**
* **Time-axis characterization** (short/medium/long horizons)
* **Aging / cycling curves** where applicable

---

## 4) Standardized Metrics Invite Gaming (Goodhart Risk)

Once a metric becomes compliance, systems will be tuned to pass the metric rather than to be robust in real deployment. This is especially acute in analog domains, where “passing” can be achieved by shaping behavior within a narrow test window.

**Recommendation:** Conformance should include:

* **Adversarial / stress testing** (environmental sweeps, corner cases)
* **Round-robin interlab reproducibility requirements**
* **Disclosed measurement harness and calibration traceability**
* A “minimum reporting set” that makes gaming detectable

---

## 5) Interoperability Comes From Declaring an Operating Envelope

Interoperability does not require identical encodings. It requires a **declared and testable compatibility envelope**.

**Recommendation:** Define compatibility as:

* A **declared signal/response envelope** (ranges, dynamics, bandwidth, tolerances)
* A **machine-readable “capability declaration”** (schema)
* A **standard method** for verifying that declaration

This supports multiple approaches while still enabling integrators to design systems predictably.

---

## 6) What “Unified” Should Mean in This Standard

“Unified” should not mean “one way.” It should mean:

1. **Unified vocabulary** (what we call things)
2. **Unified test methods** (how we measure them)
3. **Unified reporting schema** (how results are published)
4. **Unified comparison rules** (how to compare apples-to-apples)

If those four exist, implementations can remain diverse while still being comparable and integratable.

---

## 7) Proposed Deliverables for a Standards-First Approach

### A. Analog AI Component Characterization Report (Template)

A normative template requiring:

* Measurement conditions (temp, supply, loading, timing, calibration)
* Uncertainty and instrumentation specs
* Population sampling method
* Plots + data tables for required curves/distributions

### B. Capability Declaration Schema (Machine-Readable)

A versioned schema (e.g., JSON/YAML) for:

* Operating ranges and limits
* Statistical descriptors (quantiles, CI, sample size)
* Drift/aging descriptors (time constants, bounds)
* Environmental sensitivities
* Known failure modes / guard bands

### C. Conformance Levels = Reporting Completeness

Instead of Type A/B device categories, define levels such as:

* **Level 1:** Minimal envelope + uncertainty + reproducibility metadata
* **Level 2:** Adds distributions, quantiles, environmental sweeps
* **Level 3:** Adds time/aging/cycling + interlab round-robin evidence

This creates a clean on-ramp without forcing architectural choices.

---

## 8) Requested Action from the Committee

1. Reframe the current proposal to **avoid selecting a canonical encoding**.
2. Prioritize **characterization + reporting standards** as the first publishable output.
3. Establish a working subgroup focused on:

   * Statistical metrology requirements
   * Reporting schema + templates
   * Interlab reproducibility (round-robin plan)

---

## Closing

A standards effort can be enormously valuable here—**if it standardizes the language of measurement and declaration rather than the internal representation of meaning**. The field will benefit most from a framework that makes systems *comparable and integratable* while preserving room for innovation across signal modalities and device physics.

**Provenance Credit:** IAAI Analog Working Group (Openware) – synthesis and committee comments prepared for standards review.
