# Socially-Aware Traffic Manager with Pedestrian Interaction: UPPAAL Model

This artifact provides an advanced UPPAAL model of a traffic intersection manager that coordinates multiple autonomous vehicles while also accounting for pedestrian interactions. This model extends a baseline traffic management system by introducing a new social agent—a pedestrian—and equipping the autonomous vehicles with the ability to make socially-aware decisions.

## 1. Overview of the Use Case

This model simulates a complex urban intersection where autonomous vehicles must negotiate for passage not only with each other but also with a pedestrian who may wish to cross. The scenario introduces a new layer of social complexity, forcing the vehicles to balance their primary goal of crossing the intersection efficiently with the higher-priority social norm of ensuring pedestrian safety.

The key enhancement is the introduction of a pedestrian and a socially-aware role for the vehicles:

- **Pedestrian**: An automaton modeling a person who can approach the crosswalk and signal their intent to cross.
- **Vehicle**: The vehicle template is updated with a role attribute. A vehicle can be `STANDARD` (following basic traffic rules) or `SOCIALLYAWARE` (prioritizing pedestrian safety). This role change is triggered by the vehicle's decision-making process.
- **Interaction**: When a pedestrian is present, a `SOCIALLYAWARE` vehicle will yield, whereas a `STANDARD` vehicle may not. This creates a clear distinction between purely technical compliance and social compliance.

## 2. Modeled Requirements: Social and Safety Norms

This model verifies several social norms, including new rules that arise from the pedestrian's presence. These are formalized using the observer patterns described in the accompanying paper.

### 2.1. Core Traffic Safety Norms (Rules 1-5)

The fundamental safety and efficiency rules from the baseline model are preserved:

- **Prohibition of Collisions (Obs1)**: Two vehicles cannot cross conflicting paths simultaneously.
- **Prohibition of Ambiguous/Unsolicited Grants (Obs2, Obs3)**: The manager's signals must be clear and tied to actual requests.
- **Timed Back-off and Re-engagement (Obs4Alt, Obs6)**: Vehicles must adhere to a fair and efficient retry mechanism after being denied access.

These rules ensure the basic technical and social functioning of the intersection remains robust.

### 2.2. Rule 6: Obligation to Yield to Pedestrians (Socially-Aware Role)

- **Social Norm**: An autonomous vehicle, upon detecting a pedestrian wishing to cross, has a social obligation to yield. This is a higher-priority rule that overrides the vehicle's own goal of crossing quickly.
- **Social Importance**: Pedestrian safety is paramount in urban environments. A vehicle that fails to yield poses a direct threat and violates a deeply ingrained social contract.
- **Pattern Classification**: This is a combination of the Power-Inducing Pattern (Social Norm Pattern 5) and the Obligation Pattern (Social Norm Pattern 3).
  - The `humanInComing` event triggers a decision that can change the vehicle's role to `SOCIALLYAWARE`.
  - Once in this role, the vehicle is obligated to wait for the `crossed` signal from the pedestrian before proceeding.
- **Implementation**: This is modeled in the vehicle template. When a `humanInComing` signal occurs, the vehicle enters a `DecisionPoint`. If it chooses the `letPass` path, its role changes to `SOCIALLYAWARE`, and it enters the `WaitForHuman` state. The `Obs8` observer tracks whether the vehicle makes the correct choice. The query `A[] !obs8.NotPass` is designed to fail for a standard vehicle but pass for a socially-aware one, highlighting the difference in behavior.

### 2.3. Rule 7: Fuzzy Prohibition on Pedestrian Inconvenience

- **Social Norm**: While a vehicle is not strictly obligated to yield to a pedestrian (it can choose the `letNotPass` path), doing so causes social friction. A pedestrian who is repeatedly ignored may become frustrated or attempt to cross unsafely. This "inconvenience" can be quantified as a social cost.
- **Social Importance**: This models the real-world consequence of non-cooperative behavior. While a single instance of not yielding might be acceptable, a pattern of such behavior is socially negative. Quantifying it allows for optimizing strategies that balance traffic flow with pedestrian satisfaction.
- **Pattern Classification**: This is a direct application of the Fuzzy Prohibition Pattern (Social Norm Pattern 6). It associates a cost with a socially undesirable action.
- **Implementation**: The `Obs7` observer listens for the `giveUp` signal, which is emitted by the Pedestrian if it waits too long. Each time this happens, the `social_cost` is incremented. This transforms the verification problem into an optimization one, where strategies can be found to minimize this cost.

