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

