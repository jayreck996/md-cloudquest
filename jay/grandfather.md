# Grandfather Doc — Jay's Study Reference

**Author:** Jay Reck
**Updated:** 2026-11-16
**Tracks:** CCNA 200-301 | AWS CloudQuest

---

## CCNA 200-301

### Study Plan and Course Status

Goal: complete the CCNAv7 three-course curriculum through Cisco Networking Academy and prepare for the CCNA 200-301 exam.

CCNA1 (ITN — Introduction to Networks): COMPLETED 2023-06-27 at Auckland University of Technology.
CCNA2 (SRWE — Switching, Routing, and Wireless Essentials): TODO.
CCNA3 (ENSA — Enterprise Networking, Security, and Automation): TODO.

Primary course: Cisco CCNA 200-301: The Complete Guide to Getting Certified (Packt, Coursera). Financial aid available per course — apply separately, up to 75% off, review takes up to 16 days. Companion GitHub repo: https://github.com/packtpublishing/cisco-ccna-200-301-the-complete-guide-to-getting-certified

Course workflow: watch all lectures in order, do lab exercises after each section, use slide PDFs for exam review. Anki flashcards installed — do each section's deck immediately after finishing that section and merge into one growing deck.

Practice exams: AlphaPrep ($45/month, flackbox.com/apccna) — Cisco Press question bank, adaptive engine, PassIQ score predicts exam readiness. Real exam costs $330 USD per attempt. Use AlphaPrep topic quizzes after each section, full exams in final prep.

---

### OSI Model

An ISO standard conceptual framework dividing network communication into seven ordered layers. Each layer serves the layer above it and is served by the layer below it. It is not a protocol or physical technology — it is a reference model used to reason about networking.

Layer 1 — Physical — bits — AWS Direct Connect
Layer 2 — Data Link — frame — Elastic Network Interface
Layer 3 — Network — packet — VPC
Layer 4 — Transport — segment — Network Load Balancer
Layer 5 — Session — dialog — Elastic Load Balancer
Layer 6 — Presentation — translation — AWS Certificate Manager
Layer 7 — Application — data — Application Load Balancer

Mnemonic bottom-to-top: Please Do Not Throw Sausage Pizza Away.
Mnemonic PDU top-to-bottom (data, translation, dialog, segment, packet, frame, bits): Dedicated Teachers Develop Students Pushing Forward Boldly.

Encapsulation: sender starts at Layer 7, wraps data in a header at each layer going down. De-encapsulation: receiver strips one header at a time from the outside in, checking each layer before passing the payload up. Layer 2 checks destination MAC — discards if not matching. Layer 3 checks destination IP — discards if not matching. Layer 4 port number routes traffic to the correct application.

Troubleshooting: work bottom-up. Layer 1 problem = cable unplugged. Layer 8 = user error (informal).

---

### OSI Layer Definitions

Layer 7 — Application: provides network services directly to end user applications. Does not provide services to any other layer. Establishes availability of communication partners, synchronizes error recovery procedures, and verifies data integrity.

Layer 6 — Presentation: ensures information sent by one system's Application layer is readable by another's. Translates between different encoding schemes using a common format.

Layer 5 — Session: establishes, manages, and terminates sessions between two communicating hosts. Tracks multiple simultaneous sessions on a server independently. Handles efficient data transfer, Class of Service, and upper layer exception reporting.

Layer 4 — Transport: segments, transfers, and reassembles data for individual communications between end devices. Breaks large files into smaller segments. Tracks sessions via port numbers. TCP or UDP. Flow control.

Layer 3 — Network: provides connectivity and path selection between geographically separated hosts. Manages logical addressing (IP). Routers are Layer 3 devices.

Layer 2 — Data Link: defines how data is formatted for transmission and controls physical media access. Includes error detection and correction. MAC addressing on Ethernet. Switches are Layer 2 devices.

Layer 1 — Physical: enables bit transmission between end devices. Defines voltage levels, data rates, maximum distances, physical connectors.

---

### TCP/IP Suite

A protocol stack — not a single protocol — developed in the 1960s by DARPA. Condenses OSI's seven layers into four: Application (OSI 5+6+7), Transport (OSI 4), Internet (OSI 3), Link (OSI 1+2). Real networks run TCP/IP; OSI is the reference language engineers use to discuss it.

---

### TCP vs UDP

TCP: connection-oriented, sequenced, reliable, flow control. Three-way handshake: SYN → SYN-ACK → ACK. Large header with sequence numbers, acknowledgements, window, checksum. Used where reliability matters — web, email, file transfer.

UDP: connectionless, unsequenced, unreliable, no flow control. Minimal header — source port, destination port, length, checksum only. Used for real-time traffic — voice, video — where latency matters more than reliability.

