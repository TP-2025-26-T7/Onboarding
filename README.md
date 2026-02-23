# Onboarding
Information about the project and how it works.

## Modules

- **Sumo + Sumo api** --> Sumo is a program for simulating traffic, and sumo api is an api overhead for the program
- **Frontend**
- **Alg-runner** --> Algrunner is a program that is the brain of the cars, it sends instructions to offline and real cars
- **car integration** --> Car integration is a bridge for communicating with real-world cars
- **OMNET** --> serves as a network simulator that receives and sends back JSON files via a UDP connection to simulate network processing
- **Central unit** --> a hub for all connections, traffic routing and logging

## Information repos

- Onboarding
- T7 website
- .github

## Architecture

![alt text](img/image.png)

## Kubernetes Architecture
This represents a concept for how Kubernetes should be set up. 

![alt text](img/image-1.png)

## How does it all work (workflow)

Workflow is split into two parts:
- Real cars (rc cars)
- Virtual cars (cars simulated via sumo)

### Real cars
This module/workflow represents communication and instructions for real-world RC cars. Where real cars communicate through *car-integration->LCU* to *Alg runner*. Alg runner sends directions and instructions. Real world cars send their telemetry data and receive instructions. 

![alt text](img/image-2.png)

### Virtual cars
This module/workflow represents communication and instructions for virtual cars. This is intended for "offline testing" purposes. This module simulates comunication trough car-integration to simulate traffic going outside to "real-cars" simulated by OMNET network simulator. Cars are simulated via SUMO, and instructions are sent via ALG runner.

***NOTE*** - communication is going through OMNET and not directly through **SUMO->Alg** runner to simulate real communication.

![alt text](img/image-3.png)

## Detailed System Workflow

The following steps describe the typical lifecycle of a simulation within this project:

### 1. Initialization
- The system is deployed via Docker Compose or Kubernetes.
- **Central Unit** starts and acts as the orchestrator.
- **SUMO Service** initializes and waits for simulation commands.
- **Alg-Runner** prepares to execute driving algorithms (e.g., FIFO, PrioQ).

### 2. Configuration & Start
- A user uploads a SUMO configuration (network XML, routes XML) via the **Frontend**.
- The **Central Unit** signals **SUMO-API** to parse the configuration and start the SUMO binary in the **SUMO Service**.
- The static road network data is sent to **Alg-Runner** so it understands the map topology (junctions, roads).

### 3. The Simulation Loop
The system operates in discrete time steps (ticks). For every step:

1.  **Advance Simulation**: The **Central Unit** requests **SUMO-API** to advance the simulation by one step.
2.  **Telemetry Gathering**:
    - **SUMO-API** fetches telemetry data (position, speed, sensors) for **each individual car** from the simulation.
    - These per-car data packets are returned to the **Central Unit**.
3.  **Network Simulation (OMNeT++)**:
    - If enabled, the **Central Unit** forwards the telemetry data for each car to **OMNeT++**.
    - OMNeT++ simulates network conditions (5G/V2X interactions, latency, packet loss) for each transmission.
    - The processed data is sent back to the **Central Unit**, where it is **collected and aggregated** into a single coherent state update (simulating the central server receiving data from many individual vehicles).
4.  **Decision Making**:
    - The (potentially network-degraded) telemetry is sent to **Alg-Runner**.
    - **Alg-Runner** processes the state using the selected algorithm (e.g., stopping at an intersection, changing lanes).
    - **Alg-Runner** computes instructions for each car (e.g., "set speed to 0", "turn left").
5.  **Execution**:
    - Instructions are sent back to the **Central Unit**.
    - **Central Unit** forwards these commands to **SUMO-API**.
    - **SUMO-API** applies the commands to the specific vehicles in the SUMO simulation via TraCI.
6.  **Visualization**:
    - The **Frontend** polls the **Central Unit** or **SUMO-API** to fetch the current state and renders the cars moving on the map.


