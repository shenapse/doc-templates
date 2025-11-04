# 🧩 Component-Level Architecture (mini-SLA)

> **Purpose:**
> This document describes the internal architecture of a single component, treating it as a self-contained subsystem.
> It explains how internal modules cooperate, how data moves within the component, and what design assumptions or principles govern its structure.
> Each section includes a **Goal**, **Writing Guidance**, and an **Example**, ensuring consistent conceptual alignment with the System-Level Architecture.

---

## 1. Context within the System

**Goal:** Clarify the component’s role within the larger system and its relationship to neighboring components.

### Writing Guidance

* Describe where this component fits in the overall architecture.
* Identify **upstream** (data sources) and **downstream** (data consumers).
* Explain **why** this component exists — its essential purpose in fulfilling system objectives.
* Avoid code or API references; focus on functional and conceptual relationships.

### Example

> The `TrajectoryPlanner` component operates within the **Navigation Subsystem**.
> It consumes positional and obstacle data from `PerceptionModule` and provides trajectory commands to the `MotionExecutor`.
> Its purpose is to bridge perception and control, ensuring smooth and collision-free motion generation under dynamic environmental conditions.

---

## 2. System-Level View (of the Component)

**Goal:** Present the internal structure of the component from a systems perspective.

### Writing Guidance

* Depict the **submodules** that make up this component and how they interact.
* Focus on **control flow** and **information exchange** within the component.
* Optionally use diagrams or structured listings to express internal organization.

### Example

> Internally, the `TrajectoryPlanner` consists of three submodules:
>
> * **Preprocessor:** Normalizes raw sensor data.
> * **Optimizer:** Computes feasible paths using a cost-based search algorithm.
> * **Postprocessor:** Refines trajectories to meet curvature and smoothness constraints.
>
> Data flows sequentially through these stages per planning cycle, forming a controlled processing pipeline.

---

## 3. Core Subcomponents and Their Roles

**Goal:** Define the responsibilities, boundaries, and rationale for each submodule.

### Writing Guidance

* For each subcomponent, state its **input**, **output**, and **responsibility**.
* Clarify **dependency direction** among submodules.
* Note design motivations (e.g., separation for maintainability or performance).

### Example

> * **Preprocessor:** Receives unfiltered sensor readings; outputs standardized occupancy grids.
>
>   * Motivation: Encapsulate domain-specific data normalization.
> * **Optimizer:** Consumes occupancy grids, applies A* or RRT algorithms to generate feasible trajectories.
>
>   * Motivation: Keep search logic decoupled for algorithmic flexibility.
> * **Postprocessor:** Smooths trajectories and adjusts velocity profiles for execution.
>
>   * Motivation: Decouple numerical optimization from physical actuation models.

---

## 4. Data Flow Through the Component

**Goal:** Describe how information is transformed within the component and how intermediate states evolve.

### Writing Guidance

* Define **logical data pathways** — what enters, how it changes, and what exits.
* Include **data types** (e.g., continuous, discrete, event-based).
* Connect the flow to the component’s conceptual objectives.

### Example

> The component’s data flow follows:
>
> ```
> sensor_data → preprocess() → cost_map → optimize_path() → raw_trajectory → smooth_path() → final_trajectory
> ```
>
> This flow models an abstract **transformational pipeline**, converting perception data into actionable motion commands.
> Each stage corresponds to a conceptual function (perception → planning → control).

---

## 5. Component Interactions

**Goal:** Describe how this component communicates with others at runtime.

### Writing Guidance

* Explain interaction **patterns** (push/pull, pub-sub, event-driven).
* Specify **communication semantics** (synchronous vs asynchronous).
* Indicate **boundary contracts** (input/output message types or timing expectations).

### Example

> The `TrajectoryPlanner` receives asynchronous updates from the `PerceptionModule` via an event queue.
> It publishes `TrajectoryCommand` objects to the `MotionExecutor` every simulation cycle.
> The planner is stateless between cycles, ensuring deterministic reproducibility when replayed with identical input streams.

---

## 6. Separation of Concerns

**Goal:** Demonstrate how internal logic is partitioned to ensure modularity and maintainability.

### Writing Guidance

* Identify distinct **functional zones** (e.g., computation vs control).
* Justify why the boundaries exist — e.g., encapsulation, performance, testing.
* Mention constraints ensuring independence between submodules.

### Example

> Input preprocessing, optimization, and trajectory refinement are strictly decoupled:
>
> * Each submodule exposes a minimal, well-defined interface.
> * Shared state is forbidden; data is passed via immutable data structures.
> * This separation allows independent tuning and benchmarking of each substage.

---

## 7. Modularity and Extensibility

**Goal:** Explain how this component can evolve or be customized without breaking existing interfaces.

### Writing Guidance

* Define **extension mechanisms** (e.g., plug-in loading, inheritance, configuration-based selection).
* Explain how new algorithms can be integrated while preserving compatibility.
* Identify **points of abstraction** intended for substitution.

### Example

> The `Optimizer` submodule is dynamically selected at runtime through a registry mechanism.
> Developers can add new pathfinding strategies (e.g., `DynamicAStar`, `Dijkstra`) implementing the same interface without altering other modules.
> This design supports algorithmic experimentation and subsystem evolution with minimal code coupling.

---

## 8. Architectural Assumptions & Design Principles

**Goal:** Make explicit the assumptions, constraints, and philosophical principles underpinning the component’s design.

### Writing Guidance

* State assumptions about input data, execution context, or temporal consistency.
* Connect design choices to overarching system or research principles (e.g., reproducibility, determinism).
* Acknowledge known trade-offs or constraints.

### Example

> * **Assumptions:**
>
>   * Input data arrives at fixed intervals (10 Hz).
>   * The environment remains static during each planning cycle.
> * **Principles:**
>
>   * Determinism over reactivity: reproducible trajectories are prioritized over real-time adaptability.
>   * Traceability: all internal states are logged for post-hoc analysis.
> * **Trade-off:**
>
>   * Slight latency increase in exchange for robust reproducibility.

---

### 🧭 Summary

| Section                       | Focus                  | Abstraction   | Outcome                          |
| ----------------------------- | ---------------------- | ------------- | -------------------------------- |
| 1. Context                    | Position within system | Conceptual    | Defines purpose and dependencies |
| 2. System-Level View          | Internal structure     | Structural    | Outlines key submodules          |
| 3. Subcomponents              | Functional roles       | Analytical    | Explains purpose and rationale   |
| 4. Data Flow                  | Transformation         | Dynamic       | Tracks information movement      |
| 5. Interactions               | External relations     | Operational   | Defines runtime behavior         |
| 6. Separation of Concerns     | Structure              | Design        | Ensures independence             |
| 7. Modularity & Extensibility | Adaptability           | Strategic     | Enables flexible evolution       |
| 8. Assumptions & Principles   | Foundation             | Philosophical | States design rationale          |
