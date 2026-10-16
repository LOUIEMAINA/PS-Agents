# ICS 3202: Artificial Intelligence
## Practical Exercise 3: Designing a Searching Agent (Self-Driving Trailer)

**Group members:**  Wario Louis Maina-190124
                    Ongoma Derrick Omondi-182842
                    Muhoro Benedict Kuria-171534

## Question 1: Agent Design Using the PEAS Framework

PEAS stands for Performance measure, Environment, Actuators, Sensors.

| Component | Description |
|---|---|
| **Performance measure** | Shortest total distance travelled; safe arrival with zero collisions; delivery of cargo undamaged; obeying traffic laws and speed limits; low fuel consumption; timely arrival |
| **Environment** | Kenya's highway network across Nairobi, Central Highlands, Rift Valley and Lake Victoria Basin: tarmac roads, other vehicles, pedestrians, animals, junctions, traffic signs, weather, road works, police checkpoints |
| **Actuators** | Steering, accelerator, brakes, gear selection, indicators, headlights, horn, windscreen wipers, trailer coupling brakes |
| **Sensors** | Global Positioning System receiver, cameras, radar, Light Detection and Ranging (lidar), speedometer, odometer, fuel gauge, trailer load sensors, road map database, microphone for sirens |

### Environment Properties (Five Definitions of Agent Environments)

| Property | Classification | Justification |
|---|---|---|
| **Observability** | Partially observable | Sensors see only nearby surroundings; the agent cannot see traffic jams, accidents or road closures far ahead |
| **Determinism** | Stochastic | Other drivers, weather and road conditions are unpredictable, so the same action does not always give the same result |
| **Episodic vs sequential** | Sequential | Each decision (route choice, turning, braking) affects all future decisions and the final outcome |
| **Static vs dynamic** | Dynamic | The environment changes while the agent is deciding: vehicles move, traffic lights change, weather shifts |
| **Number of agents** | Multi-agent (competitive and cooperative) | Other drivers, pedestrians and police share the roads and affect each other |



---

## Question 2: Directed Graph and Search Tree

### Abstraction Applied

Abstraction means removing irrelevant detail to make the problem solvable:

- Towns become **nodes (states)**; only major towns on highways are kept.
- Highways become **edges (actions)** weighted by road distance in kilometres.
- Ignored details: road curves, lanes, traffic, weather, junction layouts, fuel stops, minor roads and villages.
- Every two-way road is represented as **two directed edges** (one in each direction).
- Regions are labelled: Nairobi, Central Highlands, Rift Valley, Lake Victoria Basin.

### Problem Formulation

| Element | Definition |
|---|---|
| **Initial state** | In Nairobi |
| **Actions** | Drive along a highway to an adjacent town |
| **Transition model** | Driving from town A to town B along a road results in being in town B |
| **Goal test** | Current town is Kisumu |
| **Path cost** | Sum of distances in kilometres between towns |

### Distance Table (Kilometres, From Google Maps)

| From | To | Distance | Region of destination |
|---|---|---|---|
| Nairobi | Limuru | 30 | Central Highlands |
| Nairobi | Thika | 45 | Central Highlands |
| Nairobi | Narok | 140 | Rift Valley |
| Limuru | Naivasha | 60 | Rift Valley |
| Thika | Murang'a | 40 | Central Highlands |
| Murang'a | Nyeri | 55 | Central Highlands |
| Nyeri | Nyahururu | 70 | Central Highlands |
| Naivasha | Gilgil | 30 | Rift Valley |
| Gilgil | Nakuru | 35 | Rift Valley |
| Nyahururu | Nakuru | 85 | Rift Valley |
| Nakuru | Londiani | 75 | Rift Valley |
| Londiani | Kericho | 35 | Rift Valley |
| Londiani | Eldoret | 85 | Rift Valley |
| Narok | Bomet | 95 | Rift Valley |
| Bomet | Kericho | 50 | Rift Valley |
| Kericho | Kisumu | 90 | Lake Victoria Basin |

### Directed Graph

Each double-headed arrow stands for two directed edges (one each way) with the same distance in kilometres.

```mermaid
flowchart LR
    subgraph NRB_R["Nairobi"]
        NRB("Nairobi")
    end
    subgraph CH["Central Highlands"]
        LIM("Limuru")
        THK("Thika")
        MUR("Muranga")
        NYE("Nyeri")
        NYA("Nyahururu")
    end
    subgraph RV["Rift Valley"]
        NAI("Naivasha")
        GIL("Gilgil")
        NAK("Nakuru")
        LON("Londiani")
        KER("Kericho")
        ELD("Eldoret")
        NAR("Narok")
        BOM("Bomet")
    end
    subgraph LVB["Lake Victoria Basin"]
        KIS("Kisumu")
    end

    NRB <-->|"30"| LIM
    NRB <-->|"45"| THK
    NRB <-->|"140"| NAR
    LIM <-->|"60"| NAI
    THK <-->|"40"| MUR
    MUR <-->|"55"| NYE
    NYE <-->|"70"| NYA
    NAI <-->|"30"| GIL
    GIL <-->|"35"| NAK
    NYA <-->|"85"| NAK
    NAK <-->|"75"| LON
    LON <-->|"35"| KER
    LON <-->|"85"| ELD
    NAR <-->|"95"| BOM
    BOM <-->|"50"| KER
    KER <-->|"90"| KIS
```

