# Software-Defined-Sensor-Addressed-Motion-Control-Fabric
A software-defined motion control fabric that decouples control algorithms from physical actuators, enabling hardware reuse across multiple PLCs and machines through sensor-addressed power multiplexing over Ethernet. This project is based on patented technology and covered by one or more issued or pending patents.

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
