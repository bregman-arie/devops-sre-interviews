# Networking - Beginner Interview Mock #1 - Answer Key

> **Difficulty:** Beginner  
> **Duration:** ~30 minutes  
> **Goal:** Assess your understanding of basic networking concepts, protocols, and troubleshooting.

---

## 🧠 Section 1: Core Questions - Answers

### 1. What is the difference between TCP and UDP? When would you use each?

**Answer:** 
- **TCP (Transmission Control Protocol)**:
  - **Connection-oriented**: Establishes a connection before data transfer
  - **Reliable**: Guarantees data delivery and order
  - **Error checking**: Built-in error detection and correction
  - **Flow control**: Manages data transmission rate
  - **Use cases**: Web browsing (HTTP/HTTPS), email (SMTP), file transfers (FTP), SSH

- **UDP (User Datagram Protocol)**:
  - **Connectionless**: No connection establishment needed
  - **Unreliable**: No guarantee of delivery or order
  - **Faster**: Lower overhead, minimal error checking
  - **Use cases**: DNS queries, video streaming, online gaming, DHCP

### 2. Explain what DNS is and how it works.

**Answer:** DNS (Domain Name System) is like the internet's phone book that translates human-readable domain names into IP addresses.

**How it works:**
1. **User types** a domain name (e.g., google.com)
2. **Local DNS cache** is checked first
3. **Recursive resolver** queries root nameservers
4. **Root servers** point to TLD (Top-Level Domain) servers (.com)
5. **TLD servers** point to authoritative nameservers
6. **Authoritative servers** return the IP address
7. **IP address** is returned to the user's browser

### 3. What is the difference between a router, a switch, and a hub?

**Answer:**
- **Hub** (mostly obsolete):
  - **Layer 1** (Physical layer) device
  - **Half-duplex**: Only one device can transmit at a time
  - **Collision domain**: All ports share the same collision domain
  - **Security**: No security features

- **Switch**:
  - **Layer 2** (Data Link layer) device
  - **Full-duplex**: Simultaneous send/receive
  - **MAC address table**: Learns and stores device locations
  - **Collision domains**: Each port has its own collision domain

- **Router**:
  - **Layer 3** (Network layer) device
  - **Routes traffic** between different networks
  - **IP routing**: Makes decisions based on IP addresses
  - **Network segmentation**: Separates broadcast domains

### 4. What are the different OSI model layers? Can you name at least 4?

**Answer:** The OSI (Open Systems Interconnection) model has 7 layers:

1. **Physical Layer** - Hardware, cables, electrical signals
2. **Data Link Layer** - MAC addresses, switches, frame handling
3. **Network Layer** - IP addresses, routing, routers
4. **Transport Layer** - TCP/UDP, port numbers, end-to-end delivery
5. **Session Layer** - Session management, connections
6. **Presentation Layer** - Encryption, compression, data formatting
7. **Application Layer** - HTTP, FTP, SMTP, user applications

**Memory trick**: "Please Do Not Throw Sausage Pizza Away"

### 5. What is an IP address and what's the difference between IPv4 and IPv6?

**Answer:**
An **IP address** is a unique identifier assigned to devices on a network to enable communication.

**IPv4**:
- **32-bit** address (4 bytes)
- **Format**: 192.168.1.1 (dotted decimal notation)
- **Address space**: ~4.3 billion unique addresses
- **Problem**: Address exhaustion

**IPv6**:
- **128-bit** address (16 bytes)
- **Format**: 2001:0db8:85a3:0000:0000:8a2e:0370:7334
- **Address space**: 340 undecillion addresses
- **Benefits**: No address exhaustion, built-in security, simplified header

### 6. What is a subnet and why is subnetting useful?

**Answer:**
A **subnet** is a logical subdivision of an IP network that allows you to break a large network into smaller, manageable segments.

**Benefits of subnetting:**
- **Improved performance**: Reduces broadcast traffic
- **Enhanced security**: Isolates network segments
- **Better organization**: Groups related devices
- **Efficient IP usage**: Reduces IP address waste
- **Easier management**: Simplifies network administration
- **Reduced congestion**: Limits broadcast domains

**Example**: 192.168.1.0/24 can be divided into:
- 192.168.1.0/26 (64 addresses)
- 192.168.1.64/26 (64 addresses)
- 192.168.1.128/26 (64 addresses)
- 192.168.1.192/26 (64 addresses)

