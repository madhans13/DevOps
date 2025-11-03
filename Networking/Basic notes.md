# 🌐 Complete Network Request/Response Flow - Study Guide

A comprehensive guide to understanding how network requests work from your device to a server and back, including all layers, protocols, and hardware involved.

---

## 📚 Table of Contents

1. [Overview](#overview)
2. [Key Concepts](#key-concepts)
3. [OSI Model Layers](#osi-model-layers)
4. [Complete Request Flow](#complete-request-flow)
5. [Complete Response Flow](#complete-response-flow)
6. [Network Address Translation (NAT)](#network-address-translation-nat)
7. [MAC vs IP Addressing](#mac-vs-ip-addressing)
8. [Hardware Filtering](#hardware-filtering)
9. [Protocol Summary](#protocol-summary)
10. [Practical Examples](#practical-examples)

---

## 🎯 Overview

When you visit a website (e.g., `google.com`), a complex series of events happens across multiple network layers and devices. This guide explains every step in detail.

### High-Level Flow

```
Your Device → Router → Internet → Server
     ↓                              ↓
     ←      Router    ←  Internet  ← 
```

---

## 🔑 Key Concepts

### Socket
A socket is the combination of:
- **IP Address** (which device)
- **Port Number** (which application)
- **Protocol** (TCP/UDP)

**Example:** `192.168.1.5:54321/TCP`

### IP Addresses

**Private IP (Local Network):**
- Used within your home/office network
- Assigned by router (DHCP)
- Examples: `192.168.1.5`, `10.0.0.15`
- Not routable on internet

**Public IP (Internet):**
- Used on the internet
- Assigned by ISP
- Examples: `103.50.20.15`
- Visible to websites you visit

### MAC Address
- **Physical hardware address**
- Burned into network card during manufacturing
- Format: `AA:BB:CC:DD:EE:01`
- Used for local network delivery
- Never changes

### Ports

**Well-known Ports (0-1023):**
- HTTP: 80
- HTTPS: 443
- SSH: 22
- FTP: 21
- SMTP: 25
- DNS: 53

**Registered Ports (1024-49151):**
- Custom applications

**Dynamic/Ephemeral Ports (49152-65535):**
- Temporary ports for client connections
- OS assigns automatically

---

## 📊 OSI Model Layers

```
┌─────────────────────────────────────────────────────────────┐
│ Layer 7: Application Layer                                  │
│ Protocols: HTTP, HTTPS, FTP, SMTP, DNS                     │
│ What: User-facing protocols and data                       │
├─────────────────────────────────────────────────────────────┤
│ Layer 6: Presentation Layer                                │
│ What: Data formatting, encryption (TLS/SSL)                │
├─────────────────────────────────────────────────────────────┤
│ Layer 5: Session Layer                                     │
│ What: Session management, connections                      │
├─────────────────────────────────────────────────────────────┤
│ Layer 4: Transport Layer                                   │
│ Protocols: TCP, UDP                                        │
│ What: Port numbers, reliable delivery, flow control        │
├─────────────────────────────────────────────────────────────┤
│ Layer 3: Network Layer                                     │
│ Protocols: IP (IPv4, IPv6)                                 │
│ What: IP addressing, routing between networks              │
├─────────────────────────────────────────────────────────────┤
│ Layer 2: Data Link Layer                                   │
│ Protocols: Ethernet, WiFi (802.11)                         │
│ What: MAC addressing, local network delivery               │
├─────────────────────────────────────────────────────────────┤
│ Layer 1: Physical Layer                                    │
│ What: Physical transmission (radio waves, electrical)      │
└─────────────────────────────────────────────────────────────┘
```

### Protocol Stack Example

```
Application:    HTTP/HTTPS
Transport:      TCP
Network:        IP
Data Link:      Ethernet/WiFi
Physical:       Radio waves / Electrical signals
```

---

## 📤 Complete Request Flow

### Scenario: You visit `https://google.com` in your browser

```
┌──────────────────────────────────────────────────────────────┐
│ YOUR DEVICE (Laptop)                                         │
│ Private IP: 192.168.1.5                                      │
│ MAC Address: AA:BB:CC:DD:EE:01                              │
└──────────────────────────────────────────────────────────────┘
```

### Step 1: Application Layer (Layer 7)

**Your Browser (Chrome):**

```
User types: https://google.com

Browser creates HTTP request:
┌─────────────────────────────────┐
│ GET / HTTP/1.1                 │
│ Host: google.com               │
│ User-Agent: Chrome/120.0       │
│ Accept: text/html              │
└─────────────────────────────────┘
```

### Step 2: DNS Resolution (if needed)

```
Browser: "What's the IP for google.com?"
    ↓
DNS Query to 8.8.8.8:53 (Google DNS)
    ↓
DNS Response: "google.com = 142.250.185.46"
```

### Step 3: Transport Layer (Layer 4) - TCP

**Operating System creates TCP segment:**

```
┌─────────────────────────────────────┐
│ TCP Segment                        │
├─────────────────────────────────────┤
│ Source Port: 54321 (random)       │
│ Destination Port: 443 (HTTPS)     │
│ Sequence Number: 1000             │
│ Flags: SYN (connection request)   │
│ Data: [HTTP request]              │
└─────────────────────────────────────┘
```

### Step 4: Network Layer (Layer 3) - IP

**OS wraps TCP in IP packet:**

```
┌─────────────────────────────────────┐
│ IP Packet                          │
├─────────────────────────────────────┤
│ Source IP: 192.168.1.5            │
│ Destination IP: 142.250.185.46    │
│ Protocol: TCP                      │
│ TTL: 64                           │
│ Payload: [TCP Segment]            │
└─────────────────────────────────────┘
```

### Step 5: Data Link Layer (Layer 2) - Ethernet/WiFi

**Network card wraps IP packet in Ethernet frame:**

```
┌─────────────────────────────────────────────────┐
│ Ethernet Frame                                 │
├─────────────────────────────────────────────────┤
│ Destination MAC: AA:BB:CC:DD:EE:FF (Router)   │
│ Source MAC: AA:BB:CC:DD:EE:01 (Your laptop)   │
│ Type: IPv4 (0x0800)                           │
│ Payload: [IP Packet]                          │
│ CRC: [Checksum]                               │
└─────────────────────────────────────────────────┘
```

**Why Router's MAC?**
- Your device knows it needs to send to router (default gateway)
- ARP table maps router's IP (192.168.1.1) to its MAC

### Step 6: Physical Layer (Layer 1)

```
Network Card:
├─ Converts digital frame to radio waves (WiFi)
└─ Transmits via antenna 📡
```

### Step 7: Router Receives (Layer 2 → Layer 3)

```
┌──────────────────────────────────────────────────────────────┐
│ ROUTER                                                       │
│ Private IP (LAN side): 192.168.1.1                          │
│ Public IP (WAN side): 103.50.20.15                          │
│ MAC Address (LAN): AA:BB:CC:DD:EE:FF                        │
└──────────────────────────────────────────────────────────────┘

Router's WiFi card:
├─ Receives radio signal
├─ Checks MAC: "AA:BB:CC:DD:EE:FF == MY_MAC?" ✅ YES
├─ Passes to router's CPU
└─ Router reads IP packet
```

### Step 8: NAT Translation (Layer 3-4)

**Router performs Network Address Translation:**

```
BEFORE NAT (from your device):
┌─────────────────────────────────────┐
│ Source: 192.168.1.5:54321         │
│ Destination: 142.250.185.46:443   │
└─────────────────────────────────────┘

Router's NAT Table:
┌─────────────────────────────────────────────────────┐
│ Internal IP:Port  →  External Port  →  Destination │
│ 192.168.1.5:54321 →  12345         →  142.*.*.46:443│
└─────────────────────────────────────────────────────┘

AFTER NAT (to internet):
┌─────────────────────────────────────┐
│ Source: 103.50.20.15:12345        │ ← Changed!
│ Destination: 142.250.185.46:443   │
└─────────────────────────────────────┘
```

### Step 9: Internet Routing (Layer 3)

```
Router → ISP → Internet Backbone → Google's Network
         (Multiple routers using IP routing)

Each router:
├─ Reads destination IP: 142.250.185.46
├─ Checks routing table
├─ Forwards to next hop
└─ Decrements TTL
```

### Step 10: Google Server Receives

```
┌──────────────────────────────────────────────────────────────┐
│ GOOGLE SERVER                                                │
│ Public IP: 142.250.185.46                                    │
│ Port 443: HTTPS Server listening                            │
└──────────────────────────────────────────────────────────────┘

Server receives packet:
├─ Source: 103.50.20.15:12345 (your router's public IP)
├─ Destination: 142.250.185.46:443
└─ Processes HTTP request and prepares response
```

---

## 📥 Complete Response Flow

### Step 1: Google Server Sends Response (Layer 7-4)

```
HTTP Response:
┌─────────────────────────────────┐
│ HTTP/1.1 200 OK                │
│ Content-Type: text/html        │
│ Content-Length: 50000          │
│                                │
│ <html>...Google homepage...</html>│
└─────────────────────────────────┘

Wrapped in TCP:
├─ Source Port: 443
├─ Destination Port: 12345
└─ Acknowledgment, etc.

Wrapped in IP:
├─ Source IP: 142.250.185.46
└─ Destination IP: 103.50.20.15 (your router's public IP)
```

### Step 2: Internet Routing Back

```
Google → Internet → ISP → Your Router
         (Routers use destination IP: 103.50.20.15)
```

### Step 3: Router Receives Response (NAT Reverse Translation)

```
Router receives packet:
┌─────────────────────────────────────┐
│ Source: 142.250.185.46:443        │
│ Destination: 103.50.20.15:12345   │
└─────────────────────────────────────┘

Router checks NAT table:
"Port 12345 maps to 192.168.1.5:54321"

Router translates:
┌─────────────────────────────────────┐
│ Source: 142.250.185.46:443        │
│ Destination: 192.168.1.5:54321    │ ← Changed back!
└─────────────────────────────────────┘
```

### Step 4: Router Checks ARP Table

```
Router's ARP Table:
┌─────────────────────────────────┐
│ IP Address    →  MAC Address   │
│ 192.168.1.5   →  AA:BB:CC:DD:EE:01 │
└─────────────────────────────────┘

Router: "Need to send to 192.168.1.5"
        "That's MAC: AA:BB:CC:DD:EE:01"
```

### Step 5: Router Creates Ethernet Frame (Layer 2)

```
┌─────────────────────────────────────────────────┐
│ Ethernet Frame                                 │
├─────────────────────────────────────────────────┤
│ Destination MAC: AA:BB:CC:DD:EE:01 (Your laptop)│
│ Source MAC: AA:BB:CC:DD:EE:FF (Router)         │
│ Type: IPv4                                     │
│ Payload: [IP Packet with response]            │
└─────────────────────────────────────────────────┘
```

### Step 6: Router Broadcasts on WiFi (Layer 1)

```
Router's WiFi card:
├─ Converts frame to radio waves
└─ Broadcasts 📡

All devices in range receive the signal:
├─ Your Laptop
├─ Someone's Phone
└─ Someone's Tablet
```

### Step 7: Hardware Filtering at Each Device (Layer 2)

This is the **critical efficiency step!**

```
┌─────────────────────────────────────────────────────────────┐
│ YOUR LAPTOP'S WIFI CARD (Hardware)                          │
├─────────────────────────────────────────────────────────────┤
│ Receives radio signal                                       │
│ Converts to digital frame                                   │
│ Reads Destination MAC: AA:BB:CC:DD:EE:01                   │
│ Compares with MY_MAC: AA:BB:CC:DD:EE:01                    │
│ MATCH! ✅                                                    │
│ → Pass packet to CPU (via interrupt)                       │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ PHONE'S WIFI CARD (Hardware)                                │
├─────────────────────────────────────────────────────────────┤
│ Receives radio signal                                       │
│ Converts to digital frame                                   │
│ Reads Destination MAC: AA:BB:CC:DD:EE:01                   │
│ Compares with MY_MAC: AA:BB:CC:DD:EE:02                    │
│ NO MATCH! ❌                                                 │
│ → DROP packet (CPU never sees it)                          │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ TABLET'S WIFI CARD (Hardware)                               │
├─────────────────────────────────────────────────────────────┤
│ Receives radio signal                                       │
│ Converts to digital frame                                   │
│ Reads Destination MAC: AA:BB:CC:DD:EE:01                   │
│ Compares with MY_MAC: AA:BB:CC:DD:EE:03                    │
│ NO MATCH! ❌                                                 │
│ → DROP packet (CPU never sees it)                          │
└─────────────────────────────────────────────────────────────┘
```

**Key Point:** Only YOUR laptop's CPU is interrupted. Other devices filter in hardware!

### Step 8: Your Laptop's CPU Processes (Layer 3-7)

```
CPU receives interrupt from network card:
    ↓
Operating System Network Stack:
    ↓
┌─────────────────────────────────────────┐
│ Layer 2: Remove Ethernet frame         │
│ Layer 3: Check IP                      │
│   - Destination: 192.168.1.5           │
│   - MY_IP: 192.168.1.5                 │
│   - MATCH! ✅                           │
├─────────────────────────────────────────┤
│ Layer 4: Check TCP                     │
│   - Destination Port: 54321            │
│   - Check socket table:                │
│     Port 54321 → Chrome (PID: 1234)    │
├─────────────────────────────────────────┤
│ Layer 7: Extract HTTP response         │
│   - HTTP/1.1 200 OK                    │
│   - HTML content                       │
│   - Deliver to Chrome                  │
└─────────────────────────────────────────┘
    ↓
Chrome receives data:
    ↓
Renders Google homepage! 🎉
```

---

## 🔄 Network Address Translation (NAT)

### Why NAT Exists

**Problem:** Not enough IPv4 addresses (only ~4.3 billion)

**Solution:** NAT allows many devices to share one public IP

### How NAT Works

```
┌─────────────────────────────────────────────────────────────┐
│                     YOUR HOME NETWORK                       │
│                                                             │
│  Laptop:  192.168.1.5  (Private)                           │
│  Phone:   192.168.1.8  (Private)                           │
│  Tablet:  192.168.1.9  (Private)                           │
│                                                             │
│               ↓↓↓                                           │
│          [Router NAT]                                       │
│               ↓↓↓                                           │
│                                                             │
│      All share: 103.50.20.15 (Public)                      │
└─────────────────────────────────────────────────────────────┘
```

### NAT Translation Table

```
┌───────────────────────────────────────────────────────────────┐
│ Internal IP:Port  │ External Port │ Destination              │
├───────────────────────────────────────────────────────────────┤
│ 192.168.1.5:54321 │ 12345        │ 142.250.185.46:443 (Google)│
│ 192.168.1.8:49152 │ 12346        │ 208.65.153.238:443 (YouTube)│
│ 192.168.1.9:50001 │ 12347        │ 157.240.241.35:443 (Facebook)│
└───────────────────────────────────────────────────────────────┘
```

### NAT Process

**Outgoing (Your Request):**
```
1. Packet leaves your device:
   Source: 192.168.1.5:54321
   
2. Router NAT changes it:
   Source: 103.50.20.15:12345
   Records: 12345 → 192.168.1.5:54321
   
3. Internet sees:
   Source: 103.50.20.15:12345
```

**Incoming (Response):**
```
1. Response arrives at router:
   Destination: 103.50.20.15:12345
   
2. Router checks NAT table:
   12345 → 192.168.1.5:54321
   
3. Router changes it:
   Destination: 192.168.1.5:54321
   
4. Delivers to your device
```

### NAT Entry Lifespan

```
Created: When connection starts
Active: While data flows or recent activity
Expires: After timeout (TCP: few minutes, UDP: 30-60 seconds)
Deleted: Frees external port for reuse
```

---

## 🆚 MAC vs IP Addressing

### Comparison Table

| Feature | MAC Address | IP Address |
|---------|-------------|------------|
| **Layer** | Data Link (Layer 2) | Network (Layer 3) |
| **Scope** | Local network only | Global (internet-wide) |
| **Format** | AA:BB:CC:DD:EE:01 | 192.168.1.5 |
| **Assignment** | Manufacturer (permanent) | DHCP (temporary) |
| **Changes** | Never | Yes (DHCP lease) |
| **Purpose** | Physical delivery | Logical routing |
| **Used by** | Switches, WiFi cards | Routers, IP stack |
| **Filtering** | Hardware (fast) | Software (slower) |

### Why Both Are Needed

```
┌──────────────────────────────────────────────────────────┐
│ IP ADDRESS (Logical)                                     │
│ - Identifies device on network                          │
│ - Routes across different networks                      │
│ - Works globally                                        │
│ - Example: "Send to 192.168.1.5"                       │
└──────────────────────────────────────────────────────────┘
         ↓ (needs physical delivery)
┌──────────────────────────────────────────────────────────┐
│ MAC ADDRESS (Physical)                                   │
│ - Identifies network card hardware                       │
│ - Delivers on local network                             │
│ - Works only on same network segment                    │
│ - Example: "Deliver to network card AA:BB:CC:DD:EE:01" │
└──────────────────────────────────────────────────────────┘
```

### ARP: Bridging IP and MAC

**ARP (Address Resolution Protocol)** maps IP addresses to MAC addresses:

```
Scenario: Router needs to send to 192.168.1.5

1. Router checks ARP cache:
   ├─ Found: 192.168.1.5 → AA:BB:CC:DD:EE:01 ✅
   └─ Use that MAC address

2. If not in cache:
   ├─ Router broadcasts: "Who has 192.168.1.5?"
   ├─ Device with that IP replies: "That's me! MAC: AA:BB:CC:DD:EE:01"
   ├─ Router stores in ARP cache
   └─ Uses MAC address
```

**View ARP Table:**
```bash
# Windows
arp -a

# Mac/Linux
arp -n
```

---

## 🔧 Hardware Filtering

### Why Hardware Filtering is Critical

**Without Hardware Filtering (Hypothetical):**

```
10,000 packets/second on WiFi network
    ↓
ALL packets go to your CPU
    ↓
CPU checks each: "Is this for 192.168.1.5?"
    ↓
9,900 packets discarded (wasted CPU work)
100 packets processed

Result:
❌ CPU usage: 95%
❌ Battery drain: Massive
❌ Performance: Terrible
❌ Heat: Device gets hot
```

**With Hardware Filtering (Current Reality):**

```
10,000 packets/second on WiFi network
    ↓
Network card filters by MAC in hardware
    ↓
9,900 packets: ❌ Dropped (MAC doesn't match)
100 packets: ✅ Passed to CPU (MAC matches)
    ↓
CPU processes only 100 packets

Result:
✅ CPU usage: 5%
✅ Battery: Normal
✅ Performance: Fast
✅ Heat: Cool
```

### Network Card Architecture

```
┌──────────────────────────────────────────────────────────┐
│               NETWORK CARD (WiFi/Ethernet)               │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────┐      ┌──────────────────┐              │
│  │  Antenna/  │  →   │  Radio/Signal    │              │
│  │   Port     │      │  Processing      │              │
│  └────────────┘      └──────────────────┘              │
│                              ↓                           │
│                      ┌──────────────────┐               │
│                      │  MAC Filter Chip │ ← HARDWARE!   │
│                      │  (Hardwired)     │               │
│                      └──────────────────┘               │
│                              ↓                           │
│                      Does MAC match?                     │
│                    ┌─────┴─────┐                        │
│                   NO           YES                       │
│                    ↓             ↓                       │
│                 [DROP]    [Buffer Memory]                │
│                                  ↓                       │
│                           [DMA to RAM]                   │
│                                  ↓                       │
│                         [Interrupt CPU] ← Only if match! │
│                                                          │
└──────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────┐
│                    CPU / OPERATING SYSTEM                │
│                                                          │
│  - Woken up by interrupt                                │
│  - Processes packet (check IP, Port)                    │
│  - Delivers to application                              │
└──────────────────────────────────────────────────────────┘
```

### Performance Comparison

```
MAC Filtering (Hardware):
├─ Speed: ~5 nanoseconds per packet
├─ Location: Network card chip
├─ CPU usage: Zero
└─ Power: Milliwatts

IP Filtering (Software):
├─ Speed: ~1000 nanoseconds per packet (200x slower)
├─ Location: CPU/Operating System
├─ CPU usage: High
└─ Power: Watts (1000x more)
```

### Network Card Modes

**1. Normal Mode (Default):**
```
- Accepts only packets with matching MAC
- Most efficient
- Used for regular networking
```

**2. Promiscuous Mode:**
```
- Accepts ALL packets (no MAC filtering)
- Used for network monitoring/debugging
- Tools: Wireshark, tcpdump
- Requires admin privileges
- Heavy CPU usage

# Enable (Linux):
sudo ifconfig wlan0 promisc
```

**3. Broadcast Mode:**
```
- Accepts broadcast MAC: FF:FF:FF:FF:FF:FF
- Enabled by default
- Used for: DHCP, ARP
```

**4. Multicast Mode:**
```
- Accepts specific multicast addresses
- Used for: Streaming, group communication
```

---

## 📋 Protocol Summary

### Application Layer Protocols

```
HTTP (Hypertext Transfer Protocol)
├─ Port: 80
├─ Transport: TCP
├─ Encryption: None
└─ Use: Web browsing (insecure)

HTTPS (HTTP Secure)
├─ Port: 443
├─ Transport: TCP
├─ Encryption: TLS/SSL
└─ Use: Secure web browsing

FTP (File Transfer Protocol)
├─ Port: 21 (control), 20 (data)
├─ Transport: TCP
└─ Use: File transfer

SMTP (Simple Mail Transfer Protocol)
├─ Port: 25
├─ Transport: TCP
└─ Use: Sending email

DNS (Domain Name System)
├─ Port: 53
├─ Transport: UDP (usually), TCP (large queries)
└─ Use: Domain name resolution

SSH (Secure Shell)
├─ Port: 22
├─ Transport: TCP
└─ Use: Secure remote access

DHCP (Dynamic Host Configuration Protocol)
├─ Port: 67 (server), 68 (client)
├─ Transport: UDP
└─ Use: IP address assignment
```

### Transport Layer Protocols

```
TCP (Transmission Control Protocol)
├─ Connection-oriented
├─ Reliable (guarantees delivery)
├─ Ordered delivery
├─ Error checking
├─ Flow control
├─ Use: HTTP, HTTPS, FTP, SSH, Email
└─ Example: Web browsing, file downloads

UDP (User Datagram Protocol)
├─ Connectionless
├─ Unreliable (no delivery guarantee)
├─ No ordering
├─ Faster than TCP
├─ Use: DNS, DHCP, streaming, gaming, VoIP
└─ Example: Video calls, online games
```

### TCP vs UDP

| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | Connection-oriented | Connectionless |
| **Reliability** | Guaranteed delivery | No guarantee |
| **Ordering** | Packets in order | No ordering |
| **Speed** | Slower (overhead) | Faster |
| **Error Checking** | Yes (retransmission) | Basic checksum only |
| **Use Cases** | Web, email, file transfer | Streaming, gaming, DNS |
| **Overhead** | Higher | Lower |

### Stateless vs Stateful

**HTTP is Stateless:**
```
Request 1: GET /homepage
Server: "Here's homepage" [forgets you]

Request 2: GET /profile
Server: "Who are you?" [doesn't remember Request 1]

Solution: Cookies, Sessions, Tokens
```

**TCP is Stateful:**
```
- Tracks connection state
- Remembers sequence numbers
- Maintains connection until closed
```

---

## 💡 Practical Examples

### Example 1: Browsing a Website

```
You: Type "https://github.com" in browser
    ↓
┌─────────────────────────────────────────────────────────┐
│ 1. DNS Resolution                                       │
│    Browser → DNS Server (8.8.8.8:53)                   │
│    Query: "What's the IP for github.com?"              │
│    Response: "140.82.121.4"                            │
└─────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────┐
│ 2. TCP Connection (3-Way Handshake)                    │
│    You → Server: SYN (synchronize)                     │
│    Server → You: SYN-ACK (acknowledge)                 │
│    You → Server: ACK (acknowledge)                     │
│    Connection established! ✅                           │
└─────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────┐
│ 3. TLS Handshake (HTTPS)                               │
│    Exchange encryption keys                            │
│    Verify server certificate                           │
│    Establish encrypted tunnel                          │
└─────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────┐
│ 4. HTTP Request                                         │
│    GET / HTTP/1.1                                      │
│    Host: github.com                                    │
│    (Encrypted by TLS)                                  │
└─────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────