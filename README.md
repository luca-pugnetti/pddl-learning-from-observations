# From Observation to Symbolic Logic: Learning PDDL Action Models from Observed Plans and Transitions

This repository contains the implementation of an AI agent designed to automatically infer a symbolic PDDL domain model by observing state transitions within a stochastic, partially observable grid world environment. Instead of receiving pre-defined logical operators, the agent reconstructs the domain rules directly from action execution logs, separating successes from failures and lifting concrete episodes into generalized, reusable parametric rules.

Developed as a project for the **Artificial Intelligence** course at the **Università di Brescia**[cite: 2].

---

## Project Structure & Verbatim Files

The core files included in this project are:
* `PDDL_domain_inference_from_observed_plans.ipynb`: The main notebook containing the experiment environment simulation, data collection, and parametric operator learning framework[cite: 1].
* `report.pdf`: The detailed academic report explaining the problem framework positioning, logical notation, architecture, and convergence metrics[cite: 3].
* `presentation.pdf`: The summary slide deck introducing the methodology, pipeline pipeline, and results[cite: 2].

---

## Overview & Key Mechanisms

The core engine relies on a pipeline of three sequential mechanisms to transition from raw transition logs to abstract symbolic rules:

* **Statistical Estimation:** Preconditions and effects are calculated based on empirical co-occurrence frequencies, conditioned exclusively on successful action executions to filter out environment noise.
* **Inertia Completion (`apply_inertia`):** An online heuristic completion step that addresses sensory blindness (`mask_prob`) by propagating persistent facts forward and backward across transitions, transforming sparse data into a dense learning matrix.
* **Parametric Lifting:** Maps concrete objects and coordinates into abstract logical variables (e.g., converting specific cell indices into generic parameters like `?x1`, `?y1`) using pattern matching on predicate signatures.

---

## Architecture & Pipeline Components

### 1. Environment Simulator (`GridWorldStochastic`)
Models a 2D discrete grid world containing a robot and a manipulable object. It features three primary actions: moving, picking up the object, and dropping it. The environment incorporates a dual-layered noise model:
* **Actuator Noise (Non-Determinism):** Modeled via `slip_prob` (locomotion failure), `pick_fail_prob` (grasping failure), and `drop_misplace_prob` (release misalignment).
* **Sensor Blindness (Partial Observability):** Controlled by `mask_prob`, where each true state predicate has an independent probability of being omitted from the observation delivered to the learner.

### 2. Transition Memory (`MiniActionModelLearner`)
Maintains the complete global history of transitions and keeps track of success/failure counts per action type. It applies configurable statistical thresholds to filter domain rules:
* **Precondition Threshold ($\tau_{pre}$):** Retains an atom as a precondition if its empirical probability satisfies $P_{pre}(p,a) \ge \tau_{pre}$.
* **Effect Threshold ($\tau_{eff}$):** Retains positive or negative effects if their empirical probabilities satisfy $P_{add}(p,a) \ge \tau_{eff}$ or $P_{del}(p,a) \ge \tau_{eff}$.

### 3. Experience Generation & Hybrid Learning
To overcome the sparsity of rare manipulation actions (such as `pick` or `drop` whose preconditions are seldom met by chance), the framework implements a hybrid data collection strategy:
1.  **Manual Plan Executions:** Guided sequences repeated across varying initial world configurations to guarantee adequate data coverage for manipulation tasks.
2.  **Random Exploration:** Hundreds of automated episodes to gather environmental variety, capture edge cases, and collect natural stochastic noise.

---

## Validation Framework

The project evaluates the quality of the learned operators using a two-tier validation scheme:

### Weak Validation (`check_pddl_inference`)
A structural completeness check that monitors iterative convergence. It simply verifies that the four fundamental PDDL blocks (`parameters`, `precondition`, `add-effects`, `del-effects`) are non-empty.

### Strong Validation (`is_strictly_valid`)
A rigorous semantic validation engine powered by the **Unified Planning Framework (UPF)**. It evaluates spatial and coordinate relationships to ensure logical correctness. For example, the `move` operator validation explicitly enforces that:
* The robot's starting position in the precondition matches the coordinate removed in the `del-effect`.
* The final position in the `add-effect` is distinct from the origin point.
* Exactly one `at` predicate for the robot exists across each respective block.

---

## Experimental Results

Our robust test suite yields high convergence scores across extreme noise constraints:
* **Adaptive Convergence:** The data collection engine dynamically increases training steps, successfully reaching stability and optimal convergence within an interval of 1,600 to 2,300 episodes.
* **Operator Robustness:** The `move` operator exhibits absolute robustness, achieving a 100% semantic validation rate across high masking (up to 0.4) and severe actuator noise. Manipulation operators (`pick` and `drop`) perform flawlessly in low-to-moderate noise conditions, with performance decay concentrated in settings where sensor blinding and stochastic failures scale concurrently.

### Pseudo-PDDL Output Example (`pick`)
Below is an example of an inferred operator generated by the system after convergence under nominal conditions:

```pddl
(:action pick
 :parameters (?robot ?obj ?x1 ?y1)
 :precondition (and (at robot ?x1 ?y1)
                    (at ?obj ?x1 ?y1)
                    (handempty ?robot))
 :add-effects (holding robot ?obj)
              (at ?obj held)
 :del-effects (at ?obj ?x1 ?y1)
              (handempty ?robot))
```

---

## References

This framework builds upon and positions itself relative to the following reference literature:
1.  **OLAM:** *Online Learning of Action Models for PDDL Planning* (Lamanna et al., IJCAI-21) — Inherits the concept of online updates and tracking successful vs. failed executions.
2.  **OffLAMPT:** *Lifted Action Models Learning from Partial Traces* (Lamanna et al., AIJ 2025) — Inspires the logical inertia rules for partial observability management.
3.  **PAL:** *Plan-Act-Learn: On-line Learning of Planning Domains from Sensor Data* (Lamanna et al., AAAI-21) — Provides the baseline paradigm for active learning within initially unknown environments.

---

## Authors & Contacts

* **Sara Moglia** - [@saramoglia](https://github.com/saramoglia)
* **Luca Pugnetti** -  [@luca-pugnetti](https://github.com/luca-pugnetti)
