# Crown Jewels

**Category:** Syyntax  
**Points:** 75  
**Challenge:** Crown Jewels  
**Flag:** `deadface{annual_report.pdf}`

## Challenge Description

**Question:** Which PDF was read first from the executive share?

## 1. Initial Triage

The provided `demonne.pcap` capture was approximately 7.8 MB. Since `tshark` was unavailable in my environment, I used Python with Scapy to inspect and analyze the network traffic.

The initial traffic breakdown revealed:

- **TCP/22 (SSH):** Approximately 50,000 packets, accounting for most of the capture.
- **TCP/445 (SMB):** 575 packets.
- **TCP/4815:** 466 packets.
- **HTTP/WebDAV (port 80):** A small amount of traffic.
- **Other protocols:** ARP, mDNS, and LLMNR.

The SMB traffic appeared particularly interesting because the challenge involved identifying files accessed from a network share.

## 2. Identifying the Executive Share

A search through the printable strings in the capture revealed the following network path:

```text
\\10.77.10.30\executive
```

This identified the executive SMB share hosted at `10.77.10.30`.

An SMB2 directory listing in **packet 6898** revealed the following files:

```text
annual_report.pdf
executive_risk_register.xlsx
board_transition_timeline.pdf
succession_vote_draft.pdf
candidate_background_notes.docx
```

Now that the available files were identified, the next step was determining their access order.

## 3. Investigating Potential File Access Evidence

Before analyzing the SMB traffic further, I examined two other potentially relevant network activities.

### WebDAV — Port 80

The HTTP/WebDAV traffic contained:

- `OPTIONS`
- `PROPFIND /executive`

However, there were no corresponding `GET` requests.

This indicated directory enumeration rather than actual file retrieval, so the WebDAV traffic did not establish which PDF was read first.

### ZIP Transfer — Port 4815

Further investigation revealed that four of the listed files, excluding the `.docx` document, appeared in a ZIP archive transferred over TCP port `4815`.

This looked like the likely exfiltration activity.

However, the ZIP central directory listed:

1. `board_transition_timeline.pdf`
2. `executive_risk_register.xlsx`

This ordering did not match the order in which the files were opened from the SMB share.

Therefore, the ZIP archive's internal file ordering could not reliably establish the original access sequence. It reflected archive construction order, not necessarily file read order.

## 4. Determining the SMB File Open Order

Most of the SMB session was encrypted, making it difficult to inspect the actual file read operations.

However, SMB2 CREATE requests still exposed the filenames in plaintext.

My initial approach was to search for SMB2 headers directly inside TCP payloads, but this did not produce the expected results, possibly because the messages were wrapped or segmented.

I then changed my approach and searched the complete raw packet bytes for the filenames encoded in UTF-16LE.

Filtering for client-to-server traffic from port `52682` to port `445` revealed the following sequence:

| Packet Number | Filename |
|---|---|
| 7403 | `annual_report.pdf` |
| 8320 | `executive_risk_register.xlsx` |
| 9085 | `board_transition_timeline.pdf` |
| 9737 | `succession_vote_draft.pdf` |

The packet numbers established the chronological order of the SMB file-open requests.

Additionally, each CREATE request was followed by hundreds of packets of data transfer before the next file was opened.

The first file opened was therefore:

```text
annual_report.pdf
```

## 5. Conclusion

By correlating the SMB CREATE requests with their packet numbers, I reconstructed the chronological sequence in which the files were opened from the executive share.

Although the SMB read commands themselves were encrypted, the plaintext filenames in the CREATE requests provided sufficient evidence to identify the first opened PDF.

**Final Flag:**

```text
deadface{annual_report.pdf}
```

## 6. Analysis Notes and Limitations

- The file access sequence was reconstructed using SMB2 CREATE (open) requests, not the actual SMB READ commands.
- The SMB session was not decrypted, so the precise read operations could not be independently verified.
- The packet-number gaps and subsequent data-transfer activity support the inferred opening sequence.
- ZIP archive ordering was treated separately from SMB access ordering to avoid drawing an incorrect conclusion from the exfiltrated archive.

This investigation demonstrates how plaintext SMB metadata can still provide useful forensic evidence even when the underlying SMB session is encrypted.
