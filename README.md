# Java-basic-software-firewall
Assumptions made

To design and implement an efficient firewall, following assumptions were made. 
•	The network is subnetted to 256 hosts (/24); where only 254 hosts are usable. 
•	All hosts lie within the IP range 192.168.0.2 and 192.168.0.254 and no subnetting other than the above mentioned subnet.  
•	The web-server is assigned the IP address 192.168.0.12.
•	The PC which runs the software firewall is assigned the IP address of 192.168.0.2 (The first usable IP address of the network)
•	Light network activity as the software firewall is less efficient compared with a hardware firewall.
•	The PC with the software firewall has a direct connection with the router which perform the routing; where all incoming and outgoing packets are forwarded to the firewall PC before sent to the internal network and vice versa.
•	The mail server has an IP address of 192.168.0.30/24

Basic design

The software is implemented in such a way that the incoming packet on the network interface is captured and stripped down.  If the network packet header details meet the defined rule criteria, the packet is forwarded to the specific user in the internal network. 
The following gives a basic understanding on how the firewall works. 
•	The JnetPcap class library is imported to the java class and all the available network interfaces are opened using the PcapIf function. The user is prompted to select an interface to capture packets. 
•	The user selected interface is given the MODE_PROMISCOUS where all the packets are captured. All packets arriving at the interface are captured and passed to the PcapPacketHandler function. 
•	The captured packet then passes down to the nextPacket method; which scans for the protocol in the packet and passes down to the respective session declared within ‘if’, ‘else if’ and ‘else’ statements. 
•	The protocol specific section then analyses the section and performs the designated task. Else the packet is re-routed to a loopback interface for discarding. (The program doesn’t consist of a loopback interface, instead they are forwarded to the wireless interface of the author’s PC; where the redirected packets can be captured for testing purposes).
•	All packets arriving at the capturing interface are captured and dumped to an offline capture; where it will be written to a file which can be opened using a network analysis tool for further analysis. The packet dumper is declared within the main program; so that it gets activated every time the program executes and makes an offline record of the captured packets.
Ruleset satisfying the above requirements

The software firewall is implemented in such a way that the following basic criteria has been met. 
1.	Pass the allowed packets to the respective user. 
•	The incoming packets enter the internal network through the firewall PC, where the packet header is decoded and checked. If the rules allow the packet, the packet is encoded again and sent to the respective user in the internal network. If the packet is destined for a specific host, the packet fields are changed accordingly.

2.	Drop the rejected packets 
•	The decoded packet passes through the rule set and if the packet is not allowed through the firewall, it is dropped and no reply is sent to the source host.
•	The firewall is implemented to re-route the packets instead of dropping. JnetPcap doesn’t support packet dropping as it passively capture packets on interfaces. In order to drop the packets, an active packet capture has to be performed on the interfaces. 
3.	Reply with an ICMP destination host unreachable packet for dropped packets
•	If the packet is not allowed through the firewall, the packet is dropped and a “destination unreachable” ICMP packet is generated and sent to the source host.


Rules implemented

•	Provide unrestricted internet access to all the internal machines
•	Allow the following incoming traffic to the firewall PC
o	TCP Port 22 (SSH packets) 
o	TCP port 113 (SMTP and IRC) services used for identification and authentication
o	ICMP echo-request packets
•	Redirect TCP port 80 connections to the internal web-server
•	Allow Telnet connections from internal network (TCP port 23).
•	Allow DNS query packets (UDP port 53)
•	Allow incoming and outgoing SMTP packets (TCP port 25) between the mail server and any host.
•	Reply with a TCP RST or ICMP Unreachable for blocked packets.
•	Apply a default deny rule set.


•	The program initiates and scans for the available ports; prompts the user to select an interface. If the user selects an interface out of bound, the program loops back to the interface scan and prompts the user to enter a valid interface. If a valid interface is selected, the interface details such as device name, MAC address and the description details are printed. 
•	The captured packet is then passed to the PcapPacketHandler function which strips down the packet and pass it down to the nextPacket function. A packet capture loop is declared, so that the program runs until the user defined number of packets are captured. 
•	A condition checker is used to scan for the packet type. If the firewall captures a TCP packet, it checks for the port number of the packet. If the destination port number in the packet matches the firewall rule, the packet is passed down. Else the packet is redirected to the loopback interface.
•	If the captured packet is UDP, the packet is passed down to the analyzer and if the decoded source or destination port is 53, the packet is allowed to pass into the internal network. Other packets are redirected to the loopback interface.
•	If the packet has an ICMP header, the packet is decoded and the sub-header is checked and if the header equals to ICMP echo request, the packet is forwarded to the firewall PC (IP address 192.168.0.2)
•	If the packet has the TCP ports 22, 113, 23, 25 and 80; the packet passes down to the packet analyzer of the firewall. the following functions happen during the TCP packet analysis
o	If TCP port equals to 22 or 113, the packet is redirected to the firewall PC 	
(IP address 192.168.0.2) 
o	If TCP port 80 is detected, the packets are redirected to the web-server; configured under the IP address 192.168.0.12
o	If TCP packet has destination port as 25, the packet is allowed through the firewall regardless of the source and destination IP addresses. The packets are redirected to the mail-server running on the IP address 12.168.0.30
o	Else, TCP port 23 is detected, incoming traffic from external hosts are redirected to the loopback interface. Only outgoing traffic is allowed by the firewall. 
•	All the packets that arrive at the capturing interface are captured by the Pcap dumper and saved in an offline file, which will be useful for future analysis. 
•	If the packet capture cycle reaches the user defined value (Ex; 1000), the program will exit. Else, the program will loop until the packet capture cycle reaches the user-defined value. 



Limitations

The following limitations exist in the implemented system
•	Highly memory intensive and system time consuming as all the packets arriving at the selected interface are analyzed.
•	The program built operates on a single java-thread; has a limited ability in handling multiple packets at an instance.
•	No functionality to analyze multiple packets simultaneously. 
•	Only TCP, UDP and ICMP packets can be monitored using the software.
•	The disallowed packets are redirected to another network interface instead of dropping
•	The function for ICMP reply packet sent for the dropped packets hasn’t been implemented in the software as there is no enough resources available on ICMP packet creation and redirection using JnetPcap. But the concept is explained. 
