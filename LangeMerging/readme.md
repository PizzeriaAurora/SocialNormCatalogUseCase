# Socially-Aware Autonomous Vehicle Merging: UPPAAL Model

This artifact provides the UPPAAL model for a highway merging scenario involving autonomous and human-driven vehicles. The model demonstrates how to formalize and verify complex social driving norms, such as yielding to an ambulance, using a structured, pattern-based approach. The focus is on ensuring that autonomous agents can make decisions that are not only technically correct but also socially compliant, especially in safety-critical situations.

## 1. Overview of the Use Case

This model simulates a highway merging scenario where an autonomous vehicle (`Car`) must react to another vehicle (`ApproachingVehicle`) attempting to merge onto the same lane. The merging vehicle can be either a regular car or an ambulance, and the ambulance may or may not be in an emergency state (i.e., with sirens on).

The core challenge is to model and verify the nuanced social rules that govern this interaction:

- **An emergency vehicle has the right of way.** All other vehicles must yield.
- **A non-emergency vehicle (or a non-emergency ambulance) should be yielded to as a matter of courtesy,** but this is not a strict obligation.

This use case highlights the need for autonomous systems to distinguish between hard obligations (must-do actions for safety) and soft, optional courtesies (should-do actions for social harmony).

## 2. Modeled Requirements: Social and Safety Norms

The model formalizes three key social norms using the observer patterns described in the accompanying paper. These observers monitor the system to ensure that both mandatory and optional behaviors are correctly handled.

### 2.1. Rule 1: Obligation to Stop for an Emergency

-   **Social Norm**: When an ambulance with its emergency signals on is merging, all other vehicles have a strict obligation to stop and yield the right of way.
-   **Social Importance**: This is a critical, life-saving norm universally recognized in traffic laws and social conventions. Failure to comply can delay emergency services and endanger lives.
-   **Pattern Classification**: This is a classic example of the **Obligation Observer Pattern** (Social Norm Pattern 3). The condition `(ambulance_merging and isEmergency)` triggers an obligation for the `DrivingCar` to enter its `Stop` state.
-   **Implementation**: This rule is formally verified by the `SocialObl` observer (`SO`). It enters an `Error` state if the car fails to stop when an emergency ambulance is merging. The query `A[] !SO.Error` proves that this obligation is always fulfilled.

### 2.2. Rule 2: Optional Courtesy to Yield to Non-Emergency Vehicles

-   **Social Norm**: When a non-emergency vehicle (either a regular car or a non-siren ambulance) is merging, it is socially courteous for the car on the highway to slow down and create a gap. However, this is optional, not mandatory. The driver may choose not to yield if, for example, it would disrupt traffic flow.
-   **Social Importance**: This norm governs everyday traffic politeness. While not a hard safety rule, consistent adherence makes traffic flow smoother and less confrontational. Modeling this as a non-deterministic choice allows the system to be flexible—it can be "polite" but is not forced to be if other conditions make it inadvisable.
-   **Pattern Classification**: This norm is a form of **Permission Observer Pattern** (Social Norm Pattern 1). The system is *permitted* to slow down but also *permitted* not to. The `DrivingCar` automaton has a non-deterministic choice at its `DecisionPoint`. The `SocialObl2` observer is set up to fail if the car *never* slows down, demonstrating that while optional, the behavior is expected to be possible.
-   **Implementation**: This is modeled by the `DrivingCar`'s committed location `DecisionPoint`, from which it can choose to slow down or continue normally. The query `A[] !SO2.Error` is expected to **fail**, as there is a valid trace where the car chooses not to slow down for a non-emergency vehicle, confirming the behavior is indeed optional and not a strict obligation.

### 2.3. Rule 3: Quantifying Social Cost for Inconvenience

-   **Social Norm**: Choosing *not* to slow down for a non-emergency vehicle, while permissible, is not entirely without consequence. It creates a minor social disruption or inconvenience for the merging driver. We can quantify this "impoliteness" as a social cost.
-   **Social Importance**: This allows for a more sophisticated analysis of system behavior. Instead of a binary pass/fail, we can measure and optimize for "good" behavior. A system that frequently ignores other drivers (accumulating a high social cost) might be technically correct but socially undesirable.
-   **Pattern Classification**: This is an application of the **Fuzzy Prohibition Pattern** (Social Norm Pattern 6), also referred to as a "debt" or "cost" pattern. Instead of an error state, the observer accumulates a cost (`social_cost`) each time a socially suboptimal action is taken (i.e., the `incrementCost` channel is synchronized).
-   **Implementation**: The `SocialObl3` observer (`SO3`) listens for the `incrementCost` signal, which is fired when the `DrivingCar` chooses not to slow down for a non-emergency vehicle. The `social_cost` variable accumulates a penalty for each such occurrence. Using statistical model checking queries like `E[<=...;...](max: SO3.social_cost)`, we can analyze the distribution of this social cost over many simulations, providing insight into the trade-offs between different driving strategies.

## 3. Artifact Structure

-   **File**: `AmbuLangeMerging.xml` containing the UPPAAL model.

## 4. Prerequisites

-   **UPPAAL IDE** (version 4.1 or newer), including the **Stratego** extension for optimization and statistical model checking. Download from [www.uppaal.org](http://www.uppaal.org).

## 5. How to Run the Artifact

### Step A: Simulating the System's Behavior

1.  Launch the UPPAAL IDE and open `AmbuLangeMerging.xml`.
2.  Navigate to the **Simulator** tab.
3.  Run a random simulation by clicking "Trace."
4.  **Observe the social norms in action:**
    -   When `isEmergency` is true, watch the `DrivingCar` be forced into the `Stop` state.
    -   When `isEmergency` is false, observe the non-deterministic choice at the `DecisionPoint`. The car may either go to `SlowDownABit` or continue normally.
    -   Notice the `social_cost` in the `SocialObl3` automaton increase when the car chooses *not* to slow down.

### Step B: Verifying the Properties

1.  Navigate to the **Verifier** tab.
2.  The queries are pre-loaded. Key queries include:
    -   `A[] !SO.Error`: Verifies the **obligation** to stop for an emergency. This will pass.
    -   `A[] !SO2.Error`: Verifies if the car **must** slow down for a non-emergency. This will **fail**, correctly demonstrating the behavior is optional.
3.  Explore the statistical model checking queries at the end of the list:
    -   `E[<=...](max: SO3.social_cost) under AllCars`: Examines the social cost for a baseline strategy.
    -   `E[<=...](max: SO3.social_cost) under SociallyAware`: Examines the social cost under a strategy that is forced to be socially compliant.

**Expected Outcome**: The verification will show that the hard obligation (stopping for emergencies) is always satisfied, while the optional courtesy is not. The statistical queries will demonstrate a clear difference in the accumulated `social_cost` between a strategy that ignores the social norm and one that adheres to it, quantitatively capturing the trade-off between technical goals and social compliance. Note that by default the scenario is forced to create situations where the ambulance is merging without emergency for analysis purposes. The transitions for the controller and the approaching vehicle are set to select between 0 and 0. This can be changed to 0,1 for a realistic scenario. 

## 6. Contact

For questions, contact Name Lastname at {name.last_name}@university.com.