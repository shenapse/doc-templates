# {Component name} – Component Decomposition Record (CDR)

> **Purpose:**
> This document records the *design decisions* behind decomposing a component into subcomponents (subsystems or modules).
> It defines **how and why** the component’s internal structure was divided, documents the **responsibilities** and **interactions** among subcomponents, and establishes **traceability** to the corresponding Component Specification (CS) and Implementation Specifications (IS).
>
> Each section includes a **Goal**, **Writing Guidance**, and an **Example**, maintaining consistency and reproducibility across components.

---

## 1. Overview

**Goal:** Define the context and scope of the decomposition — what component is being split, and why this document exists.

### Writing Guidance

* Identify the **parent component** (the one being decomposed).
* Summarize its **purpose**, **role in the system**, and **complexity factors** motivating decomposition.
* Specify whether this decomposition is **initial** or a **refactor**.
* Keep it conceptual — avoid discussing code-level implementation.

### Example

> **Component:** `RewardComputation`
> This component computes scalar rewards from episodic events during simulation.
> Its responsibilities include validation, aggregation, and normalization of event data.
> Refactor: the prior monolith is being split to improve testability and maintainability.
> It has been decomposed into three subcomponents plus one orchestrator.

---

## 2. Decomposition Rationale & Criteria

**Goal:** Explain *why* the component needs to be decomposed and *what principles* guided that decision.

### Writing Guidance

* Describe the **motivating factors** (e.g., high complexity, coupling, maintainability, scalability).
* State the **criteria** used for decomposition (e.g., Single Responsibility, Interface Isolation, Data Locality).
* Optionally include a short summary of **evaluation metrics** (e.g., expected reduction in dependencies, LOC per subcomponent).

### Example

> **Rationale:**
> The monolithic `RewardComputation` logic became too complex to test and optimize individually.
>
> **Decomposition Criteria:**
>
> * Each subcomponent must encapsulate one stage of the reward computation pipeline.
> * Inter-subcomponent dependencies must be acyclic.
> * Each subcomponent must be individually testable.
> * Shared utilities (e.g., schema validation) are extracted into common modules.

---

## 3. Subcomponent Overview

**Goal:** Provide a clear inventory of all subcomponents (logical groups) and define their responsibilities.

### Writing Guidance

* List all **subcomponents** with brief summaries of their **purpose**, **scope**, and **responsibility boundaries**.
* Avoid overlapping responsibilities.
* For each, include references to their **Implementation Spec (IS)** and **Data Schema**, if applicable.

### Example

| Subcomponent          | Purpose                        | Responsibility                                     | Related Docs                 |
| --------------------- | ------------------------------ | -------------------------------------------------- | ---------------------------- |
| `RewardValidation`    | Validate event data structures | Enforces schema integrity and temporal consistency | IS §3.1, Data Schema §3.2    |
| `RewardAggregation`   | Aggregate validated events     | Computes weighted sums over time                   | IS §3.2, CS §4.2             |
| `RewardNormalization` | Normalize raw reward values    | Applies adaptive scaling and clamping              | IS §3.3, ADR-2025-04         |
| `RewardOrchestrator`  | Coordinate all subcomponents   | Orchestrates the pipeline, aggregates results      | IS §3.4, Interface Spec §2.1 |

---

## 4. Interactions & Dependencies

**Goal:** Describe how subcomponents interact, exchange data, and depend on each other.

### Writing Guidance

* Map out **data flow** and **control flow** among subcomponents.
* Describe **dependency directions** (unidirectional or bidirectional).
* Identify **shared resources** or **common interfaces**.
* If possible, include a simple diagram or bullet list.

### Example

> **Data Flow:**
> `RewardValidation → RewardAggregation → RewardNormalization`
>
> * `RewardValidation` outputs a clean event stream.
> * `RewardAggregation` consumes that stream and produces a raw scalar reward.
> * `RewardNormalization` adjusts that scalar for stability.
>
> Dependencies are strictly unidirectional; orchestration occurs in `RewardOrchestrator`.
> Shared utility functions (e.g., `validate_events`) live in `/common/validation.py`.

---

## 5. Orchestration Model

**Goal:** Describe how coordination among subcomponents occurs — who calls whom, and what controls execution order.

### Writing Guidance

