# Autonomous Traffic Intersection Manager: UPPAAL Model

This artifact provides an UPPAAL model for a centralized traffic intersection manager that coordinates multiple autonomous vehicles. The model demonstrates how to formalize and verify complex social and technical rules governing traffic flow, ensuring safety, fairness, and efficiency.

## 1. Overview of the Use Case

This model simulates a traffic intersection controlled by a central `intersection_manager`. Multiple autonomous vehicles approach the intersection and must request permission to cross. The manager evaluates these requests, checks for potential conflicts (i.e., two vehicles wanting to take conflicting paths), and either grants access or issues a retry command.

The core challenge is to model a system that is not only safe (preventing collisions) but also fair and efficient, adhering to social expectations about how traffic should be managed.

The system consists of the following automata:

- **vehicle**: A template for autonomous vehicles. Each vehicle approaches the intersection, sends a request, waits for a response, crosses, and then exits.
- **intersection_manager**: A central controller that processes requests, manages a routing table to detect conflicts, and issues grants or retry commands.
- **Observer Automata**: A suite of formal monitors used to verify that the system adheres to a set of critical safety and social norms.

## 2. Modeled Requirements: Social and Safety Norms

This model formalizes several crucial rules for traffic management, applying the social norm patterns from the accompanying paper. A violation of any of these rules causes the corresponding observer to enter an `ErrorState`, allowing for formal verification of the system's correctness.

### 2.1. Rule 1: Prohibition of Simultaneous Crossing (Collision Avoidance)

- **Social Norm**: The most fundamental rule of any traffic system is that two vehicles must never occupy the same space at the same time if their paths conflict.
- **Social Importance**: This is a non-negotiable, life-critical safety requirement. Its violation directly leads to a collision.
- **Pattern Classification**: This is an instance of the Prohibition Observer Pattern. It enforces that the state where `veh1.isCrossing` and `veh2.isCrossing` are simultaneously true is forbidden.
- **Implementation**: This is verified by the `Obs1` observer. The query `A[] not obs1.Error` proves that the manager's logic successfully prevents two vehicles from being in a crossing state at the same time.

### 2.2. Rule 2: Prohibition of Ambiguous Signaling

- **Social Norm**: The traffic manager must provide clear, unambiguous instructions. It should not grant access and then immediately retract it or send conflicting signals within a very short time frame.
- **Social Importance**: Clear signaling prevents confusion and hesitation, which are major causes of accidents and inefficient traffic flow. A driver (human or autonomous) receiving rapid-fire, contradictory signals cannot make a safe decision.
- **Pattern Classification**: This is a Timed Prohibition Pattern. It prohibits the system from being in a state where a slot has been granted (`hasGrantedSlot`) for less than 1 time unit. This ensures there is a minimum "grace period" for the grant to be acted upon before it can be changed.
- **Implementation**: This is verified by the `Obs2` observer. The query `A[] not obs2.Error` confirms that the manager provides stable, non-ambiguous signals.

### 2.3. Rule 3: Prohibition of Unsolicited Grants

- **Social Norm**: The manager should only grant access to a vehicle that has actually requested it. It should not issue spontaneous or "ghost" grants.
- **Social Importance**: This rule ensures the system's logic is sound and predictable. Unsolicited grants would indicate a major flaw in the manager's state tracking and could lead to chaos if vehicles started moving unexpectedly.
- **Pattern Classification**: This is another Prohibition Observer Pattern. It prohibits the condition where access is granted (`hasGrantedSlot`) without a corresponding vehicle request being registered (`data_veh_id == 0`).
- **Implementation**: This is verified by the `Obs3` observer. The query `A[] not obs3.Error` proves that the manager only grants access to vehicles that have made a valid request.

### 2.4. Rule 4: Timed Back-off After Denial (Fairness)

- **Social Norm**: If a vehicle is denied access due to a conflict, it must wait for a specified "back-off" period (e.g., 10 seconds) before re-requesting. It should not wait excessively longer than this, as that would be inefficient.
- **Social Importance**: This rule enforces fairness and efficiency. The back-off period prevents a vehicle from spamming the manager with requests, ensuring other vehicles get a chance. Limiting the wait time ensures the vehicle remains an active participant and doesn't hold up potential traffic flow behind it.
- **Pattern Classification**: This is a Timed Prohibition Pattern, but it defines both a lower and upper bound on behavior. The vehicle is prohibited from re-requesting before the time limit and also prohibited from not re-requesting after the time limit has passed.
- **Implementation**: This is verified by the `Obs4Alt` observer. It checks that a vehicle in the `wait_for_retry` state does not let its local clock `x` exceed 10 time units. The query `A[] not obs4alt.Error` confirms this rule.

### 2.5. Rule 5: Obligation to Re-engage After Back-off

- **Social Norm**: Complementing the rule above, once the back-off period is complete, the vehicle is obligated to promptly resume its attempt to cross the intersection.
- **Social Importance**: This norm ensures liveness and prevents gridlock. A vehicle that is denied access and then "gives up" indefinitely would obstruct traffic. This rule ensures all agents remain active participants in the system.
- **Pattern Classification**: This is an Obligation Observer Pattern. It mandates that if a vehicle has been waiting for the retry period (`vehi_clk_v >= 10`), it must eventually transition back to the Approaching state to re-submit its request.
- **Implementation**: This is verified by the `Obs6` observer. The query `A[] not obs6.Error` proves that a waiting vehicle will always re-engage with the system after its back-off period, ensuring liveness.

## 3. Artifact Structure

- File: `trafic_manager.xml` containing the UPPAAL model.

## 4. Prerequisites

- UPPAAL IDE (version 4.1 or newer). Download from [www.uppaal.org](http://www.uppaal.org).

## 5. How to Run the Artifact

### Step A: Simulating the System's Behavior

1. Launch the UPPAAL IDE and open `trafic_manager.xml`.
2. Navigate to the Simulator tab.
3. Run a random simulation by clicking "Trace."
4. Observe the system dynamics:
   - Watch as multiple vehicle automata transition from Approaching to `send_request`.
   - Observe the `intersection_manager` processing requests. When a conflict is detected (`conflict == true`), it sends a retry signal.
   - See a vehicle enter the `wait_retry` state and respect the `clk_v<=5` invariant before trying again.
   - When access is granted, watch the vehicle move through the cross and exit states.

### Step B: Verifying the Properties

1. Navigate to the Verifier tab.
2. The queries are pre-loaded. Check each of the observer-based properties:
   - `A[] not obs1.Error` (No collisions)
   - `A[] not obs2.Error` (No ambiguous signals)
   - `A[] not obs3.Error` (No unsolicited grants)
   - `A[] not obs4alt.Error` (Timed back-off is respected)
   - `A[] not obs6.Error` (Obligation to re-engage)
3. Click the Check button for each property.
4. Expected Outcome: The verification for all observer-related queries will return "Property is satisfied." This provides a formal guarantee that the traffic management system is safe, fair, and efficient according to the specified social and technical rules.

## 6. Contact

For questions, contact Name Lastname at {name.last_name}@university.com.