Common ports: FTP 21 TCP, SSH 22 TCP, Telnet 23 TCP, HTTP 80 TCP, HTTPS 443 TCP, TFTP 69 UDP, SNMP 161 UDP, DNS 53 TCP+UDP.

Port 80 = plain HTTP (unencrypted). Port 443 = HTTPS (HTTP wrapped in TLS). Most modern sites redirect port 80 to 443.

---

### Layer 3 — Network Layer and IP Header

IP is the primary Layer 3 protocol. Connectionless — no acknowledgements at Layer 3. Reliability comes from TCP at Layer 4 if needed. Other Layer 3 protocols: ICMP (used for ping), IPSec (encrypted communications).

IP addressing is logical — designed by the network administrator. Divides the network into subnets to improve performance, security, and troubleshooting. MAC addressing at Layer 2 is a flat global scheme; IP addressing at Layer 3 is hierarchical and partitioned.

IP header fields: version (IPv4 or IPv6), header length, Type of Service (QoS marking), total length, fragmentation fields (Ethernet MTU default 1500 bytes — packets larger than MTU are fragmented), TTL (decremented by each router, dropped at 0 to prevent routing loops), protocol (TCP or UDP), checksum, source IP, destination IP, options, data.

---

### Cisco IOS Variants

Cisco founded 1984, originally routing only. Acquired Crescendo 1993 — produced Catalyst switch line (original CatOS now deprecated, switches run IOS). Acquired Network Translation 1995 — PIX firewall (Finesse OS, evolved to ASA, now runs IOS). Cisco standardised on IOS across all core infrastructure.

IOS: enterprise routers and switches. NX-OS: Nexus and MDS data centre switches. IOS-XR: high-end service provider routers (NCS, CRS, ASR9K, XR12K). IOS-XE: ASR1K service provider routers.

CLI experience is nearly identical across all variants. The difference is architectural: IOS uses a monolithic kernel — one process crash can bring down the whole router. NX-OS, IOS-XR, IOS-XE use microkernel architecture — processes run in isolated protected memory, one crash does not affect the rest. This matters at service provider scale. For enterprise, IOS is purpose-built, hardened, and extremely reliable. CCNA exam covers IOS only.

---

### Connecting to Cisco Devices

Network connection (day-to-day): SSH over the management IP address using PuTTY (free, putty.org). SSH encrypts the entire session. Telnet is identical in appearance but sends everything in plaintext — never used in production. Authentication is centralised through an AAA server (Authentication, Authorization, Accounting) — holds all usernames and passwords so they do not need to be configured on each device separately.

In-band management: connecting over the production network (same cables as normal user traffic). Out-of-band management: separate dedicated management network — provides a backup path if the production network fails, and is more secure.

Console cable (initial setup and troubleshooting): direct low-level connection independent of IP. Traditional cable: RJ45 into device console port, DB9 serial into laptop — requires USB-to-serial adapter on modern laptops. Newer devices: USB-to-mini-USB cable. PuTTY settings: Serial connection type, COM3 (check Device Manager), 9600 baud, 8 data bits, 1 stop bit, no parity, no flow control. Console is also the tool of choice when a device is unreachable over the network or failing to boot.

---

### IOS CLI Navigation

Three main modes:

User exec — `Router>` — read-only, limited commands.
Privileged exec — `Router#` — entered with `enable` (abbreviated `en`). Show commands and debugs run here.
Global configuration — `Router(config)#` — entered with `configure terminal` (abbreviated `conf t`). All configuration changes made here.

Sub-levels beneath global config: `interface fastethernet 0/0` enters interface config mode `Router(config-if)#`. Must go deeper one level at a time. `exit` drops one level. `end` or Control+C returns to enable prompt in one step.

Question mark shows context-sensitive help at any point. Spacebar scrolls output page by page. Control+C breaks out of output. Up arrow cycles through command history (scoped to current mode). Control+A jumps cursor to start of line. Tab completes a unique partial command. Command abbreviation works as long as the input is unambiguous.

The `do` prefix runs show commands from inside global config or any sub-level — `do show ip interface brief` works from anywhere.

Error messages: `invalid input detected` with caret = typo or wrong level. `incomplete command` = needs more keywords.

Most used show commands: `show ip interface brief` (all interfaces, IPs, up/down status). `show running-config` (full active config). Pipe filters: `| begin`, `| include`, `| exclude`, `| section` — case sensitive (regular expressions).

Show commands = point-in-time snapshot. Debug commands = real-time output as events occur — use carefully in production.

---

### IOS Configuration Management

Two config files: running config (active, in RAM, volatile) and startup config (loaded on boot, in NVRAM, persistent). Changes take effect immediately in running config but are not saved to startup config until explicitly copied.

Save changes: `copy running-config startup-config` (abbreviated `copy run start`) — must be run from privileged exec.

Boot sequence: IOS loads from flash into RAM → startup config loads from NVRAM into RAM as running config.

