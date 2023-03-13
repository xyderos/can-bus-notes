# Can Bus Notes

## __The vehicle is the human body, CAN is the nervous system, the nodes are the organs__ 


### __In short__ 

* Based on __multi master broadcast__, the message is not one on one, __rather broadcast it to the whole network__ 

* __A node can either send or receive, cannot do both together__ 

* There is the notion of important messages 

    * __via a priority scheme based on the ID of each node__ 

    * __The lower the ID, the higher the priority__

* __Ability to detect signal errors__ 

    * First version of CAN could disconnect faulty nodes 

* __Fully distributed__ 

    * Information can be obtained from each node 

* No complex wiring, single lined 

* __No security__ 

* Uses __serial port__

* Either __control actuators or give feedback__ 

* Transmits __serially__ into the bus 

* __If the bus is idle (TTL= 5V), any node can begin transmitting__

 
## __Two early hardware versions__ 

* __Basic CAN__

    * Basic CAN utilized a __single message buffer__ 

    * Standard CAN controllers had up to __16 message buffers__ 

        * Used for both reading and writing 

    * The received message goes through __acceptance filtering__ 

        * Decide whether to __accept or ignore__ it 

        * This is happening on the __software level__ 

        * __Bit masking__ is used to ignore messages based on __specific identifiers__

            * Reduces software load on each node 

* __Full CAN__ 

    * Every message is accompanied with either 8 or 16 memory buffers 

    * __Hardware is performing message filtering__

    * __Buffers__ 

        * __Specific buffers are made to accept specific identifiers__ 

        * __Allows more processing time per message__ 

        * __Better data consistency because of the one on one buffer to message__ 

    * Limited in the amount of frames that can be received 

    * More computational chips per node 

* __Now__ 

    * __Manufacturers combine the frame handling and acceptance filtering strengths of both__ 


* __An arising issue was the bandwidth CAN offered__ 

    * CAN data link layer had to be updated and __CAN FD__ arose 

        * Fixed Bandwidth, 12 Mbit/sec 

        * Fixed payload size, 64 bytes  

        * __As long as there are nodes that do not need to be synchronized, the bandwidth can be increased__ 

### __Each node has__

1) __Host processor__

    * __Decides what received messages mean and which messages to transmit itself__ 

    * __Sensors or actuators can be connected to the processor__

2) __CAN controller__ 

    * __Stores the bits in a buffer until a whole message is available when receiving__

    * __The host processor stores the bits in the CAN CONTROLLER and transmits them serially into the bus__ 

3) __Transceiver__ 

    * __Adapts signal levels from the bus to level the CAN controller expects__ 

    * __Protects the CAN controller__

    * __Converts the received from the CAN controller signals to a signal that should be sent to the bus__ 


## __Data transmission__ 

* If two nodes begin transmitting at the __same time__, the __highest priority will overwrite__ 

* If one node transmits a dominant bit (0) and another one a recessive bit (1) 

    * __Dominant wins__ 

* If any node transmits a high voltage, everyone will see it 

    * __Asserts priority__ 

    * __No delay__

* __The lower priority message attempts to re-transmits six bit clocks after the end of the dominant message__ 

* __When we talk about a differential bus__

    * The principles of __Carrier Sense Multiple Access/Bitwise Arbitration__ are applied 

        * If two nodes start transmitting on the same time, we need to decide which one will go first 

    * During arbitration each transmitting node __monitors the bus state__  

    * __Compares the received bit with the transmitted bit__

        * If __a dominant bit is received when a recessive bit is transmitted__ 

            * The node __stops__ transmitting 

    * Arbitration is happening during the __transmission of the ID field__ 

        * When each node starts transmitting 

            * __At the same time sends an ID with a dominant 0__ 

                * In the __high bit__ 

    * __As soon as their ID has higher priority__

        * __They will send 1 and receive 0__ 

            * Time to back off 

    * At the end of ID transmission, __all nodes but 1 will back off__ 

        * __Will send the message__ 

 

### __Example of the priority scheme__ 

_Assume 11 bit CAN IDs transmit at the same time_ 

* 0x000000011111 

* 0x000000100000 

1) __Both transmit the first 6 zeroes__ 

    _No arbitration is being made so far_ 

