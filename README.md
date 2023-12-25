# MEAM_510_Robot Final Project
 ```
    * Team Number: 3
    * Robot Name: Black Beatle
    * Team Members: Jacob Donnini, Abdinajib Mohamed, Nhat Le
    * Description of hardware: Microcontrollers: 2 ESP32- C3, 1 ESP32- S2, 1 servos motor, 
      2 motors + encoders, 1 inverter, 1 H-bridge, 1 Level shifter, 3 TOF sensors, 1 voltage regulators,
      2 IR phototransistors, 2 Vive trackers photodiodes

```
![PXL_20231219_214855812](https://github.com/jakedonnini/MEAM_510_Robot/assets/24221155/5a0785ca-84ef-4776-b3e6-27d95a9d1de8)

# Project Requirements

Each performance task will run in 2 minutes. They may be reset and run sequentially. The robot may not be reprogrammed or changed between tasks, but sensors or web interface or on/off switches may be used to indicate which task to do. Robots and objects start in their game start positions at the beginning of each task.
Minimum requirements for passing Final Project.
Control of mobile base to reliably reach any of the 5 on-field objects.
Robot must transmit a ESP-NOW packet every time it receives communication.
Minimum requirements for full marks on Graded Evaluation. This will be tested as if during normal game play but with no other robots on the field. Robots will be tested to see if they can achieve the following within the normal game time:
Perform wall-following autonomously.
Use the Vive system to transmit your X-Y location via UDP broadcast, once per second.
Autonomously identify, go to and push either trophy or fake (using 550Hz  for trophy or 23Hz for fake beacon tracking) that is randomly placed.
Autonomously move to the police car box which is randomly placed and push it so the center of the car moves at least 10 inches.
 
Performance score. Do enough to get 50 pts for full marks. 65 pts are possible giving the possibility of 15pt extra credit.

- Control Robot 				(15pt controlled to reach object)
- Robot comms via ESP-NOW	 	(2pt sending with each message)
- Vive XY via UDP			(3pt sending at 1Hz with proper protocol)
- Follow wall – full circuit		(15pt partial credit: % of full circuit)
- Autonomous navigate to trophy	 (15pt partial credit: % distance to trophy (or fake) )
- Autonomous move police 10"		(15pt partial credit: 8pt reaching box, +1pt for each inch pushed from original spot.)





# Functionality
Given the objectives and conditions of the game, our general approach was to control the robot using a website with the following functions:
- A digital map representing the actual playing field for robot navigation
- Real and fake trophy searching
- Autonomous wall-following
- Toggle grippers on/off
- Push police car

  
## All functionalities and how they were achieved:
### Control of Mobile Base to Reach Objects:
Black Beatle was equipped with a PID controller integrated with wheel encoders. This setup provided precise control over speed and direction, enabling the robot to navigate accurately to the five designated on-field objects. Our website featured a digital field, which was mapped from the real-life field, and through it, you could click on a position and the robot would move to the corresponding real location. We received data from the Vive to get the location of the robot relative to the field, allowing us to know the robot’s exact location and orientation without needing to look at the field itself. The website map shows the location of the police car (via the router data) and the robot, along with the robot’s coordinates.

### ESP-NOW Communication for Consistent Data Transfer:
Despite initial challenges with Wi-Fi connectivity, our implementation of ESP-NOW for communication was crucial. It allowed Black Beatle to send and receive packets reliably with each communication instance. This continuous data exchange was essential for coordinating the robot's movements and actions in real time, ensuring seamless performance throughout the tasks.

### Wall-Following Autonomy:
For autonomous wall-following, we utilized ToF sensors on the robot’s front and side. These sensors provided distance readings from the walls, enabling Black Beatle to maintain a consistent and safe distance while navigating along the walls. When the robot approaches a certain distance away from the wall, it will drive slightly backwards and steer outwards. The PID functionality was integral for the robot to complete a full circuit around the field autonomously, as it allowed for the robot to move adequately away from the wall while still “following it.” The derivative part of the PID was especially important in having the robot make smooth corrections.

### Vive System for Accurate Positioning:
The robot used two Vive trackers, which were key in determining its exact position and orientation on the field. These trackers transmitted the robot's X-Y location via UDP broadcast every second, providing the necessary data for precision navigation and strategic placement in relation to the game objects. From that, we could establish a coordinate system ensuring that the robot could reach any point on the field reliably. The robot was equipped with two Vive trackers to not only capture its position, but also its orientation. On the software side, a line is formed with the two Vive tracker points, and when a location is marked, the robot will angle itself to be in line with the point before driving towards it. 

### Trophy Identification and Retrieval:
Black Beetle’s trophy search mechanism involved a servo with two phototransistors. This setup allowed the robot to sweep the area until both sensors had equal readings, indicating the trophy's location. The robot then aligned itself accordingly (aiming for a servo angle of 0) and moved towards the trophy. The ToF sensor was used to gauge proximity to the trophy, triggering the grabber mechanism at the right moment. This process was vital for distinguishing between the real trophy and the fake, using 550Hz and 23Hz beacon frequencies.

### Moving the Police Car Box:
The robot was programmed to autonomously locate and move to the police car box. Once in proximity, determined by the ToF sensor readings, Black Beatle applied the necessary force to push the box, ensuring it moved at least 10 inches. This task showcased the robot's capability to perform complex maneuvers and apply controlled force when needed.

### What worked and what didn’t work:
Everything worked on our system but sending packages with ESP-NOW which resulted in us getting a 63/50. Having ESP-Now and wifi proved a bit challenging which resulted in having one of them working by themselves but not together or we would get weird data when doing both.


# Electrical Design

### Circuit Design


<div align="center">
    <img src="https://github.com/jakedonnini/MEAM_510_Robot/assets/24221155/106a62c3-d0fb-4411-8a3b-1b29d191e835" alt="Electrical Schematic">
</div>


The robot features two vive sensors with an amplifying circuit for each, H-bridge motor driver, a logic inverter to save a pin for the H-bridge, IR scanning amplifier, two time of flight sensors connected with I2C, two voltage regulators, two ESP32Cs and one ESP32S2. Each board communicates through UDP through the router in STA mode. We tried using I2C to communicate between the boards but we found that it interrupted our functions that required precision timing. We then tried ESPNOW but we were unable to get that to work with the WiFi functions. The only way we were able to get each ESP to talk to each other and still get good data from the sensors was with UDP.
We took many precautions to deal with noise in the circuit. On the main board with the motor driver the motors are wired directly to the battery and a voltage regulator with decoupling capacitors is used to power the 5V devices. The biggest concern with noise came from the IR trophy searching amplifier. Because the gain is so high it is very sensitive to any noise. We added a decoupling capacitor to the power inputs of each op-amp and twisted the wires of the two photodiodes to reduce noise. Additionally, an additional op-amp as a comparator with hysteresis was used at the output of each amplifier. This was done to make the path between high impedance inputs and outputs as small as possible to reduce the interference. Additionally, it gave a much cleaner signal for the ESP to detect even when the amplifier output is small.

### Processor architecture and code architecture

<div align="center">
    <img src="https://github.com/jakedonnini/MEAM_510_Robot/assets/24221155/7765a2ed-e128-4874-91b6-5e28b5d83056" alt="Processsing">
</div>

Starting at the top, the vive sensor took a long time to get working. We tried many different communication protocols but the only one that was able to send the positions of the two sensors was UDP. We found that an ESPC3 was not fast enough to send and collect the data so we went with an ESP32S2 and even then it could only transmit data at a frequency of 3 Hz or it would break.
The next ESP32C3 handled the time of flight distance sensors and the IR trophy scanner. I2C bus couldn’t be used while the IR sensor was sensing frequency because something in the I2C library was blocking so it was returning the wrong periods. We decide that when a search is happening the distance values are only sent every second or so and then a search is not happening the distances are sent continuously. This solved the problem and allowed us to keep both features without major reworks to the architecture. It also receives messages from the main board indicating whether it should be searching for real or fake trophies or stop searching.
The last board is where most decisions are made. The other two boards mainly send their data to this main board. Each action is treated as a state with flags to indicate which is active. Using the data from the other boards it can follow with the distance measurements and search with the servo position. During a search, the aim is to keep the position at 0 so the robot will rotate until it's close to 0 in which case it will drive forward. If it doesn’t see anything then it drives to the center and spins until it does. Using the vive data we can determine our position and angle. The move function uses this by taking a target coordinate from clicking on the map on the GUI and the robot will rotate until the angle matches and drive forward. If the angle stops matching then it rotates again to readjust until its at the destination. To find the police car, it gets the coordinates from the other UDP channel and runs the moveTo function on that target point until the distance sensor reads its close then it drives full power into it. Lastly, The wheels have a PID controller and the wall following implements a PD controller. 
The website is an interactive map of the field that displays the location of the police car and the two vive sensors in real time. The user can click on the map and the robot will autonomously navigate to that point. It displaces the readings from both distance sensors and statuses for each action. It features buttons to toggle between the different actions the robot can take during the competition.


# Mechanical Design

## Frame
The frame of the robot consists of three layers of 0.25” acrylic, which are stacked on top of eachother using standoffs. Starting from the bottom up, the first layer houses the 2 ESP32-C3’s (controlling the motors, ToF sensors, and IR phototransistors), the gripper structure, and time-of-flight distance sensors. The servo motors and the caster wheels are also attached to this base. Attached to the second layer is the beacon-finding servo and phototransistor structure and several perfboards connecting the circuits. Finally, the top layer houses the Vive trackers, which send the location of the robot at two points in order for us to determine robot location as well as orientation.

<div align="center">
    <img src="https://github.com/Abdi1717/MEAM_510_Robot/assets/24221155/0359c0e6-2546-45ea-8868-e9cc16d67fe2" alt="Top design">
</div>


## Gripper Structure
The grippers are powered via an MG996R servo motor, and motion is transmitted to the other claw using gears. The servo is attached to a 3d-printed mount, and on its underside there is a ToF distance sensor in order to determine the robot’s front distance from the wall and trophies.


<img src="https://github.com/Abdi1717/MEAM_510_Robot/assets/24221155/35fa44a0-d186-40f8-88df-21721b722451" width="45%"></img> <img src="https://github.com/Abdi1717/MEAM_510_Robot/assets/24221155/48425738-a3b2-46df-88c9-06d4ba6537ca" width="45%"></img> 

## IR Scanner Structure
The IR scanner sweeps the surroundings using an SG90 servo motor. The arm is attached to a thin panel separating the two IR phototransistors. The separation of the phototransistors in this way is crucial to our detection system.

<img src="https://github.com/Abdi1717/MEAM_510_Robot/assets/24221155/f08c44e2-7d2e-4bf7-9949-954fb8a815b2" width="45%"></img> <img src="https://github.com/Abdi1717/MEAM_510_Robot/assets/24221155/91d59a60-8c59-4a85-abb5-0d4efda333c2" width="45%"></img> 



# UI Design


<div align="center">
    <img src="https://github.com/jakedonnini/MEAM_510_Robot/assets/24221155/92533aa6-4708-4a2a-97cb-0e33ef75ecc2" alt="UIUpdated">
</div>


# PID Function

<div align="center">
    <img src="https://github.com/Abdi1717/MEAM_510_Robot/assets/24221155/f8329f5b-a19b-40cd-a461-0162c3b0cad1" alt="PID">
</div>

To get the PID controller to work we mapped out the RPS of the motors to PWM values and found functions that translate the two.

# Robot Final Design


## Top View
<img src="https://github.com/jakedonnini/MEAM_510_Robot/assets/24221155/6cf05e2a-1407-4638-840a-ac0282e26e56" width="30%"></img> <img src="https://github.com/jakedonnini/MEAM_510_Robot/assets/24221155/fd774aef-d0ac-41b4-994f-fe54eb430ce8" width="30%"></img> <img src="https://github.com/jakedonnini/MEAM_510_Robot/assets/24221155/af394804-f2d1-4609-84b7-f9270213f7c1" width="30%"></img> 

## Front View
<img src="https://github.com/jakedonnini/MEAM_510_Robot/assets/24221155/49e2d5d2-82d9-410e-bd17-3794a6fa6250" width="90%"></img> 


## Side View
![PXL_20231219_214635299](https://github.com/jakedonnini/MEAM_510_Robot/assets/24221155/f1166ba6-b639-443f-b510-34723e3bd3bc)


## Bottom View
![PXL_20231219_214743413](https://github.com/jakedonnini/MEAM_510_Robot/assets/24221155/b6f62fa5-8e3e-4cdb-866a-701edd8ade88)


## Closes Ups
<img src="https://github.com/jakedonnini/MEAM_510_Robot/assets/24221155/dd5edef2-572c-4ea0-9a77-e553f91c13c9" width="45%"></img> <img src="https://github.com/jakedonnini/MEAM_510_Robot/assets/24221155/68c2552a-c8a8-459e-b69d-5c0b69a9d88c" width="45%"></img> <img src="https://github.com/jakedonnini/MEAM_510_Robot/assets/24221155/3110d174-1c1b-4f28-8e01-1cbb7e9e8d6c" width="45%"></img> <img src="https://github.com/jakedonnini/MEAM_510_Robot/assets/24221155/582b417c-8b75-4134-85dd-f09d9d7f0a0e" width="45%"></img> 


# Bill of Materials
<div align="center">

| Quantity | Product | Link | Unit Cost | Total Cost |
|----------|---------|------|-----------|------------|
| 1        | MG996R servo motor | [Link](https://a.co/d/fwwtBwr) | $20.99 | $20.99 |
| 2        | 6V DC motor with encoder and wheels | [Link](https://a.co/d/bmUqD89) | $19.15 | $38.30 |
| 1        | VL53L0X TOF sensor | [Link](https://a.co/d/7BILPp3) | $9.99 | $9.99 |
| 1        | 2 cell LiPo battery (purchased last lab) | [Link](https://a.co/d/6FUXKF0) | $16.92 | $16.92 |
| 1        | Caster wheels | [Link](https://a.co/d/bK0Z9JW) | $6.99 | $6.99 |
| 2        | 0.25" acrylic sheet | Supplied by RPL | $0 | $0 |
| 1        | 0.125" acrylic sheet | Supplied by RPL | $0 | $0 |
|          |         |      | Grand Total | $93.19 |

</div>


# Video of Robot

Video of UI during Competition: https://drive.google.com/file/d/1AQmu_FXn1FKisQ_EzTf44Z2-oWu9fJfS/view?usp=drive_link
Robot functionalities (all autonomous):

Police car finding and pushing:
https://drive.google.com/file/d/1e66Wrx3Amz-WSneGCqED4nHg9eY3CdbR/view?usp=sharing

Trophy search:
https://drive.google.com/file/d/1jjL6m83JAJZ-NY_DRQsCbKNYrPyRwJiF/view?usp=sharing

Wall-following:
https://drive.google.com/file/d/1eSuuxfysWur8YmAnl5r6YCQXDigigevD/view?usp=sharing



## Retrospective
Most important thing learned:
Integration of Control Systems with Hardware and Software:
The most significant learning from this project was understanding how the control system intricately interacts with both the hardware and software components. The use of Proportional-Integral (PI) controllers and various algorithms was crucial in enabling our robot, Black Beatle, to move efficiently and determine the best angles for maneuvering. This hands-on experience in troubleshooting and iterative development enhanced our practical skills, providing a deep insight into robotics engineering.

Best parts of class:
Designing the Software UI and System Integration:
The highlight of the class was the opportunity to design the Software User Interface (UI) and work with the software that controls our robot's hardware, especially the mechanical design. The culmination of different systems - the electrical circuits, processor architecture, mechanical design, and the UI - working in unison was incredibly rewarding. It was a testament to our team's effort in creating a cohesive and functional robot.

Most challenging aspects:
Troubleshooting and Hardware/Software Bugs:
The most challenging aspect was troubleshooting and identifying the root causes of various hardware and software bugs. For instance, we encountered issues with servo motors consuming excessive current, leading to the failure of two ESP32-C3 microcontrollers due to a short circuit. Additionally, programming the robot for autonomous movement and logical decision-making required substantial effort, especially in managing different movement actions like backing up or determining the most effective angles.


Areas for Improvement:
Servo Motor and Power Management:
The issue with servo motors consuming excessive current and causing microcontroller failures highlights a need for better power management. Implementing current limiting circuits, using more robust microcontrollers capable of handling higher currents, or choosing different servo motors could mitigate these issues.

Noise Management in Electrical Circuits:
While precautions were taken to manage noise, particularly in the IR trophy-searching amplifier, there’s room for further improvement in noise reduction techniques. Implementing more robust filtering methods or redesigning certain circuit elements to minimize electromagnetic interference could improve the accuracy and reliability of the sensors.

Alternative to ESP-NOW Communication:
A significant area for improvement would be finding an alternative to the ESP-NOW communication protocol. Although it was a requirement, it introduced several challenges, including Wi-Fi connectivity issues, which were not crucial for the robot's operation. A more reliable and less intrusive communication method would have streamlined the process.

Anything about the classes:
Electrical and Mechanical Design Learning Curve:
The project allowed us to delve deep into electrical and mechanical design aspects. Working with vive sensors, H-bridge motor drivers, IR scanning amplifiers, and time-of-flight sensors, among others, provided us with a comprehensive understanding of the complexities involved in robot design. The hands-on experience with circuit noise management and processor architecture, particularly the use of UDP for communication between ESP32 microcontrollers, was invaluable.

Experience with User Interface Design:
Developing an interactive web interface that displayed real-time locations and allowed users to control the robot remotely was a unique aspect of our project. This experience broadened our understanding of user-centric design and its importance in robotics.

Team Collaboration:
Working in a team with diverse skills was crucial in overcoming the numerous challenges we faced. The collaborative effort in designing, building, and programming Black Beatle underscored the importance of teamwork in engineering projects.


Learning Through Failure:
The iterative process of failing, rebuilding, and succeeding taught us resilience and the importance of a growth mindset in engineering.

In summary, this project was a comprehensive learning experience, blending theory with practical skills in robotics. It not only honed our technical abilities but also enhanced our problem-solving, teamwork, and design thinking skills.









