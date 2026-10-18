# Uninvited Guest

**Event:** DEADFACE CTF  
**Category:** Network Forensics  
**Challenge:** Uninvited Guest  
**Flag:** `deadface{10.77.10.66}`

## Challenge Overview

The challenge involves investigating suspicious network activity to identify a rogue host impersonating a legitimate file server.

Nathan's workstation (`10.77.10.20`) repeatedly attempts to resolve a file-server hostname through local network name-resolution protocols. However, a suspicious host responds to these requests with its own IP address.

The objective is to identify the IPv4 address responsible for supplying these suspicious responses.

## Investigation

### 1. Analyzing the PCAP

The provided PCAP contained **53,326 packets**. Since Wireshark and Scapy were unavailable in the environment, I developed a small Python-based PCAP parser to inspect the traffic.

The parser handled:

- Ethernet frames
- IPv4 and IPv6 packets
- UDP traffic
- DNS-style name-resolution messages

This allowed me to isolate the relevant name-resolution requests and responses.

### 2. Identifying Name-Resolution Traffic

The workstation `10.77.10.20` repeatedly attempted to resolve the hostname `DMF-FIELD-01` using multiple local name-resolution protocols.

The investigation revealed the following:

| Protocol | Port | Queries | Suspicious Responder |
|---|---:|---:|---|
| mDNS | UDP 5353 | 40 | `10.77.10.66` |
| LLMNR | UDP 5355 | 40 | `10.77.10.66` |
| NBNS | UDP 137 | 1 | `10.77.10.66` |

The queried names included:

- `dmf-field-01.local` (mDNS)
- `dmf-field-01` (LLMNR)
- `DMF-FIELD-01<1c>` (NBNS)

All three protocols received responses pointing to the same suspicious IPv4 address.

### 3. Inspecting the Responses

Next, I examined the A-record resource data in the captured mDNS, LLMNR, and NBNS responses.

The relevant hexadecimal bytes were:

```text
0a 4d 0a 42
```

Converting these bytes into dotted-decimal IPv4 notation:

```text
0a = 10
4d = 77
0a = 10
42 = 66
```

Therefore:

```text
10.77.10.66
```

This confirmed that the suspicious host was responding to name-resolution requests with its own IP address instead of the expected file-server address.

### 4. Verifying the IPv6 Responses

To strengthen the finding, I also examined IPv6 LLMNR traffic.

LLMNR multicast requests were sent to:

```text
ff02::1:3
```

The suspicious responses originated from:

```text
fe80::5054:ff:fe77:6601
```

The corresponding MAC address was:

```text
52:54:00:77:66:01
```

This MAC address matched the host associated with `10.77.10.66`.

No other host was observed responding to these requests, further supporting the identification of a single suspicious responder.

### 5. Investigating Additional Network Activity

I also examined the traffic involving `10.77.10.66` to understand its broader network activity.

Notable observations included:

- Heavy TCP/22 communication with `10.77.10.20` and `10.77.10.254`.
- SMB (445) and RPC (135) connections involving `10.77.10.30`.
- SMB and LDAPS communication with `10.77.10.1`.
- A long-running external connection to `167.172.132.224:4815`.
- DNS lookups for the legitimate hostname `DMF-FILE-01` through `1.1.1.1`.

These observations suggest that `10.77.10.66` may be a compromised or attacker-controlled machine. However, the flow analysis alone does not establish the exact purpose of every connection.

## Attack Technique

The observed behavior is consistent with **LLMNR/NBT-NS/mDNS poisoning**.

When a workstation cannot resolve a hostname through its normal DNS configuration, it may fall back to local name-resolution mechanisms such as LLMNR or NBT-NS.

An attacker on the same network can respond to these requests with a forged address, potentially redirecting the victim to an attacker-controlled machine.

Tools such as **Responder** automate this technique and can capture authentication attempts or facilitate further attacks, depending on the network configuration and victim behavior.

## Flag

The suspicious host consistently supplied its own IPv4 address in response to the victim's name-resolution requests.

**Identified IPv4 Address:**

```text
10.77.10.66
```

**Final Flag:**

```text
deadface{10.77.10.66}
```

## Remediation

To reduce the risk of name-resolution poisoning attacks:

1. **Disable LLMNR:** Use Group Policy to disable LLMNR across the Windows environment.
2. **Disable NBT-NS:** Disable NetBIOS over TCP/IP where it is not required.
3. **Restrict mDNS:** Disable mDNS in environments where it is unnecessary.
4. **Enforce SMB Signing:** Require SMB signing to help prevent NTLM relay attacks.
5. **Investigate the Rogue Host:** Isolate and investigate `10.77.10.66`, including its communication with `167.172.132.224:4815`.
6. **Monitor Name Resolution:** Alert on unexpected LLMNR, NBT-NS, and mDNS responses from unauthorized hosts.

## Conclusion

This challenge demonstrated how network packet analysis can reveal suspicious name-resolution behavior even without relying on tools such as Wireshark.

By examining mDNS, LLMNR, and NBNS requests, decoding response records, and correlating IPv4, IPv6, and MAC address information, I identified `10.77.10.66` as the host supplying the suspicious responses.

**Key Takeaway:** Unnecessary local name-resolution protocols can create opportunities for attackers to impersonate legitimate network services.
