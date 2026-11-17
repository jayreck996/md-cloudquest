ASSET LOG

####### <!-- ANCHOR MARKER - ADD ALL NEW ASSET ENTRIES DIRECTLY BELOW THIS LINE -->

## ASSET:ccna 2026-11-17 → Decimal to binary conversion: method, column values, and verification

**Source:** Packt CCNA 200-301 Foundations — Course 1, Converting from Decimal to Binary lecture

Computers store and process everything in binary — 1s and 0s — because electrical circuits have only two states: on and off. IP addresses that look like `203.0.113.5` to humans are four 8-bit binary numbers underneath. Binary conversion is not optional for CCNA — subnetting is built entirely on binary arithmetic.

The column system is the core concept. In decimal each column represents a power of 10, multiplying by 10 as you move left: 1, 10, 100, 1000. In binary each column represents a power of 2, multiplying by 2 as you move left: 1, 2, 4, 8, 16, 32, 64, 128. For IP addressing only 8 columns matter at a time because each octet of an IPv4 address is 8 bits wide. The 8 column values are always 128, 64, 32, 16, 8, 4, 2, 1 — memorise these.

The conversion method works left to right, one column at a time. For each column value ask: does this fit into what I have remaining? If yes, write 1 and subtract it from the remainder. If no, write 0 and carry the full remainder forward unchanged. When you reach the last column you must have exactly 0 remaining — anything left means an arithmetic error was made.

```
Convert 236:
Column:   128   64   32   16    8    4    2    1
Fits?     yes  yes  yes   no  yes  yes   no   no
Rem:      108   44   12   12    4    0    0    0
Bit:        1    1    1    0    1    1    0    0

Result: 11101100

Verify: 128 + 64 + 32 + 8 + 4 = 236 ✓
```

```
Convert 179:
Column:   128   64   32   16    8    4    2    1
Fits?     yes   no  yes  yes   no   no  yes  yes
Rem:       51   51   19    3    3    3    1    0
Bit:        1    0    1    1    0    0    1    1

Result: 10110011

Verify: 128 + 32 + 16 + 2 + 1 = 179 ✓
```

Verification step: after completing all 8 columns, add up every column where the bit is 1. The total must equal the original decimal number. If it does not, go back and find the mistake.

Why this matters for IP addressing: a subnet mask like `255.255.255.0` is `11111111.11111111.11111111.00000000` in binary. The 1 bits mark the network portion, the 0 bits mark the host portion. Reading and calculating subnet masks requires moving between decimal and binary quickly. The next lectures on IPv4 addresses and the subnet mask build directly on this skill.

---

## ASSET:ccna 2026-11-16 → Layer 4 Transport layer: TCP, UDP, ports, flow control, and session multiplexing

**Source:** Packt CCNA 200-301 Foundations — Course 1, Layer 4 Transport layer lecture

The Transport layer provides transparent transfer of data between hosts and is responsible for end-to-end error recovery and flow control — though neither is mandatory. Whether they are provided depends on which Layer 4 protocol is used.

Flow control is the mechanism by which a receiving host signals back to a sender that it is being overwhelmed — if the sender is transmitting faster than the receiver can process, the receiver tells it to slow down. Session multiplexing is the ability of a single host to maintain multiple simultaneous sessions over one network link, keeping each traffic stream separate. Port numbers are how this tracking works — they identify which application incoming traffic belongs to. A server running both a web server (port 80) and an email server (port 25) uses the destination port number to route each incoming packet to the correct application.

The sender also includes a source port — a randomly assigned number above 1024. The combination of source and destination port uniquely identifies a session. When the receiver responds it flips the port numbers, using the original destination port as its source and the original source port as its destination. Stateful firewalls use this to distinguish return traffic (allowed) from new inbound connections (blocked unless permitted by rule).

TCP (Transmission Control Protocol) is connection-oriented, sequenced, reliable, and supports flow control. The three-way handshake establishes a connection before data flows: sender sends SYN, receiver responds SYN-ACK, sender completes with ACK. Sequence numbers allow the receiver to reassemble segments in order and detect missing ones. The receiver sends acknowledgements; if the sender does not receive one it retransmits. The TCP header carries source and destination ports, sequence number, acknowledgement number, header length, code bits, window (flow control), checksum, and optional fields — significant overhead.

UDP (User Datagram Protocol) is connectionless, unsequenced, unreliable, and has no flow control. The sender constructs a packet and transmits it with no guarantee of delivery. The UDP header contains only source port, destination port, length, checksum, and data — minimal overhead. Any error recovery needed must be handled at the application layer.

TCP is used where reliability matters — file transfers, web browsing, email. UDP is used for real-time traffic like voice and video where TCP's overhead introduces latency that degrades quality. A dropped UDP packet produces a brief glitch, which is preferable to TCP stalling to retransmit.

Common port numbers: FTP port 21 (TCP), SSH port 22 (TCP), Telnet port 23 (TCP), HTTP port 80 (TCP), HTTPS port 443 (TCP), TFTP port 69 (UDP), SNMP port 161 (UDP), DNS port 53 (TCP and UDP).

