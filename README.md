# Onboarding
Information about the project and how it works.

**NOTE:** Main focus of TP2025/26 was to make a starting point for the project, set the ground work and prepare the simulation side of the project. Real cars are not implemented. The missing pice mainly is the car integration module, which is intended to be a bridge for communicating with real-world cars. The communication also needs updating to support real-world cars, but the current communication is designed in a way that it can be easily extended to support both real and virtual cars. (you will need to implement a change for central unit to take routing for real cars and adjust frontend accordingly)

# Showcase of the project
1. Simulation start
![alt text](demo-video/simulation-start.gif)

2. Simulation end
![alt text](demo-video/simulation-end.gif)

Whole simulation demo is in this repository file as **demo-video/SIMULATION_DEMO.mp4**

# All accesses and who you need to talk to
**JUMP & KUBERNETES CLUSTER**
- You will need to get jump server and Kubernetes access from **Ing. Matej Janeba Phd.** or any other person resposible for it

**OMNET SERVER**
- For passwords for the server and RDP connection contact the supervisor or responsible maintainer

**GITHUB**
- For access to the repositories, you need to ask for an invite to the organization and repositories

**GITHUB API TOKEN**
- You will need to generate a personal access token for the GITHUB API
- This token is used for:
    1. GHCR access for pulling and pushing images in the cluster

After this you should have access to all the necessary resources to work on the project. If you have any issues with access, please contact the supervisor or the responsible person for the specific resource.

## Modules

- **SUMO + SUMO service + SUMO API** --> SUMO simulates traffic, the SUMO service container runs the simulator, and SUMO API exposes HTTP endpoints and TraCI control.
- **Frontend** --> Svelte UI for uploading configs and viewing simulation state
- **Alg-runner** --> computes driving instructions from simulation state
- **car integration** --> bridge for communicating with real-world cars
- **OMNET** --> network simulator used when the OMNeT path is enabled through the central unit
- **OMNET API** --> manages OMNeT simulation lifecycle on the host
- **Central unit** --> orchestrates requests between the simulation, algorithm, and network services

Every repository has its own README file with more detailed information about the specific module, how to run it, and how it works. It is recommended to read through the README files of each module to get a better understanding of the project as a whole.

![alt text](img/image-5.png)

Frontend interface

## Information repos

- Onboarding
- T7 website
- .github

## Architecture

![alt text](img/image.png)
- Adjustments:
    - OMNeT API (placed before OMMNET - Serves as API controller for OMNET simulations)
    - SUMO API+SUMO was split into for better developmnet and SUMO Service was added to interact with SUMO binary and run the simulation 

## Kubernetes Architecture
This represents a concept for how Kubernetes should be set up. 

![alt text](img/image-1.png)

## How does it all work (workflow)

Workflow is split into two parts:
- Real cars (rc cars)
- Virtual cars (cars simulated via sumo)

### Real cars (not implemented yet)
This module/workflow represents communication and instructions for real-world RC cars. Real cars communicate through *car-integration->LCU* to *Alg runner*. Alg runner sends directions and instructions, and real-world cars send telemetry data and receive instructions.

![alt text](img/image-2.png)

### Virtual cars
This module/workflow represents communication and instructions for virtual cars. This is intended for "offline testing" purposes. Cars are simulated via SUMO, instructions are sent via ALG runner, and OMNET is used for network simulation.

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
- The **Central Unit** signals **SUMO-API** to parse the configuration and start the SUMO binary in the **SUMO** container.
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

## Local Development with Docker Compose

**NOTE**: The current version does not support local development since it requiers OMNET. Since autodeploy is enable. Testing is done on development branch of kubernetes cluster.

To test the system locally, you can use the provided `docker-compose.yaml`.

### 1. Download Required Components
Clone/Download the following repositories into a single root directory:
- `alg-runner`
- `central-unit`
- `frontend`
- `sumo-service` (Note: This must be renamed or cloned into a folder named `sumo` to match the docker-compose configuration)
- `sumo-api`
- `frontend`
- `docker-compose.yaml`

### 2. Prepare Docker Compose
Copy the `docker-compose.yaml` file from the `Onboarding` directory to the root directory (where all the component folders are located).

The directory structure should look like this:

```text
/root
|-- alg-runner/
|-- central-unit/
|-- frontend/
|-- Onboarding/
|-- sumo/
|-- sumo-api/
`-- docker-compose.yaml
```

### 3. Run the System
From the root directory, run:

```bash
docker compose up --build
```
This will build and start all services locally. Wait for the services to become healthy before interacting with the system. The current compose file points OMNeT-related traffic to an external host, so that path only works when the host is reachable.

The exposed ports in the current compose file are:
- Frontend: `5173`
- Central unit: `8001`
- SUMO API: `8002`
- SUMO: `8003` and `1337`
- Alg-runner: `8000`


---

# Diagrams

To work with and update the diagrams, you can use the following tools:
- **Draw.io**: The diagrams are created using Draw.io. You can open the `.drawio` files in the respective repositories to edit them Onboarding/diagrams folder.


