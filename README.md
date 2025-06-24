# Social Norm Pattern Catalog: Use Case Models

This repository contains the UPPAAL model files that serve as the companion artifacts for the research paper:  
**_“Bridging Social and Technical Compliance in Autonomous Systems through Formal Verification.”_**

The models demonstrate how a standardized catalog of observer patterns can be used to specify, verify, and analyze social norms in complex autonomous systems.

---

## Overview

As autonomous systems are increasingly deployed in human-centered environments, they must go beyond technical guarantees like **safety** and **liveness** to also conform with implicit **social expectations**.

This work proposes a standardized catalog of **observer patterns** that enables the formal verification of **social norms** alongside technical requirements within a unified framework.

This repository provides the concrete **UPPAAL models** for the use cases discussed in the paper’s evaluation section.

---

## Use Cases Included

This repository contains five distinct use case scenarios, each designed to test different aspects of the social norm framework:

### 1. The Collaborative Farm
- **Scenario**: A drone performs a task near a human.
- **Focus**: Trade-offs between mission efficiency (power consumption) and human-robot collaboration norms (e.g., safe distance, adaptive behavior).

### 2. Highway Merging with Emergency Vehicle
- **Scenario**: An autonomous car yields to a merging vehicle, potentially an ambulance.
- **Focus**: Tiered normative priorities (hard obligations vs. soft courtesies).

### 3-4. Autonomous Traffic Intersection
- **Scenario**: A central manager coordinates vehicles (and pedestrians).
- **Focus**: Multi-agent coordination, complex rules on safety, fairness, and role-based behavior.

### 5. Autonomous Pest Control Drone
- **Scenario**: A drone completes a spraying mission while avoiding a no-spray zone.
- **Focus**: Quantifying performance cost of compliance; handling defeasible norms (rules with exceptions).

---

## Prerequisites

To run and verify these models, you will need:

- **UPPAAL IDE** (v4.1 or newer)
- **UPPAAL Stratego**: Required for quantitative analysis (e.g., `E<>`, `sup:`, and `control:` queries)

Download the UPPAAL tool suite at [www.uppaal.org](https://www.uppaal.org)

---

## Repository Structure

```
├── CollaborativeFarm/
│   ├── HumanCollaboration.xml
│   └── HumanCollaborationAn.xml
├── HighwayMerging/
│   └── AmbuLangeMerging.xml
├── PestControlDrone/
│   └── UniqueDrone.xml
├── TrafficIntersection/
│   ├── trafic\_manager.xml
│   └── trafic\_manager\_ped.xml
└── README.md

````

---

## How to Use the Artifacts

For each use case:

1. Launch **UPPAAL IDE**
2. Go to **File -> Open System...** and select the relevant `.xml` file
3. Use the **Simulator** tab to explore the model's behavior
4. Use the **Verifier** tab to run and validate properties

---

## Use Case Instructions & Expected Results

### 1. Collaborative Farm

- **Files**: `CollaborativeFarm/HumanCollaboration.xml`, `CollaborativeFarm/HumanCollaborationAn.xml`
- **Goal**: Quantify energy cost of social compliance

**Instructions**:
- Open `HumanCollaborationAn.xml`
- In the Verifier, run the query:  
  `sup: final_power` under `ForceFinish`

**Expected Results**:
- `HumanCollaborationAn.xml`: final power = **738**
- `HumanCollaboration.xml`: final power = **570**
- Demonstrates increased cost for social compliance

---

### 2. Highway Merging

- **File**: `HighwayMerging/AmbuLangeMerging.xml`
- **Goal**: Show tiered obligations in vehicle merging

**Instructions**:
- Open `AmbuLangeMerging.xml`
- Run:
  - `A[] !SO.Error` (Emergency vehicle obligation)
  - `A[] !SO2.Error` (Non-emergency courtesy)
  - Statistical query: `E[<=...](max: SO3.social_cost)` under `AllCars` and `SociallyAware` strategies

**Expected Results**:
- `!SO.Error`: **Passes** – emergency yielding is enforced
- `!SO2.Error`: **Fails** – non-emergency yielding is optional
- Social cost is higher in the `SociallyAware` strategy

---

### 3-4. Autonomous Traffic Intersection

- **Files**: `TrafficIntersection/trafic_manager.xml`, `trafic_manager_ped.xml`
- **Goal**: Demonstrate multi-agent safety and fairness

**Instructions**:
- Open `trafic_manager_ped.xml`
- Verify:
  - Queries `Obs1` through `Obs6` – basic safety/fairness
  - `A[] !obs10.Error` – role-dependent timing
  - `A[] !obs8.NotPass` – yielding to pedestrians

**Expected Results**:
- `Obs1–Obs6` and `obs10`: **Pass**
- `obs8.NotPass`: **Fails** for STANDARD vehicle – norm is defeasible

---

### 5. Autonomous Pest Control Drone

- **File**: `PestControlDrone/UniqueDrone.xml`
- **Goal**: Analyze environmental compliance trade-off

**Instructions**:
- Open `UniqueDrone.xml`
- Run:
  - `E[<=200; 100](min: power)` under `Finish` strategy
  - Same under `SociallyCompliant` strategy

**Expected Results**:
- `SociallyCompliant`: power ≈ **13.0**
- `Finish`: power ≈ **18.8**
- Shows cost of compliance to environmental norm

---

## Citation

If you use these models in your research, please cite the accompanying paper:

```bibtex
@inproceedings{LastName2024Bridging,
  title={Bridging Social and Technical Compliance in Autonomous Systems through Formal Verification},
  author={Name Lastname},
  year={2024},
  booktitle={Conference Name},
  publisher={Publisher}
}
````

---

## Contact

For any questions regarding this artifact, please contact:
**Name Lastname** `{name.last_name}@university.com`