### 7. What is DHCP and how does it work?

**Answer:** 
**DHCP (Dynamic Host Configuration Protocol)** automatically assigns IP addresses and network configuration to devices.

**DHCP Process (DORA)**:
1. **Discover**: Client broadcasts "I need an IP address"
2. **Offer**: DHCP server responds with available IP address
3. **Request**: Client requests the offered IP address
4. **Acknowledge**: Server confirms and assigns the IP address

**Information provided by DHCP:**
- IP address
- Subnet mask
- Default gateway
- DNS servers
- Lease time

### 8. What's the difference between a public and private IP address?

**Answer:**

**Public IP addresses:**
- **Globally unique** and routable on the internet
- **Assigned by ISPs** and managed by IANA
- **Examples**: 8.8.8.8, 1.1.1.1, 74.125.224.72
- **Limited supply** (especially IPv4)

**Private IP addresses:**
- **Used within local networks** only
- **Not routable** on the public internet
- **Reusable** across different organizations
- **Defined ranges (RFC 1918)**:
  - Class A: 10.0.0.0 to 10.255.255.255
  - Class B: 172.16.0.0 to 172.31.255.255
  - Class C: 192.168.0.0 to 192.168.255.255

---

## 🔧 Section 2: Practical Questions - Answers

### 9. How would you troubleshoot a situation where you can't access a website?

**Answer:** Follow a systematic approach:

1. **Check local connectivity:**
   ```bash
   ping 8.8.8.8  # Test internet connectivity
   ```

2. **Test DNS resolution:**
   ```bash
   nslookup example.com
   ping example.com
   ```

3. **Check routing:**
   ```bash
   traceroute example.com
   ```

4. **Verify network configuration:**
   ```bash
   ipconfig /all  # Windows
   ip addr show   # Linux
   ```

5. **Test different browsers/devices**
6. **Check firewall settings**
7. **Try accessing via IP address directly**
8. **Check if the website is down** (downforeveryoneorjustme.com)

### 10. What does the command `ping` do and what information does it provide?

**Answer:**
**Ping** sends ICMP (Internet Control Message Protocol) echo requests to test network connectivity.

**Information provided:**
- **Reachability**: Whether the destination is accessible
- **Round-trip time (RTT)**: How long packets take to reach destination and return
- **Packet loss**: Percentage of packets that don't return
- **TTL (Time To Live)**: Number of hops before packet expires

**Example output:**
```
PING google.com (172.217.12.174): 56 bytes
64 bytes from 172.217.12.174: icmp_seq=1 ttl=117 time=15.2ms
64 bytes from 172.217.12.174: icmp_seq=2 ttl=117 time=14.8ms
```

### 11. What is the difference between `ping` and `traceroute`?

**Answer:**

**Ping:**
- Tests **end-to-end connectivity**
- Shows **round-trip time** to destination
- Uses **ICMP echo** requests
- **Simple pass/fail** connectivity test

**Traceroute:**
- Shows the **path packets take** to reach destination
- Displays **each hop** (router) along the path
- Shows **latency to each hop**
- Uses **TTL manipulation** or UDP/ICMP
- **Troubleshoots routing issues**

**Use cases:**
- **Ping**: "Can I reach the server?"
- **Traceroute**: "Where is the connection failing?"

### 12. What port numbers are commonly used for HTTP, HTTPS, SSH, and DNS?

**Answer:**

| Protocol | Port Number | Description |
|----------|-------------|-------------|
| **HTTP** | 80 | Web traffic (unencrypted) |
| **HTTPS** | 443 | Secure web traffic (SSL/TLS) |
| **SSH** | 22 | Secure shell remote access |
| **DNS** | 53 | Domain name resolution |

**Additional common ports:**
- **FTP**: 21 (control), 20 (data)
- **SMTP**: 25 (email sending)
- **POP3**: 110 (email retrieval)
- **IMAP**: 143 (email access)
- **Telnet**: 23 (insecure remote access)

---

## 🌐 Section 3: Network Security & Tools - Answers

### 13. What is a firewall and how does it work?

**Answer:**
A **firewall** is a network security device that monitors and controls incoming and outgoing network traffic based on predetermined security rules.

**How it works:**
- **Packet filtering**: Examines packets and applies rules
- **Stateful inspection**: Tracks connection states
- **Application awareness**: Understands application protocols
- **Rule-based**: Allows or blocks traffic based on configured rules

