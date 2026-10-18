# Name to a Face

**Category:** Network Forensics  
**Challenge:** Name to a Face  
**Flag:** `deadface{nkessler}`

## Overview

In the previous challenge, we identified a rogue host responding to poisoned LLMNR and mDNS queries. This challenge continues the investigation by examining what happened after Nathan's workstation interacted with that host.

The workstation attempted to authenticate with the rogue server over SMB using NTLM. By analyzing the NTLMSSP authentication exchange, we can recover the username used during authentication.

## Investigation

### 1. Locate the SMB Session

My initial investigation focused on IPv4 traffic between:

- **Victim:** `10.77.10.20`
- **Rogue Host:** `10.77.10.66`
- **Service:** SMB (TCP/445)

However, the IPv4 connections contained only SYN/RST packets and no meaningful application payload.

Looking back at the poisoned LLMNR and mDNS responses from the previous challenge, I noticed that the rogue host had also advertised its IPv6 link-local address:

`fe80::5054:ff:fe77:6601`

This revealed that the workstation was communicating with the rogue host over IPv6.

The relevant IPv6 communication was:

| Field | Value |
|---|---|
| Source | `fe80::4632:93f:50be:fef3` |
| Destination | `fe80::5054:ff:fe77:6601` |
| Protocol | TCP |
| Destination Port | `445` |
| Application Protocol | SMB2 |

This connection contained approximately 226 packets of SMB2 traffic.

### 2. Follow the NTLM Authentication Handshake

After locating the correct SMB session, I examined the NTLMSSP authentication exchanges.

Each connection attempt, starting from source port `50692`, followed the standard three-message NTLM handshake:

1. **NTLMSSP Type 1 – Negotiate:** Sent by the victim workstation to initiate authentication.
2. **NTLMSSP Type 2 – Challenge:** Sent by the rogue server with a challenge value.
3. **NTLMSSP Type 3 – Authenticate:** Sent by the victim workstation containing the authentication response and username.

The third message was the key to solving this challenge.

### 3. Extract the Username from the Authenticate Message

The NTLMSSP Type 3 Authenticate message contains security buffers describing the locations and lengths of several fields:

- LM Response
- NT Response
- Domain Name
- User Name
- Workstation Name

These fields can be extracted using their corresponding offsets and lengths. The string fields are encoded in UTF-16LE.

After decoding the Type 3 messages, I recovered the following information:

| Field | Extracted Value |
|---|---|
| Username | `nkessler` |
| Workstation | `DMF-WS11-042` |
| Domain (early attempts) | `.` |
| Domain (last attempt) | `dmf-field-01` |

The final authentication attempt appeared in **frame 2325**, where the domain field changed to `dmf-field-01`.

The username remained `nkessler` across the repeated authentication attempts, confirming the result.

## Answer

The username used by Nathan's workstation during NTLM authentication was:

**`nkessler`**

### Flag

```text
deadface{nkessler}
```

## Security Takeaways

This challenge demonstrates the risks associated with LLMNR and mDNS poisoning.

An attacker controlling a rogue host can respond to name-resolution requests and trick a workstation into connecting to an unauthorized SMB server.

During NTLM authentication, the attacker may capture an NTLMv2 challenge-response that could potentially be cracked offline or relayed to another service under suitable conditions.

Recommended mitigations:

- Disable LLMNR and NBT-NS where they are not required.
- Restrict or disable mDNS where appropriate.
- Enable SMB signing to help prevent NTLM relay attacks.
- Enforce strong authentication policies and consider reducing NTLM usage.
- Monitor unusual SMB connections and unexpected NTLM authentication attempts.
- Rotate credentials if they may have been exposed.

**Key takeaway:** A poisoned name-resolution response can lead to an SMB authentication attempt, potentially exposing valuable authentication material.

---

*Challenge completed as part of DEADFACE CTF.*