### Search Tree (Uniform Cost Search from Nairobi to Kisumu)

Each node shows the town and its cumulative path cost **g(n)** in kilometres. Green nodes mark the optimal path. Dashed nodes marked "pruned" are discarded because a cheaper path to that town was already found (this avoids loops and repeated states).

```mermaid
flowchart TD
    A("Nairobi<br/>g=0") --> B("Limuru<br/>g=30")
    A --> C("Thika<br/>g=45")
    A --> D("Narok<br/>g=140")

    B --> E("Naivasha<br/>g=90")
    C --> F("Muranga<br/>g=85")
    D --> G("Bomet<br/>g=235")

    E --> H("Gilgil<br/>g=120")
    F --> I("Nyeri<br/>g=140")
    G --> J("Kericho<br/>g=285 pruned")

    H --> L("Nakuru<br/>g=155")
    I --> M("Nyahururu<br/>g=210")

    L --> O("Londiani<br/>g=230")
    L --> P("Nyahururu<br/>g=240 pruned")
    M --> Q("Nakuru<br/>g=295 pruned")

    O --> R("Kericho<br/>g=265")
    O --> S("Eldoret<br/>g=315")

    R --> T("Kisumu GOAL<br/>g=355")
    R --> U("Bomet<br/>g=315 pruned")

    classDef optimal fill:#9FE1CB,stroke:#0F6E56,color:#04342C
    classDef pruned fill:#eeeeee,stroke:#888888,color:#444444,stroke-dasharray:4 3
    class A,B,E,H,L,O,R,T optimal
    class J,P,Q,U pruned
```
---

## Question 3: Optimal Path from Nairobi to Kisumu

### Method

Uniform Cost Search (also called Dijkstra's algorithm) expands the node with the lowest path cost first. It is complete and optimal when all edge costs are positive.

### Order of Expansion

| Step | Node expanded | g(n) in kilometres | Children generated |
|---|---|---|---|
| 1 | Nairobi | 0 | Limuru 30, Thika 45, Narok 140 |
| 2 | Limuru | 30 | Naivasha 90 |
| 3 | Thika | 45 | Murang'a 85 |
| 4 | Murang'a | 85 | Nyeri 140 |
| 5 | Naivasha | 90 | Gilgil 120 |
| 6 | Gilgil | 120 | Nakuru 155 |
| 7 | Narok | 140 | Bomet 235 |
| 8 | Nyeri | 140 | Nyahururu 210 |
| 9 | Nakuru | 155 | Londiani 230 |
| 10 | Nyahururu | 210 | Nakuru 295 (pruned: worse than 155) |
| 11 | Londiani | 230 | Kericho 265, Eldoret 315 |
| 12 | Bomet | 235 | Kericho 285 (pruned: worse than 265) |
| 13 | Kericho | 265 | Kisumu 355 |
| 14 | Eldoret | 315 | none leading to a cheaper goal |
| 15 | **Kisumu (goal test passed)** | **355** | |

### Optimal Path

**Nairobi → Limuru → Naivasha → Gilgil → Nakuru → Londiani → Kericho → Kisumu**

| Leg | Distance (kilometres) |
|---|---|
| Nairobi → Limuru | 30 |
| Limuru → Naivasha | 60 |
| Naivasha → Gilgil | 30 |
| Gilgil → Nakuru | 35 |
| Nakuru → Londiani | 75 |
| Londiani → Kericho | 35 |
| Kericho → Kisumu | 90 |
| **Total** | **355** |

### Comparison of Alternative Routes

| Route | Total distance (kilometres) |
|---|---|
| **Nairobi → Limuru → Naivasha → Gilgil → Nakuru → Londiani → Kericho → Kisumu (optimal)** | **355** |
| Nairobi → Narok → Bomet → Kericho → Kisumu | 375 |
| Nairobi → Thika → Murang'a → Nyeri → Nyahururu → Nakuru → Londiani → Kericho → Kisumu | 495 |

### Properties of the Search

- **Complete:** yes, because every edge cost is positive and the graph is finite.
- **Optimal:** yes, because nodes are expanded in order of increasing path cost.
- **Time and space complexity:** O(b^(1 + C*/ε)), where b is the branching factor, C* is the optimal cost and ε is the smallest edge cost.