2) __When the 7th bit is transmitted__

    1) The node with id of 16 transmits 1 for its ID (recessive) 

    2) The node with id of 15 transmits 0 for its ID (dominant) 

3) __The node with ID of 16 realizes that it lost its arbitration__

4) __Allows node with ID of 15 to go ahead__ 


## __Timing__ 

* __Each node has its own clock__ 

* __No clock is sent during data transmission__

* __Synchronization happens by dividing each bit of the frame into a number of segments__

    * __Synchronization__

    * __Propagation__ 

    * __Phase 1__ 

    * __Phase 2__

* Length of each segment depends on the network and node conditions 

* The sample point usually falls into either phase 1 or phase 2 

 
## __Messages__ 

* __Hardware__ 

    * Two wires 

        * CAN High 

        * CAN Low 

        * __Both carry messages from nodes and connect the BUS terminators at each end__ 

            * The bus terminators are powered and grounded to create a difference of voltage providing the functionality of serial network operation 

            * Twisted quad cabling configuration 

                * Both CAN High and CAN Low wires are twisted 

                    * __Reduces electromagnetic interference and thus less errors__

* __A maximum of 30 nodes can be attached to a single section of the BUS__ 

* __A maximum of 254 nodes connected to the BUS in general__

    * Since the data signal is __8 bytes__ 

        * __Address 255 is null__ 

        * __Address 256 indicates a broadcast to all the nodes in the network__ 

    * __Broadcasting each message to each node__ 

        * __Reduces complex wiring__

        * __Local nodes implement the filters mentioned above__ 

            * Receive specific messages, __the ones they are interested to__ 

* __Format__ 

    * __Start of Frame (SoF)__ 

        * 1 bit dominant 0 

            * __Signals that the node is about to send__

                * __Alert the other nodes to be ready to listen__ 

    * CAN Identifier 

        * Contain information about the __sender__ 

            * __Message priority__ 

                * __The smaller value the higher priority__

                * Source address 

        * __CAN 2.0 is 11 bits__ 

        * Later versions are __29 bits extended__ 

            * __11 bits for the first part of the unique identifier__ 

                * Also represents message priority 

                * __Substitute remote request__ 

                    * Optional 

                    * __Must be 1__ 

                * __Identifier Extension Bit (IDE)__ 

                    * Optional 

                    * __Must be 1__ 

                * __18 bits are the second part of the data__ 
                    * Also represents priority

    * __Remote Transmission Request (RTR)__

        * __1 bit value indicating whether or not we are receiving or sending a message to a specific node__ 

    * __Control__

        * __6 bits in length__ 

            * __First bit indicates if we use the 11 bit ID or the 29 bit ID__ 

            * The second one is reserved 

            * __4 of which denote the size of the message to be transmitted__ 

                * __0 to 8 bytes__ 

    * __Data__ 

        * Contains all the CAN signals 

    * __CRC__ 

        * __15 bits that checks data integrity__

        * 1 bit is recessive (1) 

    * __Acknowledgement (ACK)__ 

        * 2 bit which indicates that the CRC found no issues 

        * First bit is 1 if 

            * __The transmitter sends 1__ 

            * __Any receiver can assert dominant__ 

        * Second bit is 1 
    * __End of Frame (EoF)__ 

        * 7 bit cap on that signals the end of transmission 

            * __Acts as padding?__ 

            * Must be 1 


### __The actual CAN Frame consists of two parts__ 

* __CAN ID__

* __Data__ 


## __Types of messages__ 

* __Data frame__ 

    * Data transmission message 

