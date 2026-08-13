### Communication and Networking
- *Network* is the organization of devices for communication
- Consists of three parts
	1. **Terminals**  -> Provides access for the network (mobile-phone, laptop, desktops...)
	2. **Switch** -> Directs data from one device to another (network-switch, routers, hubs)
	3. **Media** -> Transfers data from sender to receiver (copper-cables, electromagnetic waves, fiber-optic cables)
#### Problems in Networking
- Too complex to analyze and debug.
- Tight coupled components, complex dependency.
- Multiple Manufacturers leading compatibility issues.

To solve those problems, below approach had used.
1. *Modularization* -> according to the functionality, network divided in to several components.
2. *Layering*  -> Those above components are arranged logically in a hierarchical manner.
3. *Standardization* -> Define common set of rules for commination devices. all manufactures must implement their devices based on those rules.

### Layered Architecture of the Network - OSI Model (Open System Interconnection)

#### 1. Physical Layer
- **Purpose** :- Accept raw bits of data and transmit it via physical media (*link*)
- **Functions** :- 
	- *Encoding and Signaling* -> 
		- Converts digital (bit) data into physical signals
	- *Data Transmission and Reception(accepting)* ->
		- Send the physical signal through media/ receive physical signal from media and convert back into digital data (Decoding)
	- *Topology and Physical network Design* ->
		- The arrangement of the physical media for communication
#### 2. Data Link Layer
- **Purpose** :- Allows error-free data transferring over physical medium
- **Functions** :- 
	- *Data Framing* ->
		- Converts raw bitstreams received by physical layer in to maintainable data units (**frames**) allowing receiver to identify individual units of data.
	- *Error Detection and handling* ->
		- Detects damaged frames and handles them 
		- (*Extra Note* :- **CRC-Cycle Redundancy Checks or Parity bits** flags used for detecting errors. If a frame is damaged, frame is discarded and requests that frame again from the sender via **ARQ-Automatic Repeat reQuest**)
	- *Flow Control* ->
		- Maintaining synchronization between sender and receiver when the speeds didn't align (*sliding window, stop-and-wait* protocols)
	- *Addressing-Hardware Level* ->
		- Address physical devices in *LAN - Local Area Network* to identify the destination correctly
		- **MAC- Media Access Control Address** 
#### 3. Network Layer
- **Purpose** :- Deliver *packets/datagrams* from send-host to the receive-host while encapsulating the *routing* data from the higher levels of the network. (*host-to-host*)
- **Functions** :-
	- *Addressing-Logical* ->
		- Provides a software-based address for devices to uniquely identify across the global network
		- **IP Address (IPV4/IPV6)**
	- *Routing* ->
		- Determines the best path to send datagrams form source to the destination
	- *Datagram/Packet Encapsulation* ->
		- Wraps data coming through higher level using a wrapper including source and destination IP addresses.
	- *Congestion Control* ->
		- Tracking traffic of the network and maintain the network load
	- *Quality of Service* ->
		- Provide required priority for packets by classifying them (ex:- streaming video requires priority rather than downloading a file.)
#### 4. Transport Layer
- **Purpose** :- Provides Application end-to-end Delivery from send-host to receive-host (*process-to-process*)
- **Functions** :-
	- *Connection **establishment, management and termination*** ->
		- Establish a logical connection between two hosts and through-out the active session, maintain it and terminates when the communication is completed.
		- **TCP-3-way-Handshake**
	- *Multiplexing and Demultiplexing* ->
		- Gathering data from multiple process from various ports and wrap with headers and send it to network layer -> **Multiplexing**
		- Read the data given by the Network layer and identify correct ports and delivering payload (the data) for correct process. -> **Demultiplexing**
	- *Error Detection and Correction* ->
		- Detecting corruptions of data (*checksum*) and requests data if needed (*ACK -> Acknowledgements* and *ARQs*)
	- *Flow Control* (Sliding window protocol)
	- *Quality of Service*
#### 5. Session Layer 
- **Purpose** :- Manage and controls dialog/connection on applications host on two different hosts
- **Functions** :-
	- *Dialog Control*->
		- **full-duplex** -> can communicate both direction
		- **half-duplex** -> can communicate one direction at a time
	- *Token management* ->
		- Ensures the interactions are *mutually exclusive*, for any critical task to be done, device must hold a specific key (**token**). 
		- ex:- initiate a session, edit shared state simultaneously
	- *Synchronization* ->
		- Place checkpoints(*synchronization points*) to resume data transfers instead restarting when something goes wrong.
	- *Recovery Management* ->
		- Preserving session state information and with the help of checkpoints, recover the session safely if something goes wrong.
#### 6. Presentation Layer
- **Purpose** :- Formatting the data for transmission
- **Functions** :-
	- *Translation* ->
		- Syntax conversation
		- converts application-specific data in to abstract common format
	- *Compression* ->
		- Bandwidth optimization
		- reduces the size of the payload to improve the transmission speed (*lossy and lossless compressions*)
	- *Encryption and Decryption*  ->
		- Compromise the readable data into cyphertext to ensure the security of the payload (*TLS/SSL mechanisms*)
#### 7. Application Layer
- **Purpose** :- Provides network-based applications to the user (web browsers, social media apps etc.)


![[Gemini_Generated_Image_snx3bhsnx3bhsnx3.png]]

