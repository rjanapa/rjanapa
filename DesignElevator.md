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

Summoning The Elevator<br>
The very basic use case is summoning the elevator, or the request for an elevator. This request is made by the passenger when they press the up or down button outside the elevator on one of the floors. The request will schedule the movement of the elevator to reach the floor that the passenger is on.<br><br>
Opening/Closing the Elevator Door<br>
Both the passenger and the system (in case of an emergency) can request the opening or closing of the door when the elevator is on a floor.<br><br>
Moving Direction of the Elevator<br>
A passenger can summon the elevator. But the direction of movement will be decided by the system, based on a suitable algorithm. The algorithm can be aimed at, let’s say minimizing the wait time for passengers, or efficiently utilizing resources.<br><br>
Indicate Elevator Position and Direction<br>
The display is present both, inside the elevator and on the floors. It indicates the position of the elevator and the direction it is moving in. The system controls the display.<br><br>
Emergency Triggers<br>
There’s an emergency button inside the elevator that the passenger can press in case of an emergency. Depending on the requirements, the emergency button can trigger an alarm, make an emergency call, stop on the nearest floor, and so on.<br><br>

<b>Define Classes</b><br>

<pre>
public enum Direction {
    UP, DOWN
}
</pre>

ExternalControl<br>
ExternalControl class caters the request that’s made on the floors when the passenger presses the up or down button that’s present outside the elevator.<br>

<pre>
public class ExternalControl {
     private Direction direction;
     private int source;
}<br>
</pre>

InternalControl<br>
InternalControl is the class that governs the request made inside the elevator. There’s a control panel inside the elevator on which the passenger can select the floor to which they want to go, let’s call it “Destination”.<br>

<pre>
public class InternalControl {
     private int destination;
}
</pre>

Control<br>
Since there are two types of controls, InternalControl and ExternalControl, let’s encapsulate it in a class called Control. Control class will have two fields which are objects of ExternalControl and InternalControl classes. This class has a method called request which is used by the ExternalControl class to summon the elevator and the InternalControl class to direct the system to take the passenger to the destination floor.<br>

<pre>
public class Control {
     private InternalControl internalControl;
     private ExternalControl externalControl;
     void request(int floor, Direction direction);
}
</pre>

State<br>
The elevator can have three states — stationary, moving, out of order. We can define the State as an enum.<br>

<pre>
public enum State {
     STATIONARY, MOVING, OUTOFORDER
}
</pre>

Elevator<br>
Elevator class will have four fields: ID, State, Floor and Direction. Floor is a field of data type integer that tells which floor the elevator is currently on. Direction is an enum we defined earlier that will tell which direction the elevator is moving in, up or down.<br>

<pre>
public class Elevator {
     private int ID;
     private int currentFloor;
     private State state;
     private Direction direction;
}
</pre>

Display<br>
Display is another class that shows the floor that the elevator is on, and the direction it is moving in.<br>

<pre>
public class Display {
     private int currentFloor;
     private Direction direction;
}
</pre>

Floor<br>
The Floor class has an ID and an array for the elevator doors that are present on that floor. There can also be a function that returns the number of doors on the floor<br>

<pre>
public class Floor {
     private int id;
     private int door[];
     int NumberofDoors();
}
</pre>

Building<br>
Since it’s a multi-storey building, we’ll have a building class that will encapsulate an array of Floors and Elevators in the building. It can also have functions to retrieve the number of floors and elevators in the building.<br>

<pre>
public class Building {
     private Floor floor[];
     private Elevator elevator[];
     int numberOfFloors();
     int numberOfElevators();
}
</pre>

Scheduler<br>
The scheduler class is responsible for deciding which request to serve and which floor to send the elevator to. This class maintains a queue of requests. It has a dispatch function which passes the floor number to the elevator based on scheduling algorithms.<br>

<pre>
public class Scheduler {
     private requests[];
     dispatch(int elevator Id, int floor);
}
</pre>

<b>Class Diagram</b><br>




<b>Step 2: Define Microservice</b>

<b>Step 3: Draw Logical Architecture: Block diagram for each Microservice, Data/Logic flow between them.</b>

<b>Step 4: Deep dive into each Microservice</b>

<b>Step 4a: For each Microservice – Data Model, How data is stored in Storage and Cache Tier, API, Workflow/Algorithm for API, Flow across Tiers</b>

<b>Step 4b: For each Microservice – check whether each tier needs to scale for storage, cache, throughput (CPU/IO), API parallelization, remove hotspots, Availability and Geo-Distribution</b>

<b>Step 4c: Draw a generic distributed architecture per tier</b>