Backup to TFTP: `copy running-config tftp` — prompts for server IP and filename. Include date in filename. Never back up to flash on the same device.

Restore: `write erase` (older IOS) or `erase startup-config` (newer) → copy backup to startup config → reboot. Erase first — copying merges rather than replaces.

View flash file: `more flash:filename`.

---

### OSI Layers Mapped to AWS Services

Layer 7 — Application — Application Load Balancer: routes traffic based on HTTP/HTTPS content — URL path, hostname, method, headers. Fully client-configured.

Layer 6 — Presentation — AWS Certificate Manager: manages SSL/TLS certificates, handles TLS handshake, encrypts/decrypts traffic. Client requests and attaches; AWS auto-renews.

Layer 5 — Session — Elastic Load Balancer: sticky sessions keep a user routed to the same backend instance throughout their visit. Client-configured; session tracking runs on AWS infrastructure.

Layer 4 — Transport — Network Load Balancer: routes TCP/UDP traffic by port number. Extreme performance — millions of requests per second. Client-configured.

Layer 3 — Network — VPC: defines IP address ranges, subnets, route tables. Internet Gateway, NAT Gateway, Transit Gateway extend its routing function. Client-managed; underlying routing fabric is AWS-managed.

Layer 2 — Data Link — Elastic Network Interface: virtual NIC with MAC address, private IP, security groups. Standalone AWS resource — can be created, attached, detached independently. Virtual switching fabric is AWS-managed.

Layer 1 — Physical — AWS Direct Connect: dedicated physical fibre circuit between on-premises and AWS. Customer manages logical interfaces and BGP. AWS owns the physical infrastructure at the Direct Connect location.

---

### Payload Shape — HTTPS GET Example

```
Layer 7 — Application:
GET /home HTTP/1.1
Host: myapp.com
Cookie: session-id=abc123
Accept: text/html

Layer 6 — Presentation:
TLS Record {
  Content-Type: Application Data
  Version: TLS 1.3
  Encrypted Payload: [x9f3a...c72b]
}

Layer 5 — Session:
ELB sticky session cookie carried in HTTP header — dialog tracked across requests

Layer 4 — Transport:
TCP Segment {
  Source Port: 54321
  Destination Port: 443
  Sequence Number: 1001
  Acknowledgement: 5001
  Flags: ACK
  Data: [TLS record]
}

Layer 3 — Network:
IP Packet {
  Source IP: 203.0.113.5
  Destination IP: 10.0.1.25
  Protocol: TCP
  TTL: 64
  Data: [TCP segment]
}

Layer 2 — Data Link:
Ethernet Frame {
  Source MAC: aa:bb:cc:dd:ee:01
  Destination MAC: aa:bb:cc:dd:ee:02
  EtherType: IPv4
  Data: [IP packet]
  FCS: [checksum]
}

Layer 1 — Physical:
01000001 10110010 11001010 00110101 ...
```

Full 10-example payload table (HTTPS, DNS, SSH, FTP, SMTP, HTTP POST, SNMP, TFTP, VoIP, Video): `ccna/COULD/TABLE-2026Q4.csv`

---

## AWS CloudQuest

### Subscription and Badges

AWS Skill Builder Individual Annual Subscription — free complimentary tier, active Sep 15 2026 to Sep 14 2027, does not auto-renew.

Badges earned:
- AWS Cloud Quest: Generative AI Architect — October 16, 2026
- AWS Cloud Quest: Security — November 5, 2026 (Credly: https://www.credly.com/earner/earned/badge/94957dfe-68d7-46d8-9503-e5eba8e0a9db)

---

### CloudFormation

S3 URL auto-population: when you upload a template through the console, CloudFormation copies it to S3 and fills in the URL automatically. That S3 copy is the persistent source of truth used for drift detection, updates, and rollbacks throughout the stack's lifetime.

Infrastructure Composer: visual drag-and-drop designer showing the template as a resource diagram. The Edit in Infrastructure Composer button stays available throughout the deployment flow — return to it to fix issues before committing to stack creation.

Rollback on failure: CloudFormation deployments are all-or-nothing. If any resource fails — syntax error, timeout, insufficient IAM permissions — CloudFormation deletes everything it created and returns the environment to its pre-deployment state. This prevents orphaned resources. Disable rollback when troubleshooting to inspect the partial deployment state; manually delete the partial stack and re-deploy with rollback re-enabled after diagnosing.

---

### Key Lab References

Full lab notes with DIY steps, grader gotchas, and config details are in `cloudquest/MIGHT/ASSET-CLOUDQUEST-2026Q3.md`. Covers: Cognito Pre-SignUp Lambda trigger, Glue crawler, CloudWatch/SNS email subscription, Service Catalog AppStack, API Gateway Cognito authorizer, KMS multi-region keys, envelope encryption, IAM/S3 CLI labs, ECS scale-out, Lambda guardrail logging, security groups, EC2 key pairs.
