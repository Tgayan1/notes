![[L9-IP-Addressing-Basics.pdf]]

Belongs to *Network Layer*

Network Layer,
- Providing Addressing
- Providing Algorithms to find the path from source to destination.

Router
- Intermediate Node in the network
	- Static Routing -> Manual Configuration
	- Dynamic Routing -> Automatic Algorithms (OSPF/RIP)

IP Address Versions
	-  **IPV4**
	-  IPV6

*IPV6*
- **32-Bit** binary string
- written by *four-octet-format*
	- XXXXXXXX . XXXXXXXX . XXXXXXXX . XXXXXXXX
	- 0 . 0 . 0 . 0  ->  255 . 255 . 255 . 255
- assign IP addresses for interface. so in a single laptop,
	- Ethernet
	- WIFI
	have different IP addresses.
	
![[Pasted image 20260907110450.png|403]]

```
ex 01. 10.15.94.0/24

24 is subnet mask.
Remainning Bits = 32 - 24 = 8bits.

00000000.00000000.00000000.________
IP addresses Range is 

10.15.94.00000000 - > 10.15.94.0
10.15.94.11111111 - > 10.15.94.255

ex 02. 10.15.8.0/23

23 is subnet mask
Remainning Bits = 32 - 23 = 9bits.
00000000.00000000.0000000_.________
10.15.0000100|0.00000000|

so, IP address Range is
10.15.0000100|0.00000000| -> 10.15.8.0
10.15.0000100|1.11111111| -> 10.15.9.255

```

First and Last addresses not assigned to devices In practical.

```
10.15.8.0 -> network Address
10.15.9.255 -> broadcast Address

9 bits[512 - 2 = 510 devices]
```

- IANA -> Internet Assigned Numbering Authority 
	- Private vs Public IP classification.
	- Public IP address must be request via ISP. Which expose to the entire global.(web servers)

Private to Public Address Transformation ---> Later!!!! 


***Subnet*** requires when we need an isolation for a particular network.

```
10.15.94.0/24
(00000000.00000000.00000000.________)

10.15.94.0
10.15.94.255

Total Devices can addressed -> (256 - 2) -> 254 devices

We need 4 Labs with each equipped with 50 computers (200)

Method 1. Devide All adresses in to 4 Gorups

Lab-1 -> 10.15.94.1 -> 10.15.94.50
Lab-2 -> 10.15.94.51 -> 10.15.94.100
Lab-3 -> 10.15.94.101 -> 10.15.94.150
Lab-4 -> 10.15.94.151 -> 10.15.94.200

Problem -> cannot Achive Isolation (internet restriction etc...)

Method 2. Subnet method
Devide IP block in to each Lab seprately.

Lab-1 => 50 devices -> 6 bits (64)

10.15.94.0/6
(00000000.00000000.00000000.00______)
10.15.94.0 -> 10.15.94.63

Lab-2 =>

(10.15.94.01000000) -> 10.15.94.64/6
10.15.94.64 -> 10.15.94.127

Like above

Lab-3 =>
(10.15.94.10000000) -> 10.15.94.128/6
10.15.94.128 -> 10.15.94.191

Lab-4 =>
(10.15.94.11000000) ->10.15.94.192/6
10.15.94.192 -> 10.15.94.255

so, 10.15.94.|00|000000|
			subnets -> 2^2 => 4 subnets -> 4 labs
				 devices -> 2^6 = 64 - 2 => 62 devices -> 50 computers
				
matched our requirement.

now we have
Lab-1 -> 10.15.94.0/6
Lab-2 -> 10.15.94.64/6
Lab-3 -> 10.15.94.128/6
Lab-4 -> 10.15.94.192/6
```
