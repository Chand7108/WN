.. networks documentation master file, created by
   sphinx-quickstart on Thu Nov 28 18:08:24 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to RBAR's documentation!
================================

**Overview**

Receiver-Based AutoRate(RBAR) is a rate adaptive MAC protocol. This rate adaptation algorithm takes advantage of the control frames RTS/CTS transmitted at the basic data rate. In contrast to the existing schemes, its rate adaptation mechanism is in receiver instead of in the sender.

**Motivation**

The following observations led to the idea of this algorithm :

1. Rate selection can be improved by providing more timely and more complete channel quality information.

2. Channel quality information is best acquired at the receiver.

3. Transmitting channel quality information to the sender can be costly, both in terms of the resources consumed in transmitting the quantity of information needed as well as the potential loss in timeliness of the information due to transmission delays.

**Basic Idea**

The core idea is to allow the receiver to select the appropriate rate for the data packet during the RTS/CTS packet exchange.

Advantages of this approach include :

1. The channel quality estimation mechanism can directly access all of the information made available to it by the receiving hardware, for more accurate rate selection.

2. Since the rate selection is done during RTS/CTS exchange, the channel quality estimates are nearer to the actual transmission time of the data packet than in existing sender-based approaches.

3. It can be implemented into IEEE 802.11 

**Theory** 

In RBAR, instead of carrying the duration of the reservation, the packets carry the modulation rate and size of the data packet. This modificattion serves the dual purpose of providing a mechanism by which the receiver can communicate the chosen rate to the sender, while still providing neighbouring nodes with enough information to calculate the duration of the requested reservation.

The sender chooses a data rate based on some heuristic and then stores the rate and the size of the data packet into the RTS. Neighbouring node of sender, overhears the RTS, calculates the duration of the requested reservation using the rate and packet size carried in the RTS. The neighbouring node then updates its NAV(Network Allocation Vector) to reflect the reservation. 

While receiving the RTS, the receiver uses information available to it about the channel conditions to generate an estimate of the conditions for the impending data packet transmission. Receiver then selects the appropriate rate based on that estimate, and transmits it and the packet size in the CTS back to the sender. The neighbouring node of the receiver on overhearing the CTS, calculated the duration of the requested reservation similar to the procedure used earlier. Finally, sender responds to the receipt of the CTS by transmitting the data packet at the rate chosen by Receiver.

**Modifications**

Modifications to the standard 802.11 frames for RBAR are described below :

1. The standard 802.11 data frame has been changed to include a 32-bit check sequence positioned immediately after the source address field. The check sequence is used to protect the reservation sub header, which consists of the frame control, duration, destination address, source address and address 2 fields of the header. The new frame is distinguished by other MAC frames by a unique type code in the frame control field.
2. The RTS and CTS control frames, have been changed to encode a 4 bit rate subfield and a 12 bit length subfield, in place of the 16 bit duration field in the standard IEEE 802.11 frames. 
3. The physical layer header has been divided into two 4 bit rate subfields. The first subfield, if non-empty, indicates the rate at which the reservation subheader will be transmitted, and the second subfield indicates the rate at which the remainder of the packet will be transmitted.


.. toctree::
   :maxdepth: 2
   :caption: Contents:



Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