---

## ASSET:ccna 2026-11-16 → IOS configuration management: running config, startup config, backup and restore

**Source:** Packt CCNA 200-301 Foundations — Course 1, IOS configuration management lecture

Cisco IOS maintains two separate configuration files. The running configuration is the active configuration in effect right now — every command you enter takes effect immediately and lands here. The startup configuration is what the device loads on the next reboot. These files are independent. Changes to the running config are not automatically saved to the startup config. If the router reboots before you save, all unsaved changes are lost and the device returns to the last saved startup config. This separation is intentional — it acts as a safety net, allowing you to reboot the device to discard a catastrophic misconfiguration.

To make changes persistent across reboots, run `copy running-config startup-config` (abbreviated `copy run start`) from the privileged exec prompt. This command cannot be run from inside global configuration mode.

Storage locations: the IOS operating system image is stored in flash memory. The startup config is stored in NVRAM (non-volatile RAM), which retains contents when power is removed. The running config lives in regular RAM, which is volatile — it is lost on power loss or reboot. Boot sequence: IOS loads from flash into RAM, then the startup config loads from NVRAM into RAM as the running config.

To back up configuration, copy the running config to a TFTP server with `copy running-config tftp` — it prompts for the server IP address and a filename. Including a date in the filename is good practice. Backing up to flash on the same router is possible but inadvisable — if the router fails you lose both the device and the backup. To view a file saved to flash use `more flash:filename`.

To restore from backup: erase the startup config first (`write erase` on older IOS, `erase startup-config` on newer), then copy the backup file to the startup config, then reboot. The erase step is critical — copying merges rather than replaces by default, which would mix stale config with the restored backup.

---

## ASSET:ccna 2026-11-16 → IOS CLI navigation: modes, commands, shortcuts, and filtering

**Source:** Packt CCNA 200-301 Foundations — Course 1, IOS CLI navigation lectures (parts 1 and 2)

There are three main command modes. User exec (`Router>`) is the entry-level read-only prompt. Privileged exec (`Router#`), reached by typing `enable` (abbreviated `en`), is where show commands and debugs are run. Global configuration (`Router(config)#`), reached by typing `configure terminal` (abbreviated `conf t`), is where changes are made. Sub-levels exist beneath global config — for example interface configuration mode (`Router(config-if)#`) is entered with `interface fastethernet 0/0`. Going deeper requires passing through each level one at a time. `exit` drops one level. `end` or Control+C drops all the way back to the enable prompt in one step.

The question mark triggers context-sensitive help at any point — typed alone it shows all available commands at the current level, typed after a partial command it shows matching commands, typed after a command with a space it shows available keywords for that command. The spacebar scrolls output page by page, enter scrolls line by line, Control+C breaks out of output entirely.

IOS supports command abbreviation — type enough characters for a unique match and IOS accepts it. `sh` works for `show`, `conf t` works for `configure terminal`. If the abbreviation is ambiguous IOS returns an ambiguous command error. Tab completion fills in the rest of a word once you have typed enough for a unique match.

Command history is scoped to the current mode. The up arrow cycles back through previous commands at the same level. Control+A jumps the cursor to the start of the line, useful for prepending a word to a retrieved command.

Show commands run from inside global config or sub-levels require the `do` prefix — `do show ip interface brief` works from any level. Without `do`, IOS returns invalid input detected. At the enable prompt `do` is not needed.

Error messages: `invalid input detected` with a caret marker means a typo or wrong command level. `incomplete command` means the command needs additional keywords.

The two most used show commands are `show ip interface brief` (all interfaces, their IP addresses, and up/down status) and `show running-config`. The running config can be filtered with pipe operators: `| begin <word>` starts output from the first line containing that word, `| include <word>` shows only lines containing it, `| exclude <word>` shows all lines except those containing it, `| section <word>` shows full configuration blocks for that keyword. Pipe filters use regular expressions and are case sensitive — unlike the rest of the IOS CLI which is case insensitive.

A show command debug gives point-in-time information. A debug command produces real-time output as events occur and should be used carefully in production due to output volume.

---

## ASSET:ccna 2026-11-16 → Connecting to Cisco devices: SSH, Telnet, PuTTY, AAA, in-band vs out-of-band

**Source:** Packt CCNA 200-301 Foundations — Course 1, connecting over the network lecture

In day-to-day network engineering you rarely physically touch the devices you manage. Routers and switches live in locked cabinets anywhere in the world and you connect to them over the network from your desk. The software used for this on Windows is PuTTY (free, putty.org). For the CCNA course, lab work uses Cisco Packet Tracer instead — PuTTY is not needed for the course.

SSH (Secure Shell) is the standard protocol for connecting to Cisco devices in production. The entire session — username, password, and all commands — is encrypted. Telnet looks and works identically but sends everything in plaintext. Anyone sniffing the network can read credentials and commands. Telnet is never used in real corporate environments, though it may be used in isolated lab environments.

