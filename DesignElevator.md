<b>System Design - Elevator</b><br>

<b>Step 1: Functional Requirements, Non-Functional Requirements, Extended Requirements, Design Constraints</b>

<b>Requirements</b>

1). The elevator can have three states: Stationary, Moving, and Out of Order<br>
2). The elevator can be moving in two directions: Up, and Down<br>
3). The elevator door can only open when the elevator is stationary.<br>
4). The elevator will have two types of controls:<br>
4.1. External control, which is available on the floors for summoning the elevator by pressing the up or down button, depending on which direction the user wants to go in.<br>
4.2. Internal control, which allows the user to choose the floor that they want to go to.<br>
5). An algorithm to decide which direction the elevator should move in, if more than one passenger summons the elevator.<br>

Optimizations<br>
We can optimize the system such that the user has to wait the least amount of time in travelling from one floor to the other. <br>

Actors:<br>
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
}
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

<img src="https://github.com/rjanapa/rjanapa/blob/main/ElevatorClassDiagram.png" width="500" length="500"> <br>

<b>Elevator Scheduling Algorithm</b><br>
Passengers standing outside the elevator door press the button (up/down) to summon the elevator. The elevator system schedules to reach the passenger using an algorithm. There are two inputs for the algorithm that the elevator will use. <br>
The inputs include:<br>
Source: The floor that the passenger is waiting on.<br>
Direction: The direction in which the passenger wants to go, depending on the button they have pressed.<br>

<b>First Come First Serve (FCFS)</b><br>
The simplest method for the elevator to serve passengers’ requests is on a first-come-first-serve basis. 
The requests made by the passengers are added in a queue.
In the FCFS algorithm, requests are served in the order they were added to the queue.<br> 

Advantages and Drawbacks Of FCFS Algorithm<br>
Though the FCFS algorithm is simple to implement and makes sure all requests get a fair chance, clearly there are flaws. There may be cases where large wait times are involved for passengers. Furthermore, it also involves unnecessary movement of the elevator, resulting in wastage of resources.<br>

<b>Shortest Seek Time First (SSTF)</b><br>
This algorithm aims at serving the requests with the shortest seek time first. This approach can be implemented by generating a Minheap from the passengers’ requests, based on the distance between the source floor and the elevator’s current floor. The topmost request is picked from the heap and served first. The minheap will change continuously as the position of the elevator changes.<br>

Alternatively, an array can be generated for the requests, as shown in the diagram above. For each request, the system pre-calculates the distance of the request from the elevator and then serves the one with the minimum distance.<br>

<b>Advantages and Drawbacks of SSTF Algorithm</b><br>
There are some advantages of SSTF over FCFS algorithm. It clearly lowers the average response time of the elevator. Also, the movement of the elevator is significantly reduced. One major flaw of the design is the possibility of starvation for some of the requests. Starvation implies that large wait times may be involved for passengers that have a higher seek time (or distance from the elevator) as compared to the other incoming requests. So if there is a passenger on the top floor and most of the passengers are on the lower floors, the elevator will keep on serving those and ignore the request from the topmost floor. Also, as is the case with the FCFS algorithm, SSTF is not capable of serving requests in parallel.

<b>Elevator Algorithm (SCAN)</b><br>
SCAN is a disk scheduling algorithm that very closely reflects how an actual elevator functions, which is why it’s also sometimes called the Elevator Algorithm. Consequently, it’s an excellent approach to pick for designing your elevator scheduling algorithm. The concept aims to serve multiple requests in parallel, unlike the algorithms discussed above.<br>
The elevator will move all the way up, serving all the requests that come in its way. Only once it reaches the top floor, the elevator will change its direction, moving all the way to the last floor, serving all the requests that come in its way.<br>

<b>Advantages And Disadvantages Of SCAN</b><br>
The main advantage of SCAN over FCFS and SSTF is that it can serve several requests in parallel. However, with this approach, the elevator is continuously moving in cycles (all the way up, all the way down and repeat) even if there are no passengers, wasting resources. Also, the wait time will be considerably higher for floors that the elevator has just visited.

<b>LOOK</b><br>
The disadvantages of the SCAN algorithm can be overcome with the LOOK algorithm. This approach is similar to SCAN, except that instead of moving to the last floor (highest or lowest) in the direction it is moving in, it moves to the last request in that direction and changes its direction from there. It prevents delays due to unnecessary travel to the last floor in a direction and also eliminates the possibility of the elevator moving when there are no requests.

<b>Advantages Of LOOK</b><br>
LOOK saves the system’s resources and wait times for the passengers over SCAN by not moving ahead if there are no requests on the further floors.
Suppose that there are multiple elevators to serve requests for passengers, and the elevators are stationary since there are no requests to serve. When a new request comes in, you can use the shortest seek time to decide which elevator will move towards it. In real-world applications, elevator systems typically use the LOOK algorithm in combination with certain other algorithms to optimize the design, depending on the requirements.<br>
In case we have multiple elevators in the building, the scheduling algorithm will also take into account the state of the elevators before deciding which elevator will serve the request. Every elevator can be in one of the following states:<br>
Elevator is idle.<br>
It is moving towards the passenger where the direction of the source and destination floors is the same.<br>
It is moving towards the passenger where the direction of the source and destination floors is different.<br>
It is moving away from the passenger.<br>
An efficient solution will be to use an elevator in the first two states to serve the request.<br>

<b>Step 2: Define Microservice</b>

<b>Step 3: Draw Logical Architecture: Block diagram for each Microservice, Data/Logic flow between them.</b>

<b>Step 4: Deep dive into each Microservice</b>

<b>Step 4a: For each Microservice – Data Model, How data is stored in Storage and Cache Tier, API, Workflow/Algorithm for API, Flow across Tiers</b>

<b>Step 4b: For each Microservice – check whether each tier needs to scale for storage, cache, throughput (CPU/IO), API parallelization, remove hotspots, Availability and Geo-Distribution</b>

<b>Step 4c: Draw a generic distributed architecture per tier</b>

Reference: https://medium.com/double-pointer/system-design-interview-elevator-system-for-a-multi-storey-building-b854e766adc7<br>