* __Error frame__

    * __A message that violates CAN formatting__ 

        * Signals an error in data transfer 

    * __The original sender destroys a frame with an error message__ 

        * Then __re-transmits__

    * __All controllers are listening for potential errors__

        * The one who found it, __transmits an error flag__ 

            * __Halting the traffic__ 

    * __The rest of the nodes receive the error__ 

        * __Active error flag__ 

            * __Six dominant bits__ 

                * __Detected an error in state ERROR-ACTIVE__

        * Passive error flag 

            * __Six recessive bits__ 

                * __Detected in error state ERROR-PASSIVE__ 

        * __Transmit 8 recessive bits (error flag)__ 

            * Aka __Error Delimiter Flag (EDF)__ 

                * __Clear the bus before taking action__

                    * The most common response is to __disregard the erroneous message__ 

    * __Each node keeps a record of detected errors__ 

        * __Via two registers__

            * __The ones the node itself was responsible in the Transmit Error Counter__

                * When the transmitter detects an error in a message, __it increments the counter__ 

                    * __Faster rate than the receiving nodes since the transmitter emits system faults in most cases__ 

                    * When you exceed a limit, the node goes into a __Passive Error State__ where it will not broadcast messages to the bus. 

                    * When you exceed a second limit, it goes into a __BUS OFF STATE__ where it is removed from the network traffic 

                    * __With this logic, you can both detect errors and limit them in the network__

            * __The ones other nodes we responsible in the Receiver Error Counter__ 

* __Remote frame__ 

    * __Request data__ 

    * __The RTR bit is transmitted as a dominant bit in the data frame__ 

    * In the remote frame there is __no data payload__ 

* __Overload frame__ 

     * __Signals that a node is overloaded__

        * First bit time during an intermission 

    * __Detection of a dominant bit during intermission__

        * __One bit after the dominant bit__ 

            * __Trigger delays__

    * __Two bit fields__ 

        * __Overload flag__ 

            * The overload flag has __6 dominant bits__

                * __Active Error Flag__ 

                    * __Destroys__ the form of the intermission field 

                    * __All other stations detect the overload condition and thus start transmitting the overload__ 

        * __Overload delimiter__ 

            * __8 recessive bits__ 

                * __Same form as the error frame__ 

 

## __OSI__ 

* __Physical layer__ 

    * Encoding/Decoding 

    * Synchronization 

    * Timing 

* __Data link layer__ 

    * Acceptance filtering 

    * Overload notification 

    * Recovery management 

    * Data encapsulation 

    * Frame coding (stuffing/destuffing) 

    * Error handling 

    * Acknowledgements 

* __Application layer__ 

    ### _Network management which directs when or how CAN nodes are on or off are included in the application layer_ 

* __Node supervision is another functionality of the network management__
    
    * Detect nodes that are __missing due to faulty condition, power loss__

        * In order to detect which nodes are not transmitting messages periodically 

            * __A client/server scheme is used__ 

                * __The server sends a message periodically to a monitoring client__

                * __If an interruption occurs based on a time limit, we have an off-line status for this node__ 

* __Break up data for transmission and reassembling them on the receiving end__

    * Usually this is part of the transport layer 

        * Eg CANOpen, DeviceNet 

#### __Typically layers 3 to 6 (network, transport, session and presentation) are not explicitly implemented__

* It is usually partially implemented in the application layer eg network and transport 

    * Will not sacrifice functionality 

#### __A thing to keep in mind is that, there might and should not be a flat structure within the nodes__ 

* __The subsystems approach should be adopted since it increases capabilities and functionality in a small sub module__ 

* __A star serial bus configuration is usually implemented__ 

* High Speed CAN: 

    * The CAN HIGH and CAN LOW communicate with speeds up to 1 Mbps 

        * Eg safety critical systems 

        * Wired in a linear serial bus configuration 

* Low Speed CAN: 

    * 125 Kbps 

* Typically there are custom pre-made CAN protocol stacks 

    * CAN Application layer, NMEA2000, CAN FD. 

 * __The need to be identified__ 

    * __Every node is assigned an ID__ 

    * __This dictates priority between messages that arrived at the same time__
 
## __Logging/Metrics__ 

* __Do not measure data directly with instruments__ 

    * Approximate results can be calculated based on the theoretical relationships 

        * __Between specific metrics and other parameters that are measured by sensors__ 

            * __Usually found in plug and play tools__ 

                * While the estimates are __inexact__, one can obtain very accurate measurements  

                    * Firstly calibrate the internal sensors with precision to external sensors 

                        * This allows a reduction in the number of the sensors and the overall cost in general 

 

### _Based on all the factors above, we can extract the result that due to its __inter connectivity__ and __decentralized communication__, we have an __efficient communication pathway and thus an invaluable asset in real time data collection___


## __ISO 11898__ 

* __Data link layer__ 