Authentication is centralised through an AAA server (Authentication, Authorization, Accounting). Rather than configuring usernames and passwords on each of potentially hundreds of devices separately, the AAA server holds a central credential database. Devices forward login attempts to the AAA server for verification.

In-band management means connecting to a device over the same production network that carries normal user traffic. Out-of-band management uses a completely separate dedicated network exclusively for device management. Out-of-band provides a backup path if the production network has a problem, and is more secure since management traffic is isolated from user traffic.

---

## ASSET:ccna 2026-11-16 → Console cable connection: physical setup, use cases, and troubleshooting

**Source:** Packt CCNA 200-301 Foundations — Course 1, console cable lecture

Cisco devices do not ship with a default IP address. Before SSH is possible someone must connect to the device directly to perform initial configuration including adding IP addresses. The console connection provides this — a direct low-level connection to the command line that works independently of IP addressing, below the network layer.

The traditional console cable has an RJ45 connector that plugs into the console port on the device and a DB9 serial connector on the other end. Modern laptops no longer have serial ports, so a USB-to-serial adapter is needed to bridge the gap. Newer Cisco devices ship with a USB-to-mini-USB cable instead, eliminating the adapter requirement.

In PuTTY, select the Serial connection type. Find the correct COM port in Windows Device Manager under Ports (usually COM3). Serial settings for Cisco devices: speed 9600, 8 data bits, 1 stop bit, no parity, no flow control.

Once connected and the device powered on, the boot process is visible in real time. A factory-fresh device asks whether to enter the initial configuration dialog — the standard response is no, then configure manually. The console connection works on powered-on devices too, not just new ones — it immediately gives a live command prompt.

The console connection is critical for troubleshooting. If a device is unreachable over the network — corrupted IP config, management interface down — SSH will not work but the console cable bypasses all of that. If a device appears to be failing to boot, the IP address is not live until boot completes, so SSH is impossible — a console cable lets you watch the boot output and identify what is going wrong.

---

## ASSET:ccna 2026-11-16 → Cisco IOS variants: history, platforms, and monolithic vs microkernel

**Source:** Packt CCNA 200-301 Foundations — Course 1, Cisco operating systems lecture

Cisco started in 1984 as a routing company and IOS was their original operating system, still in use today. They entered switching through acquiring Crescendo in 1993, which produced the Catalyst switch line — the original Catalyst OS (CatOS) has since been deprecated and those switches now run IOS. Cisco entered firewalls by acquiring Network Translation in 1995, inheriting the PIX firewall running the Finesse OS — the PIX evolved into the ASA, which was also ported to IOS. Cisco standardised on IOS across all core network infrastructure.

Newer and higher-end platforms run different operating systems. Nexus and MDS data centre switches run NX-OS. High-end service provider routers (NCS, CRS, ASR9K, XR12K) run IOS-XR. ASR1K service provider routers run IOS-XE. Despite the different names, the CLI experience is nearly identical across all — a command that works on IOS almost certainly works unchanged on the others.

The reason multiple operating systems exist is architectural, not operational. IOS uses a monolithic kernel — all processes share the same memory space, so a single process crash can bring down the entire router. NX-OS, IOS-XR, and IOS-XE use a microkernel architecture where each process runs in its own protected memory space — a crashed process is isolated and does not affect the rest of the system. This matters most on service provider equipment where a full router restart would affect hundreds of thousands of customers. For enterprise networks IOS is sufficient — it is purpose-built, hardened over decades, and extremely stable under normal operating conditions.

The CCNA exam focuses exclusively on IOS. Learning IOS transfers directly to all other Cisco platforms.

---

## ASSET:ccna 2026-11-16 → AWS OSI mapping: client-invoked vs AWS-managed services by layer

**Source:** Derived from Packt CCNA 200-301 Foundations — Course 1, OSI model lectures

Layer 7 — Application Load Balancer: fully client-invoked. The customer creates it, defines listener rules, configures target groups, and decides routing logic. AWS manages the underlying infrastructure.

Layer 6 — AWS Certificate Manager: both sides. The client requests and attaches certificates. AWS handles automatic renewal without customer action.

Layer 5 — Elastic Load Balancing: client-invoked. The customer configures the load balancer, enables sticky sessions, and defines target groups. Session tracking runs on AWS-managed infrastructure.

Layer 4 — Network Load Balancer: client-invoked. The customer defines listeners, protocols, ports, and targets. AWS operates the high-performance infrastructure behind it.

Layer 3 — VPC: client-invoked and largely client-managed. The customer defines CIDR ranges, subnets, route tables, and gateways. The actual packet routing fabric underneath is entirely AWS-managed and invisible to the customer.

Layer 2 — Elastic Network Interface: client-invoked. The customer creates ENIs, assigns IP addresses, attaches them to instances, and configures security groups. The virtual switching fabric that forwards frames between ENIs is entirely AWS-managed.

