# Public Offering 

**Category:** Network Forensics  
**Challenge:** Public Offering  
**Flag:** `deadface{167.172.132.224_4815}`

## 1. Challenge Overview

Kali sent a ZIP archive outside the corporate network. Our task is to analyze the network capture and identify the **public IPv4 address** and **TCP destination port** that received the archive.

## 2. Initial Triage

The provided PCAP contains approximately 7.8 MB of data and 53,326 packets.

Since `tshark` was unavailable in the sandbox, I used Python with the `dpkt` library to parse the capture and inspect the network traffic.

## 3. Identify the Outbound Traffic

I aggregated TCP payload bytes by source IP, destination IP, and destination port to identify unusually large transfers.

Most of the traffic was internal, originating from the `10.77.10.0/24` network. The main protocols included SSH (22), SMB (445), and several high-port connections.

Only a few connections were directed outside the corporate network:

| Source IP | Destination IP | Destination Port | Approx. TCP Payload |
|---|---|---:|---:|
| 10.77.10.66 | **167.172.132.224** | **4815** | **710 KB** |
| 10.77.10.20 | 20.42.65.89 | 443 | 8 KB |
| 10.77.10.20 | 4.150.223.97 | 443 | 3 KB |
| 10.77.10.20 | 104.208.203.88 | 443 | 0.1 KB |

The three HTTPS connections transferred relatively little data and appeared consistent with ordinary background traffic.

However, the connection from `10.77.10.66` to `167.172.132.224` on TCP port `4815` transferred approximately 710 KB, making it the primary candidate for the ZIP archive exfiltration.

## 4. Confirm the ZIP Archive

To verify the transfer, I scanned TCP payloads for the ZIP local file header signature:

```text
PK\x03\x04
```

A matching signature appeared in packet `18242`.

The packet contained the following connection details and payload:

```yaml
Source:      10.77.10.66:34174
Destination: 167.172.132.224:4815

Payload:
50 4b 03 04 14 00 00 00 08 00 ... annual_report.pdf
```

The `PK\x03\x04` signature confirms the beginning of a ZIP local file header. The first archive entry is named:

```text
annual_report.pdf
```

Subsequent packets in the same TCP stream contain compressed, high-entropy data, consistent with the remaining contents of the archive.

I also encountered other `PK` signatures in the capture, but those belonged to internal SMB traffic on port `445`, including `.rels` XML data from an Office document. They were unrelated to the outbound transfer.

## 5. Final Answer

The ZIP archive was transmitted from the internal host `10.77.10.66` to the following external destination:

- **Public IPv4:** `167.172.132.224`
- **Destination Port:** `4815/TCP`
- **Source Host:** `10.77.10.66`

**Flag:**

```text
deadface{167.172.132.224_4815}
```

## 6. Conclusion

By aggregating TCP payload sizes, identifying the unusually large outbound connection, and verifying the ZIP file signature in the corresponding TCP stream, I was able to identify the destination that received the archive.

One limitation is that I did not independently verify whether `10.77.10.66` was the Kali machine using SSH banners, DHCP information, or hostname traffic. Its identity is inferred from its role as the source of the outbound ZIP transfer.

This does not affect the flag, as the destination IPv4 address and TCP port are directly confirmed by the network capture.