**Types of firewalls:**
- **Packet-filtering**: Basic rule-based filtering
- **Stateful**: Tracks connection state
- **Application-layer**: Deep packet inspection
- **Next-generation**: Advanced threat protection

### 14. What is NAT (Network Address Translation) and why is it used?

**Answer:**
**NAT** translates private IP addresses to public IP addresses, allowing multiple devices to share a single public IP.

**Why NAT is used:**
- **IPv4 address conservation**: Reduces need for public IPs
- **Security**: Hides internal network structure
- **Cost reduction**: Reduces ISP charges for IP addresses
- **Network management**: Simplifies internal addressing

**Types of NAT:**
- **Static NAT**: One-to-one mapping
- **Dynamic NAT**: Pool of public IPs
- **PAT (Port Address Translation)**: Many-to-one with port mapping

### 15. What is the difference between HTTP and HTTPS?

**Answer:**

**HTTP (HyperText Transfer Protocol):**
- **Port 80**
- **Unencrypted** communication
- **Vulnerable** to eavesdropping and tampering
- **Faster** (no encryption overhead)

**HTTPS (HTTP Secure):**
- **Port 443**
- **Encrypted** with SSL/TLS
- **Secure** against eavesdropping and tampering
- **Authentication**: Verifies server identity
- **Data integrity**: Prevents tampering
- **Slightly slower** due to encryption

**Key benefits of HTTPS:**
- **Confidentiality**: Data is encrypted
- **Integrity**: Data cannot be modified
- **Authentication**: Server identity is verified

### 16. What command would you use to check which ports are open on your local machine?

**Answer:**

**Linux/macOS:**
```bash
# Show all listening ports
netstat -tlnp
ss -tlnp

# Show specific port
netstat -tlnp | grep :80
ss -tlnp | grep :80

# Show all network connections
netstat -an
```

**Windows:**
```cmd
# Show all listening ports
netstat -an | findstr LISTENING

# Show processes using ports
netstat -ano

# Show specific port
netstat -an | findstr :80
```

**Modern alternatives:**
```bash
# Using lsof (Linux/macOS)
lsof -i :80
lsof -i TCP

# Using nmap (scan from outside)
nmap localhost
nmap -p 1-1000 localhost
```

---

## 🎯 Bonus Questions - Answers

### 17. What is ARP (Address Resolution Protocol) and when is it used?

**Answer:**
**ARP** resolves IP addresses to MAC (physical) addresses on a local network segment.

**When ARP is used:**
- When a device needs to send data to another device on the same subnet
- The sender knows the destination IP but needs the MAC address
- Communication within the same broadcast domain

**ARP process:**
1. **ARP request**: "Who has IP 192.168.1.100?"
2. **ARP reply**: "192.168.1.100 is at MAC 00:11:22:33:44:55"
3. **ARP table update**: Stores mapping for future use

**ARP commands:**
```bash
arp -a          # View ARP table
arp -d IP       # Delete ARP entry
```

### 18. Explain what happens when you type www.google.com in your browser and press Enter.

**Answer:** This is a complex process involving multiple layers:

1. **DNS Resolution:**
   - Browser checks local DNS cache
   - Queries DNS servers to resolve www.google.com to IP address

2. **TCP Connection:**
   - Browser initiates TCP handshake with Google's server
   - Three-way handshake (SYN, SYN-ACK, ACK)

3. **HTTP/HTTPS Request:**
   - Browser sends HTTP GET request
   - If HTTPS, SSL/TLS handshake occurs first

4. **Server Processing:**
   - Google's server processes the request
   - Generates HTML response

5. **Response Transfer:**
   - Server sends HTML, CSS, JavaScript files
   - Browser receives and processes content

6. **Rendering:**
   - Browser parses HTML and builds DOM
   - Applies CSS styles
   - Executes JavaScript
   - Renders the final page

**Network layers involved:**
- **Application**: HTTP/HTTPS
- **Transport**: TCP
- **Network**: IP routing
- **Data Link**: Ethernet frames
- **Physical**: Cable/wireless transmission

---

## 📚 Additional Study Resources

- **Books**: "Computer Networking: A Top-Down Approach" by Kurose & Ross
- **Online**: Cisco Networking Academy, Professor Messer's Network+ Training
- **Practice**: Set up a home lab with VirtualBox/VMware
- **Certification paths**: CompTIA Network+, CCNA