Layer 1 — AWS Direct Connect: split. The customer orders the connection, configures logical interfaces, and manages BGP sessions on their end. AWS owns and operates the physical infrastructure at the Direct Connect location — the racks, fibre, and cross-connects.

---

## ASSET:ccna 2026-11-16 → OSI layers: payload shape at each layer via AWS services

**Source:** Derived from Packt CCNA 200-301 Foundations — Course 1, OSI model lectures

Starting at Layer 7, the raw application data leaving the browser looks like this:

```
GET /home HTTP/1.1
Host: myapp.com
Cookie: session-id=abc123
Accept: text/html
```

At Layer 6, ACM and TLS wrap that HTTP request in encryption. The readable content disappears and becomes an opaque encrypted blob inside a TLS record:

```
TLS Record {
  Content-Type: Application Data
  Version: TLS 1.3
  Encrypted Payload: [x9f3a...encrypted HTTP data...c72b]
}
```

At Layer 5, ELB's sticky session mechanism attaches session tracking to the exchange. The payload itself does not change structurally but a session cookie was inserted in a prior response that the browser now carries forward — the dialog context rides in the HTTP header already present in the payload above.

At Layer 4, the NLB wraps everything so far into a TCP segment:

```
TCP Segment {
  Source Port: 54321
  Destination Port: 443
  Sequence Number: 1001
  Acknowledgement: 5001
  Flags: ACK
  Data: [TLS record containing encrypted HTTP request]
}
```

At Layer 3, the VPC wraps the TCP segment in an IP packet:

```
IP Packet {
  Source IP: 203.0.113.5
  Destination IP: 10.0.1.25
  Protocol: TCP
  TTL: 64
  Data: [TCP segment]
}
```

At Layer 2, the ENI wraps the IP packet in an Ethernet frame:

```
Ethernet Frame {
  Source MAC: aa:bb:cc:dd:ee:01
  Destination MAC: aa:bb:cc:dd:ee:02
  EtherType: IPv4
  Data: [IP packet]
  FCS: [checksum]
}
```

At Layer 1, Direct Connect transmits the frame as raw bits over the physical circuit — the entire Ethernet frame above becomes a stream of electrical or optical signals:

```
01000001 10110010 11001010 00110101 ...
```

The key thing to notice is the nesting — each layer wraps everything above it inside its own header, like envelopes inside envelopes. At the receiver, each layer strips its own envelope and passes the contents up, until the original HTTP request is unwrapped and readable at Layer 7. This is encapsulation and de-encapsulation in concrete form.

---

## ASSET:ccna 2026-11-16 → OSI layers mapped to AWS services: elaborated top to bottom

**Source:** Derived from Packt CCNA 200-301 Foundations — Course 1, OSI model lectures

The Application Load Balancer operates at Layer 7, meaning it can read and understand the actual content of HTTP and HTTPS requests before deciding where to send them. Unlike lower-layer load balancers that only see IP addresses and ports, the ALB can inspect the URL path, the hostname, the HTTP method, and even request headers. This allows it to route traffic from a single entry point to completely different backend services — sending requests to /api to one target group, /images to another, and everything else to a third. It is the intelligent traffic director sitting at the front of your application.

AWS Certificate Manager sits at Layer 6 because its job is encryption and format translation — specifically managing the SSL/TLS certificates that encrypt data moving between a client and your AWS services. When a user's browser connects to your application over HTTPS, ACM handles the certificate that proves your server's identity and establishes the encrypted channel. It provisions, renews, and deploys certificates automatically, removing the manual overhead of certificate management. The translation here is between plaintext and encrypted data — exactly the encoding and format concern of the Presentation layer.

Elastic Load Balancing at Layer 5 manages the session — the ongoing dialogue between a client and a backend instance. Its sticky session feature is the clearest Layer 5 behaviour: it ensures that once a user is connected to a particular backend instance, all subsequent requests from that user during their session are routed to the same instance. This matters for applications that store session state locally on the server rather than in a shared cache. ELB tracks and maintains these session associations across the lifetime of the user's visit.

The Network Load Balancer operates at Layer 4, working purely with TCP and UDP protocols and port numbers without reading any application-level content. It is designed for extreme performance — capable of handling millions of requests per second with very low latency. Because it operates at the transport layer it is the right choice for non-HTTP workloads such as database connections, gaming servers, or any application using raw TCP or UDP. It routes traffic based on the port number and protocol, forwarding segments to the appropriate backend target.

The Virtual Private Cloud is your private, isolated network inside AWS where you define your own IP address ranges, subnets, and routing tables. Everything in your AWS environment lives inside a VPC. At Layer 3 it is responsible for all IP addressing and routing decisions — which subnet a resource lives in, how traffic flows between subnets, and how it reaches the internet or other VPCs. The Internet Gateway, NAT Gateway, and Transit Gateway are all routing constructs that extend VPC's Layer 3 function — handling inbound and outbound internet routing and cross-VPC connectivity respectively.