* Define whether orchestration is **centralized** (controller-based) or **distributed** (event-driven).
* Specify the **orchestrator’s role** — is it procedural, reactive, or declarative?
* Mention error propagation and synchronization models at a conceptual level.

### Example

> The orchestration follows a **centralized pipeline model**.
> `RewardOrchestrator` invokes subcomponents sequentially:
>
> 1. `validate(events)`
> 2. `aggregate(events)`
> 3. `normalize(value)`
>
> All exceptions are caught at the orchestrator level and logged under the global `SimulationLogger`.
> Asynchronous expansion is planned for future versions (ADR-2025-07).

---

## 6. Alternatives Considered

**Goal:** Record other possible decomposition structures and justify why they were rejected.

### Writing Guidance

* List 2–3 alternative decomposition patterns.
* Summarize their pros/cons in comparison to the adopted one.
* Mention if any were partially retained for future iterations.

### Example

| Alternative                   | Description                     | Reason for Rejection                                 |
| ----------------------------- | ------------------------------- | ---------------------------------------------------- |
| **Monolithic design**         | All logic in one file           | Unmanageable test scope, poor maintainability        |
| **Two-stage design**          | Validation + Aggregation merged | Normalization coupling caused excessive side effects |
| **Event-driven architecture** | Each stage as async subscriber  | Overhead unjustified for current data volume         |

---

## 7. Traceability Mapping

**Goal:** Maintain explicit links between subcomponents, specifications, and implementation documents.

### Writing Guidance

* Map each subcomponent to its **CS section**, **IS document**, and **related Interface Specs**.
* Include document version or ID where available.
* If subcomponent boundaries align with code modules, mention filenames.

### Example

| Subcomponent          | CS Section | IS Document                  | Interface Spec      | Schema / Config  |
| --------------------- | ---------- | ---------------------------- | ------------------- | ---------------- |
| `RewardValidation`    | CS §4.2.1  | `IS.reward_validation.md`    | –                   | Data Schema §3.2 |
| `RewardAggregation`   | CS §4.2.2  | `IS.reward_aggregation.md`   | –                   | –                |
| `RewardNormalization` | CS §4.2.3  | `IS.reward_normalization.md` | –                   | Config Plan §2.1 |
| `RewardOrchestrator`  | CS §4.2.4  | `IS.reward_orchestrator.md`  | Interface Spec §3.1 | –                |

---

## 8. Impact & Future Revisions

**Goal:** Capture the expected impact of this decomposition and how future updates should be managed.

### Writing Guidance

* Describe how the new structure affects **testing**, **performance**, and **maintainability**.
* Note anticipated **extension points** or **refactoring risks**.
* Define a **revision policy** — e.g., when to update the CDR vs. create a new ADR.

### Example

> **Impact:**
>
> * Each subcomponent can now be unit tested in isolation.
> * Reward computation latency reduced by 12%.
>
> **Revision Policy:**
>
> * Minor adjustments (internal refactors) are logged in IS.
> * Major structural changes (e.g., adding a new subcomponent) require CDR v2.
> * Any deviation from decomposition criteria triggers an ADR update.

---

## Summary Table

| Section                        | Focus         | Abstraction    | Purpose                                    |
| ------------------------------ | ------------- | -------------- | ------------------------------------------ |
| 1. Overview                    | Context       | Conceptual     | Define what is decomposed                  |
| 2. Rationale & Criteria        | Design intent | Analytical     | Explain why and how decomposition was done |
| 3. Subcomponent Overview       | Inventory     | Structural     | List subcomponents and their boundaries    |
| 4. Interactions & Dependencies | Collaboration | Architectural  | Describe relationships and data flow       |
| 5. Orchestration Model         | Coordination  | Behavioral     | Explain control structure                  |
| 6. Alternatives                | Comparison    | Reflective     | Preserve rejected options                  |
| 7. Traceability                | Linking       | Administrative | Ensure document interrelation              |
| 8. Impact & Future Revisions   | Evolution     | Strategic      | Describe consequences and update policy    |

---

> **Relationship to Other Documents**
>
> * **Component Specification (CS):** Defines *what* the component does.
> * **Component Decomposition Record (CDR):** Defines *how it is structured internally*.
> * **Implementation Specification (IS):** Defines *how each subcomponent is implemented*.
> * **Interface Specification:** Defines *how subcomponents communicate externally*.
> * **ADR:** Captures *why* certain decomposition or orchestration decisions were made.

---
