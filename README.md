# Real-World Network Behavior Profiling with Wireshark

Packet-level analysis of real network traffic across five capture sessions, comparing **HTTP vs HTTPS**, **audio streaming vs file download**, and **Wi-Fi vs mobile hotspot**. The analysis covers the DNS, TCP, TLS and QUIC layers.

Course project for **CENG 336 – Computer Networks**, Ankara Bilim University. Individual project.

📄 **[Read the full report (PDF)](report/Wireshark_Network_Analysis_Report.pdf)**

## Experiments

| # | Experiment | Captures | Hypothesis tested |
|---|---|---|---|
| 1 | HTTPS vs HTTP browsing | `capture1_https`, `capture2_http` | HTTPS hides application data; HTTP exposes requests, headers and HTML in plain text |
| 2 | Audio streaming vs file download | `capture3_spotify`, `capture4_download` | Streaming uses steady QUIC/UDP; a download produces a high-volume TCP stream |
| 3 | Wi-Fi vs mobile hotspot | `capture3_spotify`, `capture5_hotspot_spotify` | Cellular hotspot shows different latency and retransmission behavior |

Captured with Wireshark 4.6.5 on macOS (en0 Wi-Fi interface), about 3 minutes per session, with a timed action log for each session.

## Key findings

**DNS.** 182 DNS queries out of 28,007 packets during HTTPS browsing. A large share came from background services and trackers (iCloud Private Relay, DoubleClick, Microsoft telemetry, Last.fm) rather than the sites actually visited.

| Domain | DNS response time |
|---|---|
| mask.icloud.com (cached) | 8.5 ms |
| googleleads.g.doubleclick.net | 32.1 ms |
| www.wikipedia.org (CNAME chain) | 54.8 ms |

**TLS.** 132 Client Hello packets, all on port 443. The client offered 21 cipher suites, from TLS 1.3 down to legacy 3DES for compatibility. The **SNI field exposes the visited hostname in plain text** even over HTTPS. Both TLS 1.2 and TLS 1.3 connections were observed.

**QUIC.** Spotify streamed over QUIC v1 on UDP 443. QUIC combines the transport and TLS handshakes into one round trip and avoids TCP head-of-line blocking.

| Feature | TLS over TCP | QUIC |
|---|---|---|
| Transport | TCP | UDP |
| Handshake | TCP + TLS (2 round trips) | Combined (1 round trip) |
| Packet loss | TCP retransmission | Handled inside QUIC |
| Header encryption | Partial | Most fields encrypted |

**TCP retransmissions.**

| Capture | Packets | Retransmissions | Rate |
|---|---|---|---|
| HTTPS browsing (Wi-Fi) | 28,007 | 473 | 1.70% |
| 100 MB file download (Wi-Fi) | 95,974 | 454 | 0.47% |
| Spotify (mobile hotspot) | 10,501 | 53 | 0.50% |

Contrary to the hypothesis, the sustained download had a *lower* retransmission rate than ordinary browsing, which opens many short parallel connections.

## Display filters for each analysis

```
dns && dns.flags.response == 0        # DNS queries only (182 found)
tls.handshake.type == 1               # TLS Client Hello (132 found)
quic                                  # QUIC traffic (70 packets in the Spotify capture)
tcp.flags.syn == 1 && tcp.flags.ack == 0   # TCP connection openings (138 SYNs)
tcp.analysis.retransmission           # TCP retransmissions
http                                  # Plain-text HTTP exposure
```

## About the capture files

The raw `.pcapng` captures (about 170 MB in total) are not included. They contain real personal browsing traffic, and the largest file is over GitHub's 100 MB limit. All figures above come from those captures and are documented in the report.

## Tools

Wireshark · macOS · TCP/IP · DNS · TLS 1.2/1.3 · QUIC · HTTP