An Elastic Network Interface is a virtual network card that operates at Layer 2. Every EC2 instance has at least one ENI attached, and that ENI is what holds the MAC address, the private IP address, the security group associations, and the source/destination check setting. Because it is a standalone resource in AWS you can create ENIs independently, attach them to instances, detach them and move them to different instances, and configure them with multiple IP addresses. The MAC address on an ENI is what identifies the instance within the local network segment, which is exactly the Data Link layer's role.

AWS Direct Connect is a physical, dedicated fibre connection between your on-premises data centre and AWS, bypassing the public internet entirely. At Layer 1 it is literally a cable — a private circuit provisioned through an AWS Direct Connect partner that terminates at an AWS Direct Connect location. It provides consistent bandwidth, predictable latency, and reduced data transfer costs compared to routing over the internet. For enterprises with heavy data transfer needs or strict compliance requirements around network path, Direct Connect is the physical foundation that everything else sits on top of.

---

## ASSET:ccna 2026-11-16 → OSI bottom four layers: definitions and AWS service mapping

**Source:** Packt CCNA 200-301 Foundations — Course 1, bottom four layers lecture

The bottom four layers are the bread and butter of network engineering. Each has its own dedicated section later in the course — the definitions here are what is needed for the exam.

Layer 4, the Transport layer, defines services to segment, transfer, and reassemble data for individual communications between end devices. It breaks large files into smaller segments that are less likely to cause transmission problems. TCP is used when reliability matters — it guarantees complete, ordered delivery. UDP is used when speed matters more than reliability, such as for voice or video traffic. The port number identifies which application the traffic belongs to — port 80 for HTTP, port 25 for SMTP.

Layer 3, the Network layer, provides connectivity and path selection between two host systems that may be located on geographically separated networks. It manages host connectivity through logical addressing — IP addressing is that logical addressing scheme. Routers are Layer 3 devices.

Layer 2, the Data Link layer, defines how data is formatted for transmission and how access to the physical media is controlled. It also typically includes error detection and correction to ensure reliable delivery. On Ethernet networks the Layer 2 address is the MAC address. Different Layer 2 technologies use different addressing formats — legacy Frame Relay used DLCI numbers. Switches are Layer 2 devices.

Layer 1, the Physical layer, enables bit transmission — the 1s and 0s — between end devices. It defines the specifications needed for activating, maintaining, and deactivating the physical link, including voltage levels, physical data rates, maximum transmission distances, and physical connectors.

The full OSI layer mapping with PDU names and corresponding AWS services is:

Layer 7 — Application — data — Application Load Balancer
Layer 6 — Presentation — translation — AWS Certificate Manager
Layer 5 — Session — dialog — Elastic Load Balancer
Layer 4 — Transport — segment — Network Load Balancer
Layer 3 — Network — packet — VPC
Layer 2 — Data Link — frame — Elastic Network Interface
Layer 1 — Physical — bits — AWS Direct Connect

For the Network layer, VPC is the strongest anchor since it owns the IP addressing and routing tables — Internet Gateway, NAT Gateway, and Transit Gateway are extensions of that routing function. For the Data Link layer, Elastic Network Interface is the precise service since it is what holds the MAC address and operates at Layer 2 — it is a standalone virtual network interface that can be created, configured, and attached to EC2 instances independently.

Mnemonic for PDU names top to bottom (data, translation, dialog, segment, packet, frame, bits): "Dedicated Teachers Develop Students Pushing Forward Boldly."

---

## ASSET:ccna 2026-11-16 → OSI upper layers: Layer 7, 6, and 5 definitions

**Source:** Packt CCNA 200-301 Foundations — Course 1, upper layers lecture

Network engineers do not typically work directly with the upper three layers but must know their definitions for the exam. Information at the upper layers includes things like the message body and subject line of an email.

Layer 7, the Application layer, provides network services directly to the end user's applications. It is distinct from every other OSI layer in that it does not provide services to any other layer — nothing sits above it. It establishes the availability of intended communication partners, confirming the other host is reachable and ready before communication begins. It also synchronizes and establishes agreement on procedures for error recovery and data integrity, where data integrity means verifying that data has not been altered or corrupted in transit.

Layer 6, the Presentation layer, ensures that information sent by the Application layer of one system is readable by the Application layer of another. Different computers can use different encoding schemes, and the Presentation layer translates between them using a common format, acting as a translator so the data arriving at the destination is interpreted correctly regardless of how it was formatted at the source.

Layer 5, the Session layer, establishes, manages, and terminates sessions between two communicating hosts. It synchronizes the dialogue between the Presentation layers of the two hosts and manages their data exchange. A web server handling many simultaneous users is a practical example — the Session layer keeps track of each user's session independently so their data does not get mixed up. It also handles efficient data transfer, Class of Service, and exception reporting for upper layer problems.

---

## ASSET:ccna 2026-11-16 → TCP/IP suite: protocol stack, four-layer model, and PDU terminology

**Source:** Packt CCNA 200-301 Foundations — Course 1, TCP/IP lecture