### 2.4. Rule 8: Prohibition of Unsafe Crossing Times (Role-Dependent)

- **Social Norm**: A vehicle should cross the intersection in a timely manner. However, a socially-aware vehicle that has yielded to a pedestrian is permitted a longer crossing time, as it must wait for the pedestrian to clear the path. A `STANDARD` vehicle, not having this excuse, would be considered an obstruction if it takes too long.
- **Social Importance**: This captures the context-dependency of social rules. What is considered "too long" depends on the situation. The system must be flexible enough to distinguish between legitimate delays (waiting for a pedestrian) and illegitimate ones (dawdling).
- **Pattern Classification**: This is a complex, role-based application of the Timed Prohibition Pattern. A `STANDARD` vehicle is prohibited from taking more than 11 time units to cross. A `SOCIALLYAWARE` vehicle is exempt from this strict prohibition.
- **Implementation**: The `Obs10` observer monitors the crossing time using the `cross_clock`. It enters an Error state only if a `STANDARD` vehicle takes too long. The query `A[] not obs10.Error` will pass, proving that the system correctly differentiates between the two roles and their respective time constraints.

## 3. Artifact Structure

- File: `trafic_manager_ped.xml` containing the UPPAAL model.

## 4. Prerequisites

- UPPAAL IDE (version 4.1 or newer) with the Stratego extension.

## 5. How to Run the Artifact

### Step A: Simulating the System's Behavior

1. Launch the UPPAAL IDE and open `trafic_manager_ped.xml`.
2. Navigate to the Simulator tab.
3. Run a random simulation by clicking "Trace."
4. Observe the new social interactions:
   - Manually trigger the `humanInComing!` signal from the Pedestrian automaton.
   - Observe the vehicle enter its `DecisionPoint`. It can non-deterministically choose to `letPass` or `letNotPass`.
   - If `letPass` is chosen, the vehicle's role changes to `SOCIALLYAWARE`, and it waits for the `crossed!` signal.
   - If `letNotPass` is chosen, the pedestrian will eventually `giveUp!`, and the `social_cost` in `Obs7` will increment.

### Step B: Verifying the Properties

1. Navigate to the Verifier tab. The queries are pre-loaded.
2. Verify the core safety and social norms:
   - Check the queries for `Obs1` through `Obs6`. All are expected to pass, confirming the baseline functionality is intact.
3. Analyze the new pedestrian-related norms:
   - `A[] !obs10.Error`: This will pass, proving that the system correctly handles the role-dependent crossing time limit.
   - `A[] !obs8.NotPass`: This query will fail. This is the expected outcome, as it demonstrates that a `STANDARD` vehicle is not obligated to let the pedestrian pass. This confirms the optional, "soft" nature of this social norm.
4. Analyze the Optimization and Statistical Queries (Note: These require UPPAAL Stratego):
   - `simulate[<=100;100]{obs7.social_cost}`: This simulates the system 100 times to show the distribution of the social cost incurred by vehicles not yielding to the pedestrian.
   - The strategy queries (`Finish`, `SA`) are designed to synthesize and compare control strategies. `AllCars` represents a baseline strategy focused only on getting cars through the intersection. `SociallyAware` adds the constraint that the car must yield to the pedestrian. Comparing the statistical results of `social_cost` and `carLeft` under these two strategies quantitatively demonstrates the trade-off between social politeness (lower social cost) and traffic throughput (fewer cars left).
5. Expected Outcome: The verification results will show a system that correctly balances mandatory safety rules with more nuanced, optional social courtesies. The failure of `A[] !obs8.NotPass` is a key result, confirming that the model captures the defeasible nature of the "yield to pedestrian" norm. The statistical queries will highlight the measurable costs and benefits of adopting a more socially-aware strategy.

## 6. Contact

For questions, contact Name Lastname at {name.last_name}@university.com.
