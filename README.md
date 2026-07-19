# Software Defined Sensor Addressed Motion Control Fabric - Automation & Controls - # 1
The Software Defined Sensor Addressed Motion Control Fabric (SDSAMCF) decouples control algorithms from physical actuators, enabling hardware reuse across multiple PLCs and machines through sensor-addressed power multiplexing over ProfiNET Ethernet.

The system integrates a supervisory PID voltage comparator loop with multi-tiered power distribution to manage inductive loads (motor matrices) via isolated 12VDC contactor coils while maintaining separate, galvanically isolated power domains (i.e. GND2, GND3) for fault containment. The architecture uses a networked-enforced common data phase via a 1783-LMS5 switch to synchronize DPDT relay-driven forward/reverse sensor states between redundant PLCs enabling deterministic polymorphic program execution; ensuring all sensor-driven trajectory changes are processed before the output, eliminating race conditions.

**A - Multi-Tiered Power Distribution Stage & Isolation Architecture**

The power distribution design acts as a physical hardware firewall. It prevents high-voltage grid noise and low-voltage inductive spikes from crossing over into the sensitive computing components.

*A.1 - Main High-Voltage Bus and Step-Down*

The input stage connects directly to a standard industrial 380V/415V AC 50Hz 3-Phase mains supply. This high-voltage rail feeds the primary power lines via dedicated molded case circuit breakers MCCBs.To power the control circuitry a single AC phase and a neutral line (220VAC) are split off and routed straight into a Murrelektronik Isolation Transformer. This steps the voltage down safely from 220VAC to a localized 15VAC line creating a complete physical air-gap that blocks grid-level voltage surges from passing down the line.

*A.2 - AC/DC Rectification and Regulation*

The 15VAC secondary line feeds into a full bridge rectifier matrix that converts the AC sine wave into a rough DC output. This output rests across a smoothing capacitor array, C101 through C104. 

The peak unfiltered DC voltage resting on these capacitors is calculated as:

$$V_{\text{peak}}=(15\text{\ VAC}\times \sqrt{2})-1.4\text{V\ (forward\ drop)}\approx 19.8\text{\ VDC}$$

This 19.8VDC rail runs directly into the input pin of an IC7812 Linear Regulator housed in a TO-220 package. The IC7812 drops this voltage down to a clean, flat 12VDC output. Because the 12VDC line only powers a single 30mA contactor coil the regulator dissipates very little power as heat.

$$P=I\times (V_{\text{in}}-V_{\text{out}})=0.03\text{\ A}\times (19.8\text{V}-12\text{V})=0.234\text{\ Watts}$$

This allows the IC7812 to run cool and stable without a large heatsink. To prevent high-frequency noise from feeding back onto this rail, 1µF ceramic decoupling capacitors C106, C107, C108 are placed immediately next to the regulator's pins to dump noise straight to ground.

*A.3 - Absolute Spatial and Galvanic Isolation*

The most critical feature of the power layout is the complete isolation of the grounding planes. The main PLC logic components run off an entirely separate power converter. The motor execution cells downstream run off their own dedicated step-down transformers each equipped with its own isolated grounding GND2, GND3. Because these grounds share no physical connection with the main control ground GND, an electrical breakdown or inductive spike in Motor Cell A or B is completely trapped within its own isolated pool. It cannot migrate up into the main 3-phase grid nor can it touch the logic rail powering the PLCs.

**B - PLC Relay Matrices & Forward/Reverse Control Logic**

The interface between the virtual software fabric and the physical hardware cells is controlled by an array of Double Pole Double Throw (DPDT) relays and input optocouplers. This setup allows the system to process physical sensor data across multiple nodes simultaneously.

*B.1 - The Optical Input Bridge*

When a physical sensor such as a Forward or Reverse proximity switch trips on the machine layout, it energizes the coil of a specific DPDT relay on the control panel.The dual poles of this relay switch separate low-voltage DC signals toward the input blocks of System A (PLC A) and System B (PLC B) simultaneously. These signals enter the PLC inputs through internal optocouplers. A 10kΩ series resistor limits the current flowing through the optocoupler's internal LED to prevent burnout. 
For example at a standard 24VDC signaling voltage the loop current is safely limited to:

$$I=\frac{24\text{\ VDC}-1.2\text{V\ (LED)}}{10\text{ k}\Omega }=2.28\text{\ mA}$$

This current level is perfectly optimized. It provides enough energy to saturate the optocoupler's phototransistor cleanly while drawing only 52mW of power keeping the components cool while avoiding overheating.

*B.2 - Sensor State Logic Flow*

The PLC relays use specific state logic to safely manage directional transitions. 

**The Ingress Synchronization Barrier**
When a sensor changes state both PLCs pause their logic engines at a network barrier. They broadcast their local sensor maps across the AB 1783-LMS5 network switch using high-speed data packets. No PLC can pass this barrier until both nodes confirm they have received the identical data packet.

**Polymorphic Program Augmentation**
Once the barrier clears, both PLCs evaluate the updated sensor map simultaneously in the exact same cycle. If for example Sensor X on PLC A is active the system executes the base motion control program. If however for example Sensor Y on PLC A trips which isn't supposed to but does for emergency, the system instantly appends an "add-on" program module from PLC B onto the logic loop of PLC A. This morphs the execution instructions on the fly without a multi-cycle network delay.

**Simultaneous Output Execution** 
Because both PLCs process this change during the exact same scan cycle, their isolated output pins fire concurrently. This sends a low-power control signal across the optical photon bridge to trigger the motor matrix relays. The system switches directional power to the motors instantly and smoothly eliminating the risk of race conditions or mechanical wear.

**C - The Latch/Unlatch State Machine Topology**

The directional control of the low-voltage linear actuators uses a Bi-Stable Logical Latch Matrix with Dominant-Reset Interlocking. This logic structure prevents the system from simultaneously commanding the forward and reverse contactors, which would otherwise cause phase-to-phase dead short across the isolated power grids (GND2, GND3). In this software fabric the commands to change directions are processed using Set (Latch) and Reset (Unlatch) memory blocks. The system enforces a strict Reset-Dominant Unlatch Priority rule where if both directional signals are active simultaneously the system defaults to an unlatched dead-stop state to protect the hardware.

*C.1 - High-Speed Temporal Analysis (Sensor-to-Actuator Speed)*

Because this architecture utilizes a Network Enforced Common Data Phase the physical time it takes to flip a linear actuator from forward to reverse is fast as it is limited only by physical hardware switching delays, completely eliminating traditional multi-cycle software network lag time.

The cycle for each execution is:

Sensors ─> Pre-Execution Network Ingress Barrier ─> Simultaneous Logic Solve ─> Photon Signal Bridge ─> Isolated Power Cells

Forming 0-cycle delay with cycle time:

$$T_{\mathrm{cycle}}=T_{\mathrm{input_read}}+T_{\mathrm{network_exchange}}+T_{\mathrm{logic_execute}}+T_{\mathrm{output_write}}$$

*C.2 - Detailed Elapsed Time Analysis (Mathematical Latency Budgets)*

Consider the architecture presented by [US20180024537A1]. To prove how the system achieves 0-cycle network propagation delay and operates significantly faster than standard implementation, we'll analyze the exact microsecond-level elapsed time $(T_{\text{elapsed}}\$ inside the execution cycle of both systems.

**Configuration A - The Conventional / Schneider Asynchronous Architecture** 

In an ordinary system and the architecture assumed in US20180024537A1, the network phase occurs at the end of the logic cycle [US20180024537A1]. If a sensor on PLC A trips and needs to trigger an "add-on" program on PLC B the elapsed time unfolds sequentially://

**Cycle 1 - PLC A**
**Input Phase:** PLC A reads its local physical sensor ($250\ \mu\text{s}$)
**Execution Phase:** PLC A solves its internal logic map ($500\ \mu\text{s}$)
**Output/Network Phase:** PLC A updates its local hardware pins and pushes the sensor update packet onto the network bus ($250\ \mu\text{s}$)

**Network Transit Lag**
The data packet travels across the network switches to reach PLC B ($250\ \mu\text{s}$).

**Cycle 2 - PLC B**
**Ingress Phase:** PLC B begins its next clock cycle and reads the incoming network data into its memory profile ($250\ \mu\text{s}$)
**Execution Phase:** PLC B finally evaluates the message and activates the "add-on" program block ($500\ \mu\text{s}$)
**Egress Phase:** PLC B updates its local simultaneous motor outputs ($250\ \mu\text{s}$)