TCP/IP is not a single protocol but a protocol stack — a collection of multiple protocols working together, the most prominent being TCP (Transmission Control Protocol) and IP (Internet Protocol). It was developed in the 1960s by DARPA, the US Department of Defense Advanced Research Projects Agency. A protocol defines the rules governing how two parties communicate, so that any two hosts following the same protocol can exchange data regardless of who made them — similar to diplomatic protocol between countries.

While the OSI model is a conceptual reference framework, TCP/IP is what actually runs in production networks today. Other protocol stacks existed historically — IPX/SPX from Novell and AppleTalk from Apple among them — but they are all effectively dead now. TCP/IP is ubiquitous.

TCP/IP condenses the OSI model's seven layers into four. The Application layer maps to OSI layers 5, 6, and 7 combined. The Transport layer is a direct one-to-one equivalent of OSI Layer 4. The Internet layer is equivalent to OSI Layer 3. The Link layer covers both OSI Layer 2 and Layer 1 together.

In real-world networking conversations, engineers almost always reference the OSI model's seven layers rather than the TCP/IP four-layer model. The OSI terminology is the shared language of the industry. However, the PDU naming convention does come from TCP/IP and is used constantly in practice.

A protocol data unit (PDU) is the unit of communication at a given layer. The full mapping across all seven OSI layers is:

Layer 7 — Application — data
Layer 6 — Presentation — translation
Layer 5 — Session — dialog
Layer 4 — Transport — segment
Layer 3 — Network — packet
Layer 2 — Data Link — frame
Layer 1 — Physical — bits

Presentation uses "translation" to reflect its core job of converting between encoding schemes. Session uses "dialog" to reflect its role in synchronizing and managing the ongoing exchange between two hosts. All three collapse into a single Application layer in the TCP/IP model. The word "packet" is also used loosely in everyday conversation to mean the entire communication between two hosts (technically a PDU), but the precise layer-specific terms — frame, packet, segment — are used constantly by engineers and are worth knowing cold.

---

## ASSET:ccna 2026-11-16 → OSI model: seven layers, encapsulation, and de-encapsulation

**Source:** Packt CCNA 200-301 Foundations — Course 1, OSI model lecture

An ISO standard conceptual framework — not a protocol or physical technology — that divides network communication into seven ordered layers. Each layer serves the layer above it and is served by the layer below it.

Starting from the bottom, Layer 1 is the Physical layer, responsible for raw bit transmission over a physical medium — the network device associated with this layer is the hub, though hubs are essentially obsolete today. Layer 2 is the Data Link layer, which adds the source and destination MAC address to identify devices within a local network segment — switches operate here. Layer 3 is the Network layer, which adds the source and destination IP address to enable routing between networks — routers operate here. Layer 4 is the Transport layer, which adds TCP or UDP as the protocol along with a port number such as port 25 for SMTP email or port 80 for HTTP web traffic. Layer 5 is the Session layer, which manages the establishment, maintenance, and termination of communication sessions. Layer 6 is the Presentation layer, which handles encoding, formatting, compression, and encryption so the receiving system can correctly interpret the data. Layer 7 is the Application layer, which carries the actual user-facing data — in an email this is where the From and To fields live.

Layers 5 through 7 are collectively called the Upper Layers and are primarily the domain of application developers. Network engineers focus from Layer 4 downward.

The sender starts at Layer 7, creates the application data, then wraps it in a Layer 6 header, then wraps that in a Layer 5 header, continuing down through each layer. By the time it reaches Layer 1, the original data is nested inside multiple headers like envelopes inside envelopes.

The receiver strips one header at a time from the outside in, checking each layer before passing the payload upward. At Layer 2 it checks whether the destination MAC address matches its own — if not, the packet is discarded immediately. At Layer 3 it checks the destination IP address in the same way. At Layer 4 the port number tells the operating system which application the traffic belongs to, which is how a single server can run both an email service on port 25 and a web service on port 80 simultaneously.

Engineers can focus entirely on their own layer without needing to understand every other layer, as long as they comply with the interface standards between adjacent layers. This is what enables multi-vendor interoperability and open standards across the industry. For troubleshooting, the layered structure gives you a systematic bottom-up approach — a "Layer 1 problem" means the cable was unplugged, a "Layer 8 problem" is an informal term for user error.

Mnemonic (bottom-to-top): Please Do Not Throw Sausage Pizza Away — Physical, Data Link, Network, Transport, Session, Presentation, Application.

---

## ASSET:ccna 2026-11-14 → Packt CCNA 200-301 companion GitHub repository

**Repo:** https://github.com/packtpublishing/cisco-ccna-200-301-the-complete-guide-to-getting-certified

**Publisher:** Packt Publishing — companion code and resources for the Cisco CCNA 200-301: The Complete Guide to Getting Certified specialization (also delivered via Coursera, see ASSET-CCNA-2026Q3.md entry 2026-10-30).

---

## ASSET:ccna 2026-11-14 → Course orientation module: how to use the course

**Source:** Packt CCNA 200-301 Foundations — Course 1, orientation lecture

