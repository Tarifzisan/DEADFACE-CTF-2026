# Don't Leave Me Hanging

**Event:** DEADFACE CTF 2026  
**Category:** Network Forensics  
**Challenge:** Don't Leave Me Hanging  
**Points:** 500  
**Flag:** `deadface{Pr355_0n3_f0r_D34df4c3}`

---

## Overview

The challenge provided a PCAP containing VoIP traffic from Aurora Health's help desk subnet (`10.20.200.0/24`).

The objective was to investigate the captured network traffic, identify leaked credentials and cryptographic materials, decrypt the encrypted SIP and SRTP communications, and recover the hidden flag from a phone call.

The investigation revealed the following attack chain:

1. A plaintext FTP session exposed a PBX diagnostic bundle.
2. The diagnostic bundle contained a TLS key log.
3. The TLS key log enabled decryption of SIP signalling.
4. SIP/SDP messages contained the SRTP master keys.
5. The SRTP streams were decrypted to recover the original audio.
6. One call contained DTMF tones encoding the final flag.

## 1. PCAP Triage

The capture contained approximately 256,000 packets. Most traffic consisted of background network activity, including:

- YouTube and Google TLS traffic
- Windows telemetry
- mDNS and SSDP
- NTP and DHCP

Several interesting communication patterns stood out:

| Traffic | Observation |
|---|---|
| UDP RTP (G.711 µ-law, PT 0) | VoIP audio between the PBX (`10.20.200.5`) and extensions `.21`–`.24` |
| TCP 5061 | Encrypted SIP signalling (SIPS) |
| TCP 21 | Plaintext FTP communication |
| TCP 40040/40079 | FTP passive-mode data transfers |
| HTTP from `.23` | Connections to DEADFACE-themed websites |

The RTP packets were 182 bytes long, consisting of:

- 12-byte RTP header
- 160-byte encrypted audio payload
- 10-byte SRTP authentication tag

The encrypted payload had approximately 8 bits per byte of entropy, suggesting that the audio was protected by SRTP.

Without the appropriate cryptographic keys, the captured audio could not be played directly.

## 2. Plaintext FTP Credential and Key Leak

The FTP control channel revealed an IT laptop (`AHF-IT-LT60`, `10.20.200.60`) connecting to the PBX server (`10.20.200.5`) running vsFTPd.

The FTP session revealed the following login attempts:

- Username: `mkowalski`
- First password: `Aur0ra!Helpdesk25` — Failed
- Second password: `Aur0ra!Helpdesk26` — Successful

After authentication, the user accessed:

```text
CWD /var/log/asterisk/diag
RETR pbx-diag-20261008.tar.gz
```

The FTP passive-mode data stream (`10.20.200.5:40079`) contained the diagnostic archive.

After extracting the archive, the following files were discovered:

```text
pbx-diag-20261008/
├── README.txt
├── messages.log
├── pjsip_endpoints.txt
├── tls-keys.log
└── tls-keys.log.1
```

The most important file was:

```text
tls-keys.log
```

It contained TLS session secrets associated with the PBX.

The README explained that the diagnostic bundle had been created to investigate an Asterisk one-way-audio incident (`INC0041877`). A vendor had requested decrypted SIP traces for troubleshooting.

Two additional findings were particularly important:

**1. SRTP configuration**

The `pjsip_endpoints.txt` file showed:

```text
media_encryption: sdes
```

This indicated that SRTP keys were transported through SIP SDP messages.

**2. DTMF configuration**

Extension `2003` (`10.20.200.23`) was the only endpoint configured with:

```text
dtmf_mode: inband
```

This suggested that DTMF tones from this endpoint would be transmitted directly within the audio stream.

## 3. Decrypting TLS and SIP Signalling

The next step was to decrypt the encrypted SIP communication using the recovered TLS key log.

Each TLS ClientHello random value from the PCAP was matched with its corresponding `CLIENT_RANDOM` entry in the key log.

All five sessions used:

```text
TLS 1.3
TLS_AES_256_GCM_SHA384
```

The decryption process involved:

1. Extracting the appropriate traffic secrets (`*_TRAFFIC_SECRET_0`).
2. Deriving encryption keys and IVs using HKDF-Expand-Label with SHA-384.
3. Constructing AES-GCM nonces by XORing the IV with the record sequence number.
4. Decrypting TLS application-data records using AES-GCM.
5. Using the five-byte TLS record header as additional authenticated data.

After successful decryption, readable SIP messages were recovered, including:

- REGISTER
- INVITE
- 200 OK
- SDP session descriptions

The SDP messages contained entries similar to:

```text
a=crypto:1 AES_CM_128_HMAC_SHA1_80 inline:<base64>
```

The Base64-encoded inline value contained the SRTP master key and salt:

- Master key: 16 bytes
- Master salt: 14 bytes

### Important Parsing Issue

During the initial implementation, some short Ethernet frames caused problems.

Ethernet padding was mistakenly interpreted as TCP payload, resulting in incorrect parsing of the `.21` and `.22` communication streams.

The issue was resolved by trimming the captured payload according to the IP total-length field.

## 4. Decrypting SRTP Audio Streams

After extracting the SRTP master keys and salts from the decrypted SDP messages, the next objective was to recover the original audio.

