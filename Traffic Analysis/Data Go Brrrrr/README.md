# Data Go Brrrrr — Network Forensics (DEADFACE CTF)

**Category:** Network Forensics  
**Challenge:** Data Go Brrrrr  
**Flag:** `deadface{icmp_3xfil_wh3n_th3y_least_susp3ct}`

## Challenge Description

The client noticed suspicious outbound data transfers from their network and narrowed the investigation down to a small packet capture file named `PCAP01.pcap`.

The objective was to analyze the network traffic, identify the exfiltration method, and recover the hidden data.

## Investigation

### 1. Initial Triage

The provided PCAP file was only **2,946 bytes** and contained **34 packets**.

Since neither `tshark` nor Scapy was installed in my environment, I decided to inspect the PCAP manually using Python's built-in `struct` module.

I wrote a small Python script to parse the packet headers, identify protocols, and extract packet payloads.

This allowed me to inspect the traffic without installing additional packet-analysis tools.

### 2. Identifying Suspicious Traffic

After examining the packets, I identified three different types of traffic:

| Packets | Traffic | Analysis |
|---|---|---|
| 1–8, 29–34 | ICMP echo requests/replies to `192.168.1.1` | Normal ping traffic with standard incrementing payload bytes |
| 9–10 | DNS query and response for `www.example.com` | Normal DNS activity |
| 11–12 | HTTP GET and 200 OK to `93.184.216.34` | Normal HTTP activity |
| 13–28 | ICMP echo requests/replies involving `10.10.14.7` | Suspicious activity |

The most interesting traffic was found in packets **13–28**.

### 3. Investigating the ICMP Traffic

Several unusual characteristics caught my attention:

- The internal host `192.168.1.50` was sending ICMP echo requests to `10.10.14.7`, rather than its usual gateway.
- Unlike normal ping traffic, the ICMP payloads contained short ASCII strings.
- The payloads appeared to be Base64 encoded. For example, `ZGVhZGZh` decoded to `deadfa`.
- The corresponding ICMP echo replies contained only `OK`, suggesting that the remote host was acknowledging the received data chunks.

These observations indicated a possible **ICMP-based data exfiltration technique**.

Instead of using a conventional network protocol, the sender was hiding information inside ICMP echo request payloads.

### 4. Extracting and Reassembling the Data

I extracted the eight suspicious ICMP request payloads in their original packet order:

```text
ZGVhZGZh
Y2V7aWNt
cF8zeGZp
bF93aDNu
X3RoM3lf
bGVhc3Rf
c3VzcDNj
dH0=
```

Each chunk contained eight Base64 characters, allowing them to be concatenated before decoding.

I used Python's built-in `base64` module to reconstruct the original message.

**Python Script:**

```python
import base64

chunks = [
    'ZGVhZGZh',
    'Y2V7aWNt',
    'cF8zeGZp',
    'bF93aDNu',
    'X3RoM3lf',
    'bGVhc3Rf',
    'c3VzcDNj',
    'dH0='
]

encoded_data = ''.join(chunks)
decoded_data = base64.b64decode(encoded_data)

print(decoded_data.decode())
```

**Output:**

```text
deadface{icmp_3xfil_wh3n_th3y_least_susp3ct}
```

The hidden data was successfully recovered!

## Flag

```text
deadface{icmp_3xfil_wh3n_th3y_least_susp3ct}
```

## Key Takeaways

**Detection**
- Monitor ICMP echo requests with unusual or non-standard payloads.
- Investigate ICMP traffic directed toward unexpected external destinations.
- Look for repeated ICMP packets carrying encoded or structured data.

**Indicators of Compromise**
- Unexpected outbound ICMP traffic from internal hosts.
- Base64-like strings inside ICMP payloads.
- Repeated echo requests followed by short acknowledgment responses.

**Mitigation**
- Apply appropriate outbound ICMP restrictions at network boundaries.
- Monitor ICMP traffic using IDS/IPS solutions.
- Use Deep Packet Inspection (DPI) to identify suspicious payload patterns.
- Establish baseline network behavior to detect unusual ICMP communication.

## Conclusion

This challenge demonstrated how attackers can abuse seemingly harmless ICMP echo requests to exfiltrate information.

By manually parsing the PCAP, identifying unusual ICMP payloads, and reconstructing the Base64-encoded chunks, I was able to recover the hidden flag.

**Lesson learned:** Not all ping traffic is innocent. Sometimes, the data is hiding in plain sight.

---