**Course structure:** Theory sections come first across all modules before any hands-on work. The intent is to build conceptual understanding before configuration — each section follows the pattern: theory video → instructor lab demo → student configuration lab.

**Labs:** Task-based, not step-by-step. The instructor gives a goal and expects the student to work it out using the earlier videos as reference. Each section's lab comes with two PDFs: a task sheet and a full answer key with explanation. Lab environment runs in Cisco Packet Tracer (free). Setup covered in the next section; estimated setup time 10–15 minutes.

**Slide PDFs:** Every theory lecture has a downloadable PDF of the slides. Written in full readable sentences (not short bullet points) so they function as standalone review material and act as flashcards near exam day — no need to build a separate note set.

**Supplementary resources:** Flackbox blog has free written tutorials with more depth than the videos. YouTube channel posts weekly. LinkedIn connection opens access to the instructor's network of thousands of people in the networking industry.

**Workflow:** Watch all lectures in order → do lab exercises after each section → use slide PDFs for exam review.

---

## ASSET:ccna 2026-11-14 → Study plan module: pacing and timetable approach

**Source:** Packt CCNA 200-301 Foundations — Course 1, study plan lecture

**What it is:** A downloadable PDF in the lecture's extra materials. Maps every course section and lecture into a sample timetable. Default schedule runs approximately two hours per night and reaches exam-ready in about six weeks.

**How to use it:** Do not fill in the full timetable upfront — interruptions will make a rigid pre-planned calendar demoralising. Fill in one week at a time, set targets that are hard but achievable, review progress at the start of each new week, then plan the next block. Blank pages after the sample template are provided for this rolling approach.

**Philosophy:** Skip days for genuinely important things without guilt. The plan exists to reduce unnoticed drift (the instructor's reference point: a colleague who was given a full resource list and still hadn't started a month later because of a soap opera). Consistent structured progress matters more than a perfect schedule.

**Practical action:** Download PDF from lecture extra materials. Use six-week sample as a pacing reference. Build week-by-week using blank pages, one week ahead at a time.

---

## ASSET:ccna 2026-11-14 → Recommended practice exams module: AlphaPrep

**Source:** Packt CCNA 200-301 Foundations — Course 1, recommended practice exams lecture

**Why practice tests are a separate skill from knowing the material:** CCNA exam failure isn't primarily a knowledge problem — it's a familiarity problem. The exam uses a specific question wording style that is genuinely difficult if you've never encountered it before. You can know every topic on the blueprint and still fail because you're seeing that format for the first time under time pressure. Practice tests make the question style familiar before the real exam, so on exam day your brain spends its energy recalling answers rather than parsing unfamiliar question structures.

**What AlphaPrep is and why it's recommended:** AlphaPrep is a dedicated exam prep platform with a question bank written by Cisco Press — the official Cisco certification publisher. That authorship matters because Cisco Press questions are written in the same style and at the same difficulty level as the actual exam. Beyond the question bank, AlphaPrep has an adaptive exam engine that uses machine learning to track performance across topics and adjust what it shows you. The instructor evaluated building something equivalent himself and concluded the engineering complexity was outside his capability, which is why it's a partner recommendation rather than a built-in course feature.

**The two scores — Exam Mastery and PassIQ:** Exam Mastery reflects overall coverage and performance across topic areas. PassIQ is the more important one — a predictive score estimating your probability of passing the real exam based on your performance patterns. When PassIQ reaches AlphaPrep's readiness threshold, it tells you you're ready. The real CCNA 200-301 exam costs $330 USD per attempt. PassIQ is designed to give a data-driven green light rather than a gut feeling, preventing an expensive premature attempt.

**Workflow within the course:** As you finish each course section, go to AlphaPrep → Build a Quiz → select the matching topic (organised by Cisco Press book order, slightly different from course order but straightforward to map). Do the topic quiz while the material is fresh to verify retention and flag gaps. After completing the full course and entering final prep, switch to Full Exams mode — these replicate the real exam in time limit, question count, and question type. Run full exams repeatedly, let PassIQ climb, and book the real exam when AlphaPrep indicates readiness.

**Pricing and affiliate disclosure:** $45/month via the instructor's affiliate link (flackbox.com/apccna). Signing up gives access to the CCNA 200-301 bank plus a free trial on other exam resources. The instructor discloses upfront he earns a commission but frames the recommendation as genuine — he has no equivalent alternative to offer inside the course. At $45/month against a $330 exam fee, one prevented failed attempt more than covers a month of access.

---

## ASSET:ccna 2026-11-14 → Packet Tracer installation and interface overview

**Source:** Packt CCNA 200-301 Foundations — Course 1, lab setup section: Packet Tracer install lecture

**Download:** From netacad.com — requires a free NetAcad account to access. Jay already has a NetAcad account from CCNA1 at AUT (2023), use those credentials. Minimum version required: 7.3. If already installed, check version before proceeding. Browser troubleshooting (clear cache or switch browsers) if login/download fails.

