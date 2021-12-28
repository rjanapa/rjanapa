<b>System Design - Elevator</b><br>

<b>Step 1: Functional Requirements, Non-Functional Requirements, Extended Requirements, Design Constraints</b>

<b>Requirements</b>

1). The elevator can have three states: Stationary, Moving, and Out of Order<br>
2). The elevator can be moving in two directions: Up, and Down<br>
3). The elevator door can only open when the elevator is stationary.<br>
4). The elevator will have two types of controls:<br>
4.1. External control, which is available on the floors for summoning the elevator by pressing the up or down button, depending on which direction the user wants to go in.<br>
4.2. Internal control, which allows the user to choose the floor that they want to go to.<br>
5). An algorithm to decide which direction the elevator should move in, if more than one passenger summons the elevator.

Optimizations
We can optimize the system such that the user has to wait the least amount of time in travelling from one floor to the other. 

Actors involved in the Elevator System:<br>
Passenger: The passenger is the user of the elevator system who wishes to move from one floor to another.<br>
System: The passenger interacts with the system of our elevator design. The “system” is the second actor. It’s the program that offers the service to transport the passengers between floors.<br>

<img src="https://github.com/rjanapa/rjanapa/blob/main/ElevatorUseCases.png" width="500" length="500"> <br>

<b>Step 2: Define Microservice</b>

<b>Step 3: Draw Logical Architecture: Block diagram for each Microservice, Data/Logic flow between them.</b>

<b>Step 4: Deep dive into each Microservice</b>

<b>Step 4a: For each Microservice – Data Model, How data is stored in Storage and Cache Tier, API, Workflow/Algorithm for API, Flow across Tiers</b>

<b>Step 4b: For each Microservice – check whether each tier needs to scale for storage, cache, throughput (CPU/IO), API parallelization, remove hotspots, Availability and Geo-Distribution</b>

<b>Step 4c: Draw a generic distributed architecture per tier</b>