**Total Elapsed Time (Conventional)**
$$T_{\mathrm{elapsed}} = 250 + 500 + 250 + 250 + 250 + 500 + 250 = \mathbf{2{,}250\ \mu s}$$

**The Network Lag:** Because the data had to wait for the next cycle to be evaluated, the system suffered an inherent 1-cycle network propagation delay (the data was generated in Cycle 1 but not acted upon until Cycle 2).


**C - Supervisory PID Voltage Comparator & Main Inductor Control**

While traditional motor control architectures use PID loops to handle high-bandwidth velocity/position regulation by modulating Pulse Width Modulation (PWM) duty cycles. In this fabric the PID block is repositioned as a Supervisory Voltage Verification and Interruption Engine.Its primary function is to protect the shared power domain from damage caused by inductive loads rather than just regulating dynamic speed.

The loop continuously samples the raw DC voltage plane $V_{s}\$ feeding the active motor matrix. It measures this value against a high-resolution, static software reference $V_{sref} \approx 19.8\text{ VDC}\$. The error value $e(t)\$ is calculated on every scan cycle. 

$$e(t)=V_{s\_ref}-V_{s}(t)\$$

The PID algorithm processes this error signal using standard proportional $K_{p}\$, integral $K_{i}\$, and derivative $K_{d}\$ coefficients to calculate a control output 

$$u(t)=K_{p}e(t)+K_{i}\int _{0}^{t}e(\tau )d\tau +K_{d}\frac{de(t)}{dt}$$

Because this system uses a Time-Shared Matrix Topology the inductive load profile changes drastically depending on how many motor coils are engaged at any given moment.




**This Is a Time-Shared Power Architecture — Not Simultaneous Multi-Motor Drive**

Each MUX power stage energizes only one motor at a time. PLC logic enforces: Deterministic Sequencing, Explicit Dead-Time Between Transitions, No Overlapping Activation States
Concurrent operation occurs between independent PLC/MUX cells, not within a single MUX stage.

**Each PLC Controls an Independent Power Domain**

Each PLC is paired with its own dedicated MUX power stage.
Cells are electrically isolated from one another. This architecture provides: Fault Containment, Modular Scalability, Distributed Control Domains

**Motors Used in the System Are Homogeneous and Low Voltage**

All motors within a given MUX domain: Operate within the same voltage class (≤ 25 V), Have similar stall current ranges, Have comparable inductive characteristics, Are selected to be electrically compatible with shared time-division drive.This is a design constraint, not an omission.

**Motors Include Built-In Electrical Protection**

Motors selected for this system include internal protection mechanisms such as: Integrated driver electronics (for BLDC units), Internal suppression/clamping topology where applicable.
Therefore, transient suppression occurs within the motor module not externally at the MUX. The MUX does not rely solely on wiring resistance or delay to eliminate transients.

**The PID Block Is a Supervisory Voltage Comparator**

The PID controller functions as: A filtered voltage comparator, A supervisory protection layer, A deviation detection mechanism between Vs_ref and Vs. It is not a high-bandwidth motor regulation stage.
Its purpose is fault detection and controlled interruption, not dynamic power control.

**Sequential Dead-Time Is Used for Deterministic Transitioning**

Dead-time ensures: No overlapping motor activation, Relay settling before reactivation, Controlled state transitions. It is a sequencing mechanism and not a substitute for built-in motor suppression.

**The Architecture Is Intended for Robotics Cells Not Heavy Industrial Drives**

The MUX architecture is optimized for:Low-voltage robotics, Package handling subsystems, Sequential actuation processes, Cost-constrained modular cells.

**Scalability Is Achieved Through Cell Replication**

Scalability is achieved by replicating: PLC + MUX cell units, Distributed Ethernet/IP coordination. Each cell maintains electrical independence.

© [2026] [Velikov, Aleksandar (Alexander)]. All rights reserved.
This project and the underlying technologies are protected by international copyright laws and multiple patents and/or pending patent applications. Unauthorized copying, modification, or distribution of this software/hardware/logic, in whole or in part, is strictly prohibited.