**Installation:** Accept licence → default install location → desktop shortcut → Install. Entirely next-next-next, no meaningful decisions. On first launch: decline multi-user, allow Windows Firewall, log in with NetAcad credentials inside the app, tick "stay logged in for 3 months."

**Interface basics:** Opens to a blank project by default — course lab files are pre-built and imported, no need to build from scratch. Device panel bottom-left (routers, switches, end devices etc.) → drag into workspace → Links tool to connect with appropriate cable type (straight-through Ethernet for switch-to-router) → click device → CLI tab to enter commands. That's the full workflow for all course labs.

**What follows in this section:** Next lecture is a Packet Tracer installation FAQ with common problems and updated download links. Lecture after that covers importing course lab files and using the lab environment.

---

## ASSET:ccna 2026-11-14 → Anki flashcards: installation and workflow

**Status:** Installed on this machine.

**Setup:** Download from apps.ankiweb.net (free on desktop, paid on mobile). Import course flashcard deck via File → Import — download the zip from the lecture, extract the `.apkg`, import it. Separate deck per course section appears after import.

**Workflow:** Do each section's flashcards immediately after finishing that section. Rate each card — Again (wrong), Hard (struggled), Good (got it), Easy (instant). Anki reschedules based on rating; hard cards resurface same day, easy cards space out over days. Each new card needs two correct answers before it moves to a next-day interval.

**Key tip:** As you complete each new section, drag its deck onto Section 3 in Anki to merge everything into one growing deck. That way every session reviews all completed material, not just the latest section. Daily use is recommended — spaced repetition only compounds if you show up consistently.

---

## ASSET:ccna 2026-11-14 → Packet Tracer .pkt file decryption: tools and test.pkt findings

**Context:** Packet Tracer 8.x saves `.pkt` files in an encrypted binary format (Twofish block cipher, CTR mode, EAX authenticated encryption). Files cannot be read outside of Packet Tracer without a decryption tool.

**Tools found:**
- [Unpacket](https://github.com/Punkcake21/Unpacket) — pure Python, decrypts PT 8.x files, outputs XML. Successfully tested on test.pkt.
- [ptexplorer](https://github.com/axcheron/ptexplorer) — covers older PT versions only, failed on PT 8.2.2 files (zlib header error).
- [pt-convert](https://github.com/semicolonmystery/pt-convert) — browser-based, no install, decrypts to XML or JSON.
- [packet-tracer-pkt-format](https://github.com/aacanadaa/packet-tracer-pkt-format) — byte-exact codec PT 4.1–9.0.1, Python API and CLI.

**How to run Unpacket:**
```
git clone https://github.com/Punkcake21/Unpacket.git
python3 Unpacket/unpacket.py <file.pkt>
```
Output XML is written alongside the input file (same directory, `.xml` extension).

**test.pkt contents:** Single-device starter file — one Cisco ISR4331 Router (Router0) inside a logical hierarchy: Root → Intercity → Home City → Corporate Office → Main Wiring Closet → Rack. No IP addresses, no links, no other devices. Bare save used to verify Packet Tracer was working, not an actual lab topology. Full topology in test.xml in the saves directory.

---

## ASSET:ccna 2026-11-14 → OSI model section: introduction

**Source:** Packt CCNA 200-301 Foundations — Course 1, OSI model section intro

The OSI (Open Systems Interconnection) model is a conceptual framework describing how networking protocols should function and interact. It doesn't define specific protocols — it's a reference model used to reason about networking. The instructor frames it as the bedrock of networking understanding: every technology covered later in the course maps onto one or more OSI layers, so learning the model first gives a structure for making sense of everything that follows. The section covers layer definitions first, then goes deeper on individual layers in later sections. The TCP/IP suite is also introduced alongside the OSI model — TCP/IP is what real networks actually run; OSI is the reference framework used to describe and analyse how those protocols behave.

---

## ASSET:ccna 2026-11-14 → Basic networking introduction: devices, LAN/WAN, and network characteristics

**Source:** Packt CCNA 200-301 Foundations — Course 1, basic networking intro lecture

**Core devices:** A switch provides connectivity within a LAN — multiple devices cable into it via Ethernet. A wireless access point extends LAN connectivity to Wi-Fi devices and also connects back into the switch. A router handles connectivity beyond the local network, making routing decisions to send traffic between different network areas including the internet. A firewall sits at the boundary between network areas and controls what traffic is allowed through.

**LAN vs WAN:** A Local Area Network connects devices in the same physical location (single office, campus). A Wide Area Network connects geographically separate sites — either over the internet (securely) or via a dedicated connection between routers at each site.

**Network characteristics:**
- **Topology** — how devices are physically and logically connected
- **Speed** — faster costs more
- **Cost** — affected by device types, network size, technologies used
- **Security** — not just the firewall; routers, switches, and dedicated devices (e.g. Intrusion Prevention Systems) all contribute
- **Availability** — critical components are duplicated to eliminate single points of failure
- **Scalability** — network should grow with the organisation without requiring a full redesign
- **Reliability** — the network consistently does what it's supposed to do

---
