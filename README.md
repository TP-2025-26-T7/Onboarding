# Onboarding
Information about the project and how it works.

## Architecture

![alt text](img/image.png)

## Kubernetes Architecture
This represents a concept for how Kubernetes should be setup. 

**TODO: Change for real arch**

![alt text](img/image-1.png)

## How does it all work (workflow)

Workflow is split into two parts:
- Real cars
- Virtual cars

### Real cars
This module/workflow represents communication and instructions for real world RC cars. Where real cars communicate trought *car-integration->LCU* to *Alg runner*. Alg runner sends directions and instructions. Real world cars send their telemetry data. 

![alt text](img/image-2.png)

### Virtual cars
This module/workflow represents communication and instructions for real vitrual cars. This is intended for "offline testing" purposes. This module simulates comunication trough car-integration to simulate trafic going outside to "real-cars" simulated by OMNET. Cars are simulated via SUMO and instructions are sent via ALG runner.

***NOTE*** - communication is going trough car integration and not directly trough **SUMO->Alg** runner to simulate real communication.

![alt text](img/image-3.png)

