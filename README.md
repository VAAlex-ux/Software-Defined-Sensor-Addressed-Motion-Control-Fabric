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

**B - Supervisory PID Voltage Comparator & Main Inductor Control**

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