The SRTP decryption process involved the following steps:

### Step 1: Derive Session Keys

Using the SRTP master key and master salt, three session keys were derived using AES-CM:

| Label | Purpose |
|---|---|
| 0 | Session encryption key |
| 1 | Session authentication key |
| 2 | Session salt |

### Step 2: Verify Authentication

Each SDP key was tested against the first RTP packet of the corresponding stream.

The correct key was identified by verifying its HMAC-SHA1-80 authentication tag.

### Step 3: Decrypt RTP Payload

The RTP payload was decrypted using AES-CTR.

The initialization vector was constructed using:

- Session salt
- SSRC
- Packet index (rollover counter and sequence number)

### Step 4: Recover Audio

After successful decryption, the G.711 µ-law audio payloads were decoded and converted into WAV files for analysis.

**Result:** All 56 RTP streams were successfully authenticated and decrypted.

## 5. Identifying the Suspicious Phone Call

With the audio recovered, the next step was to identify unusual calls.

A spectrogram analysis was performed on calls associated with extension `2003`.

Most calls contained ordinary voice conversations. However, one call was noticeably different.

The suspicious call had the following characteristics:

| Attribute | Value |
|---|---|
| Caller | MARGARET OKAFOR |
| Phone Number | +17168859009 |
| Media Port | 14740 |
| Audio Pattern | Repeated two-tone bursts |
| Agent Side | Silent |

The spectrogram showed a regular pattern of two-tone signals rather than normal speech.

This was consistent with DTMF (Dual-Tone Multi-Frequency) signalling.

### DTMF Frequency Analysis

A Goertzel detector was applied to identify the DTMF frequencies.

The standard frequency groups were:

| Low Frequency (Hz) | High Frequency (Hz) |
|---|---|
| 697 | 1209 |
| 697 | 1336 |
| 697 | 1477 |
| 697 | 1633 |
| 770 | 1209 |
| 770 | 1336 |
| 770 | 1477 |
| 770 | 1633 |
| 852 | 1209 |
| 852 | 1336 |
| 852 | 1477 |
| 852 | 1633 |
| 941 | 1209 |
| 941 | 1336 |
| 941 | 1477 |
| 941 | 1633 |

The detector successfully extracted the following digit stream:

```text
100101097100102097099101123080114051053053095048110051095102048114095068051052100102052099051125
```

The extracted digits represented three-digit decimal ASCII codes.

### ASCII Decoding

The digit stream was divided into groups of three:

```text
100 101 097 100 102 097 099 101 123 ...
```

Converting the decimal ASCII values into characters produced:

```text
deadface{Pr355_0n3_f0r_D34df4c3}
```

## 6. Dead Ends and False Leads

Several suspicious findings initially appeared relevant but did not contribute to the final solution.

### Sunstalker

The string `Sunstalker` appeared in the phone's User-Agent strings:

```text
SunstalkerDFBrower/2.0
Sunstalker S14
```

Initially, this appeared to be a potential clue. However, it was simply the device model and not the flag.

### DEADFACE-Themed Websites

The following websites and related content were investigated:

- `ghosttown.deadface.io`
- `maudeai.deadface`

Their PNG files, HTML content, and comments such as `MaudeAI: Saving` contained no relevant hidden information.

Other searches involving Blondie and the phrase "hiding things in plain sight" were also unproductive.

### Large HTTPS Flow

A large 6 MB communication flow from `10.20.200.22` to `185.110.215.68:443` initially looked like possible data exfiltration.

Further investigation revealed that it was simply streaming media.

### LSB Steganography

LSB steganography analysis of the recovered audio did not reveal anything useful.

### RFC 4733 DTMF Events

The RTP telephone-event packets associated with extension `2001` were also investigated.

They decoded to:

```text
40917#
```

However, this was not the required flag.

## 7. Final Flag

<details>
<summary><strong>Click to reveal the flag</strong></summary>

```text
deadface{Pr355_0n3_f0r_D34df4c3}
```

</details>

## 8. Key Takeaways

This challenge demonstrated how multiple security weaknesses can combine to expose sensitive communications.

Important lessons:

- Never transfer diagnostic archives containing sensitive information over plaintext FTP.
- TLS key logs must be handled securely and removed after troubleshooting.
- Diagnostic bundles should be reviewed before being shared with external vendors.
- SDES transports SRTP keying material inside SIP signalling, making the protection of the signalling channel critical.
- A compromised TLS key log can expose both SIP signalling and the SRTP keys exchanged through SDP.
- Encrypted network traffic can still contain recoverable information when key management practices are weak.

## Conclusion

The challenge required combining network forensics, FTP analysis, TLS decryption, SIP/SDP investigation, SRTP cryptography, audio processing, and DTMF decoding.

The most important discovery was the TLS key log hidden inside the PBX diagnostic archive. Recovering that file made it possible to decrypt the SIP signalling, extract the SRTP keys, and reconstruct all captured calls.

Ultimately, the hidden flag was recovered from a DTMF-encoded phone call.

**Flag:** `deadface{Pr355_0n3_f0r_D34df4c3}`

---

*Write-up by Tarif Bin Belal (Zisan) | DEADFACE CTF 2026*
