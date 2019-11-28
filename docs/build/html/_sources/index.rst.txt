.. WN documentation master file, created by
   sphinx-quickstart on Thu Nov 28 12:54:41 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to BARA's documentation!
================================

**Overview**
Beacon Auto Rate Adaption aka BARA uses a sender-based rate adaptation algorithm to determine proper instantaneous data rate, especially when control frames such as RTS/CTS are not available. Firstly, periodic mandatory control frame beacon is used to estimate initial data rate for sender at the beginning of a transmission. Then data rate is appropriately adapted during the transmission.

**What is the basic for this algorithm?**
RTS/CTS control frames are optional in 802.11 standard to improve the data packet throughput. They are optionally used only when the size of the data packet exceeds a certain threshold. When RTS/CTS packets are not used, the sender based data rate adaptation is the only choice for data packet data rate adaptation.
Usually, in multirate wireless networks, when a mobile station needs to send packets to an access point or other mobile station, it will send at basic data rate, heuristic rate or an estimated data rate. But sending them at estimated data rate can improve the throughput and entire network performance.
A scenario when a station A wants to send packets to station B and station B does not transmit for a long time, then sender is unware of the link status and does not know what data rate is suitable. Hence is neccessary to estimate the initial data rate.
BARA addresses these problems by calculating the initial data rate with periodic frame Beacon. Apart from this, two other mechanisms to enhance throughput and delay jitter.

**Theory of Operation**
Beacon is mandatory for both infrastructure or infrastuctureless networks. Periodic beacons frames are received at a mobile station. This gives the mobile station the necessary statistics about channel to calculate the best data rate to the source mobile station. Then this data rate is recording into a table indexed by the destination mobile station address. When this mobile station needs to transmit data packet to some mobile station, it looks up the rate table for data rate that it can utilize to communicate with the target mobile station and instructs its physical layer to transmit the data packet by making use of the associated modulation corresponding to that data rate.



.. toctree::
   :maxdepth: 2
   :caption: Contents:



Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
