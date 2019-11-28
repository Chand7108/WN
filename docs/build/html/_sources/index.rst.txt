.. WN documentation master file, created by
   sphinx-quickstart on Thu Nov 28 12:54:41 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to BARA's and RBAR's documentation!
===========================================
===================================
BARA : Beacon Auto Rate Adaptation
===================================

Overview
########
Beacon Auto Rate Adaptation aka BARA uses a sender-based rate adaptation algorithm to determine proper instantaneous data rate, especially when control frames such as RTS/CTS are not available. Firstly, periodic mandatory control frame beacon is used to estimate initial data rate for sender at the beginning of a transmission. Then data rate is appropriately adapted during the transmission.

What is the basic for this algorithm?
######################################
RTS/CTS control frames are optional in 802.11 standard to improve the data packet throughput. They are optionally used only when the size of the data packet exceeds a certain threshold. When RTS/CTS packets are not used, the sender based data rate adaptation is the only choice for data packet data rate adaptation.

Usually, in multirate wireless networks, when a mobile station needs to send packets to an access point or other mobile station, it will send at basic data rate, heuristic rate or an estimated data rate. But sending them at estimated data rate can improve the throughput and entire network performance.

A scenario when a station A wants to send packets to station B and station B does not transmit for a long time, then sender is unware of the link status and does not know what data rate is suitable. Hence is neccessary to estimate the initial data rate.

BARA addresses these problems by calculating the initial data rate with periodic frame Beacon. Apart from this, two other mechanisms to enhance throughput and delay jitter.

Theory of Operation
###################
Beacon is mandatory for both infrastructure or infrastuctureless networks. Periodic beacons frames are received at a mobile station. This gives the mobile station the necessary statistics about channel to calculate the best data rate to the source mobile station. Then this data rate is recording into a table indexed by the destination mobile station address. When this mobile station needs to transmit data packet to some mobile station, it looks up the rate table for data rate that it can utilize to communicate with the target mobile station and instructs its physical layer to transmit the data packet by making use of the associated modulation corresponding to that data rate.

Algorithm
**********
The algorithm for the described operation is

When packet is received

**if** (FrameType == Beacon)

{

	Index = GetFrameMacAddress();

	ChannelStatistics = GetChannelStatisticsFromPhy();

	**if** (ChannelStatistics > ThresholdRate_11)

	Rate = 11Mbps

	**else if** (ChannelStatistics > ThresholdRate_5.5)

	Rate = 5.5Mbps

	**else if** (ChannelStatistics > ThresholdRate_2)

	Rate = 2Mbps

	**else** {

	Rate = 0Mbps;

	Log(signal is too weak. No channel to destination mobile station);

	}

	RecordRate2Table(Rate, Index);

}

*When data packet needs to transmit*

Index = GetDestinationMacAddress();

GetRateFromTable(Rate, Index);

**if** (Rate == 0){
 
	Log(Transmission can not complete. No channel to destination mobile station);

	**return** 0;

}

TransmitDataPacket(Rate);

Adaptive Rate Adaptation During Ongoing Communication
######################################################
Dynamic data estimation with only Beacon frame might not be real time, as Beacon frame can only be sensed every beacon interval, not estimated at any time. After the initial rate is estimated from history Beacon frame, an adaptive mechanism is used to predict data rate on both beacon frame and all other data the station can receive. The rate adaptation core procedure in Adaptive Rate Adaptation is same as Rate Adaptation with Beacon strategy. The only difference is an adaptive coefficient introduced to smooth the data rate on wireless channel with history rate information. One low pass filter f is introduced in the data rate prediction:

ChanStat(present) = (1-f)ChanStat(previous) + f * ChanelStatistics

(0 < f < 1)

here, the ChanStat is the cumulative prediction of potential channel status. ChannelStatistics is instantaneous channel status.

Adaptive Rate Adaptation operates with all sensed packets, including both broadcast control packets like Beacon and data transimission packets.