* __Physical layer for High Speed medium attachment (HS-PMA)__ 

* __Physical layer for fault tolerant, low speed medium dependent interface__ 

* ISO 11898-1 

    * __Two format options__ 

        * __Classical CAN Frame format__ 

            * 1 Mbps 

            * 8 bytes per frame 

        * __CAN Flexible Data Rate Format__ 

* ISO11898-2 

    * __HS-PMA specifications__

        * __Multiplexing data for immediate use__ 

        * __Low power mode and selective wake up mode__ 

* ISO11898-3 

    * __Set up of the data exchange between nodes__

## __J1939__ 

### __Specializes how data are exchanged in heavy duty vehicles__ 

* Cross companies protocol 

* __Consistent data logging__ 

* 250 Kbps first version 

* 500 Kbps + 29 bits used for the ID 

### __Message extensions__

* __11 bits start__ 

* __18 bits PGN (Parameter Group Number)__ 

    * __Function specific frame__ 

    * __Provides more detail about the message content__ 

    * __0x00FF00 to 0x00FFFF are reserved for business logic use__ 

* The message parameters are identified by __Suspect Parameter Numbers__ 

    * __Specific SPNs correlate to specific PGNs__ 

        * Their encoded data including 

            * Bit start 

            * Total length 

            * Scale 

            * Offset 

            * Units 

        * #### __Physical value = offset + scale * raw value in decimal__

* __Request via a Remote frame__

    * Eg for diagnostics via a CAN data logger 

        * __PGN 59904__ 

            * 3 bytes of data, request the specific PGN 

            * Priority is 6 

* __Multi packets are available also__ 

    * More than 8 bytes 


### __J1939 transport protocol__

* __Broadcast Announce Message (BAM) message__ 

    * __Goes to the entire network__ 

        * A possible structure of a BAM message 

            * 0x20, 1st byte, control byte, BAM 

            * 0x2700, 2nd-3rd byte, size of message in bytes 

            * 0x06, 4th byte, number of packets 

            * 0xFF, 5th byte, reserved 

            * 0x00FEE3, 6 to 8th byte, PGN indicated  

        * __Specifies__  

            * __PGN__

            * __Data frames to be sent__ 

            * __Length__ 

            * __Max 255 frames of data__ 

* __Data transfer__ 

    * A possible data transfer 

        * 0x01, 1st byte, specifies the sequence number 

        * Rest is just data 

    * __Specifies__ 

        * __Specifies the sequence number as a byte__ 

        * __7 bytes of data__

        * __Last frame has the remaining data plus recessive bits as padding__ 

* __Connection mode__ 

    * __Intended for a specific device__ 

 _A lot of PGNs are available only by polling_

## __OBD2__ 

### __Essentially, this is the vehicle's self diagnostic system__ 

* Support a broad range of standard __Parameter IDs__

Q : __How to log these kind of data?__ 

A :

1) __Connect the OBD2 Logger__ 

2) __Request for frames__ 

3) __Get the response frames__ 

4) __Decode raw data__ 

    
### __OBD2 Frame structure__ 

* __CAN ID__ 

    * __11 bits, distinguish between request and response__

        * 0X7DF for __request__ 

        * 0X7F8-0X7FF for __response__ 

        * __For the 29 bit identifier, we use 0x18DB33F1 as a request__ 

        * __And we receive Ox18DAF100 to 0x18DAF1FF__ 

* __Length of bytes__ 

    * 03 to 06  

* __Mode__

    * __Between 01 and 0A__ 

    * __Responses replace the 0 with a 4__ 

    * Eg 01 to request current data 

    * Different modes that we can operate from 01 to 10 

        1) Show current data 

        2) Show frozen data 

        3) Show stored Diagnostic Trouble Codes (DTCs) 

        4) Clear DTCs and stored values 

        5) Test results for oxygen sensors 

        6) Test results for system monitoring 

        7) Show pending DTCs 

        8) Control operation of on board systems 

        9) Request vehicle information 

        10) Permanent DTCs 

* __PID__ 

    * __For each mode there is a set of specific PIDs__ 

        * Eg 0D for the vehicle speed 

    * __A-B-C-D__ 

        * __Usually data bytes, need to be converted before used__ 

* __There is another slot but is unused__