Basic Rate Retransmission
##########################
With sender based rate adaptation, it is impossible to accurately predict the actual data rate all the time. If sender fails to send a packet, the sender might not retry the transmission at the same data rate used during failed transmission, because the channel conditions might have deteriorated. Basic rate Retransmission solves this problem. When sender faces the above problem, it retries at basic data rate at which the receiver can certainly receive the packet, if it is still in communication range. The reason to use the basic data rate for retransmission is to avoid more failures. Then, when the sender receives acknowledgement packet for the basic data rate packet, it is able to adapt its instantaneous data rate for the next packet transmission based on the channel information of acknowledgement frame. With this the number of retransmissions after a transmission failure can be reduced to only one. This improves the netword performance especially when network load is heavy, because multiple retransmissions result in more delay between two successfull successive transmissions due to the binary exponential backoff in MAC protocol.

Conclusion
###########
In this algorithm, Beacon frame is used to adapt the intial rate without extra overhead. Then data rate can be dynamically adapted from ongoing wireless communication channel statistics at the sender. Basic data rate retransmission helps minimize packet retransmission when a packet loss occurs. This algorithm is mainly helpful for data transmission without RTS/CTS control frames, but is applicable for protocols with RTS/CTS too.

:Author : Chandana Jayaram (16CO107)

RBAR : Receiver Based Auto Rate protocol
#########################################

Overview
#########
Receiver-Based AutoRate(RBAR) is a rate adaptive MAC protocol. This rate adaptation algorithm takes advantage of the control frames RTS/CTS transmitted at the basic data rate. In contrast to the existing schemes, its rate adaptation mechanism is in receiver instead of in the sender.

Motivation
###########
The following observations led to the idea of this algorithm :

1. Rate selection can be improved by providing more timely and more complete channel quality information.

2. Channel quality information is best acquired at the receiver.

3. Transmitting channel quality information to the sender can be costly, both in terms of the resources consumed in transmitting the quantity of information needed as well as the potential loss in timeliness of the information due to transmission delays.

Basic Idea
###########
The core idea is to allow the receiver to select the appropriate rate for the data packet during the RTS/CTS packet exchange.

Advantages of this approach include :

1. The channel quality estimation mechanism can directly access all of the information made available to it by the receiving hardware, for more accurate rate selection.

2. Since the rate selection is done during RTS/CTS exchange, the channel quality estimates are nearer to the actual transmission time of the data packet than in existing sender-based approaches.

3. It can be implemented into IEEE 802.11 

Theory
#######
In RBAR, instead of carrying the duration of the reservation, the packets carry the modulation rate and size of the data packet. This modificattion serves the dual purpose of providing a mechanism by which the receiver can communicate the chosen rate to the sender, while still providing neighbouring nodes with enough information to calculate the duration of the requested reservation.

The sender chooses a data rate based on some heuristic and then stores the rate and the size of the data packet into the RTS. Neighbouring node of sender, overhears the RTS, calculates the duration of the requested reservation using the rate and packet size carried in the RTS. The neighbouring node then updates its NAV(Network Allocation Vector) to reflect the reservation. 

While receiving the RTS, the receiver uses information available to it about the channel conditions to generate an estimate of the conditions for the impending data packet transmission. Receiver then selects the appropriate rate based on that estimate, and transmits it and the packet size in the CTS back to the sender. The neighbouring node of the receiver on overhearing the CTS, calculated the duration of the requested reservation similar to the procedure used earlier. Finally, sender responds to the receipt of the CTS by transmitting the data packet at the rate chosen by Receiver.

Modifications
**************

Modifications to the standard 802.11 frames for RBAR are described below :

1. The standard 802.11 data frame has been changed to include a 32-bit check sequence positioned immediately after the source address field. The check sequence is used to protect the reservation sub header, which consists of the frame control, duration, destination address, source address and address 2 fields of the header. The new frame is distinguished by other MAC frames by a unique type code in the frame control field.
2. The RTS and CTS control frames, have been changed to encode a 4 bit rate subfield and a 12 bit length subfield, in place of the 16 bit duration field in the standard IEEE 802.11 frames. 
3. The physical layer header has been divided into two 4 bit rate subfields. The first subfield, if non-empty, indicates the rate at which the reservation subheader will be transmitted, and the second subfield indicates the rate at which the remainder of the packet will be transmitted.


:Author: Sri lalitha Devi (16CO106)

.. toctree::
   :maxdepth: 2
   :caption: Contents:




