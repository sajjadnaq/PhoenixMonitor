# PhoenixMonitor v0,1
Real-time process network monitor and threat analyzer for Windows
<div align="center">

![Platform](https://img.shields.io/badge/platform-Windows%2010%2B-blue?style=flat-square)
![.NET](https://img.shields.io/badge/.NET-8.0-purple?style=flat-square)
![Framework](https://img.shields.io/badge/UI-WPF-blueviolet?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)
![Admin](https://img.shields.io/badge/requires-Administrator-orange?style=flat-square)

Phoenix Monitor lets you see exactly what every running process is doing on your network — live, per-process, with deep packet inspection, TLS fingerprinting, and Authenticode trust verification — all in a modern dark-themed desktop UI.

</div>
 Features

 Process & Connection Intelligence
- Live sidebar listing of all running processes with owner, PID, and path
- Per-process TCP/UDP connection table (local ↔ remote endpoints, state, protocol)
- Search and filter processes in real time; toggle system process visibility
- 3-second background refresh that **never interrupts** an active capture session

 Live Packet Capture
- Automatic raw-socket capture starts the moment you click a process — no manual setup
- One capture socket per local interface, matching only that process's connections
- Packets accumulate across refreshes; explicit **Clear** button to reset
- Auto-scrolling packet stream with elapsed time, direction (→ / ←), length, and summary
- Up to 2 000 packets retained in the rolling live view

Deep Packet Dissection
- Click any packet to open a split dissection panel:
  - **Protocol tree** — IPv4 → TCP/UDP → TLS layers with every header field named and decoded
  - **Hex dump** — classic 16-byte-wide hex + ASCII view in monospace green
- Resizable splitter between the tree and hex dump

TLS / JA3 Fingerprinting
- Detects TLS ClientHello on **any port** (not just 443)
- Extracts and displays:
  - **SNI** (Server Name Indication) — the hostname the client is connecting to
  - **JA3 fingerprint** — MD5 hash of the normalized ClientHello fields, GREASE-filtered (RFC 8701)
  - **JA3 raw string** — for allow-list building and manual analysis
  - Offered **cipher suites**, **elliptic curves**, and **TLS extensions** — all named
- JA3 column in the live stream; full TLS layer in the dissection tree
- TLS version shown: SSL 2.0 through TLS 1.3

Process Signature Verification
Displayed immediately in the process header when you select a process:

| Badge | Meaning |
|-------|---------|
| ✓ **Trusted** | Valid Authenticode chain to a Windows-trusted root CA |
| ⚠ **Untrusted** | Signed but root not in the Windows trust store |
| ✗ **Tampered** | `TRUST_E_BAD_DIGEST` — binary modified after signing |
| ○ **Unsigned** | No Authenticode signature present |
| ◈ **Ad-hoc** | Self-signed certificate (issuer == subject) |

**Homoglyph / script-mixing detection** — the signer name is scanned for:
- Cyrillic + Latin character mixing (e.g. fake "Мicrosoft" with Cyrillic М)
- Greek + Latin mixing
- Hidden bidirectional / zero-width control characters

Any suspicious signer triggers a **red warning banner** in the UI.

Firewall Rule Engine
- Built-in rule editor: Block / Allow / Log rules by process, IP, port, or protocol
- Rules applied directly to **Windows Firewall** via the COM API (`HNetCfg.FwPolicy2`)
- Rules persist across sessions (JSON store in `%AppData%\PhoenixMonitor\`)
- Right-click any live connection → **Block this connection** or **Allow this connection**
- Rules panel toggled with the **🛡 Rules** toolbar button

 UI / UX
- Dark palette inspired by the macOS sibling app
- Fully **resizable panels**: sidebar width, connections / live-packets split, dissection panel
- Monospace packet display (Cascadia Code → Consolas fallback)

Getting Started

### Prerequisites
| Requirement | Version |
|---|---|
| .NET SDK | 8.0 |
| Privileges | **Administrator** (raw sockets + WFP rules require elevation) |




### Key design decisions

| Decision | Rationale |
|---|---|
| **Blocking `socket.Receive()` on a dedicated `Thread`** | `ReceiveAsync` with `SIO_RCVALL` is unreliable on Windows raw sockets; a blocking loop on a background thread is the only reliable approach |
| **`_suppressCaptureRestart` flag** | WPF's `SelectedItem` binding pushes `null` when `ItemsSource` is rebuilt; this flag prevents the 3-second refresh from tearing down an active capture |
| **One socket per local IP** | Avoids duplicate packets when a process has connections on multiple interfaces |
| **`Dispatcher.BeginInvoke(Background)` for `ScrollIntoView`** | Calling `ScrollIntoView` inside `CollectionChanged` races with WPF's `VirtualizingStackPanel`; deferring to the Background priority avoids the `ItemContainerGenerator` inconsistency exception |
| **`ConcurrentDictionary` cache in `SignatureService`** | Authenticode verification is I/O-heavy; caching by exe path keeps the UI snappy on repeated selections |

## 🔒 Security Notes

- Phoenix Monitor **reads** Windows Firewall rules and creates new ones via the standard COM API. It does **not** bypass or disable the firewall.
- Raw socket capture requires Administrator because `SIO_RCVALL` is a privileged socket I/O control.
- Firewall rules created by Phoenix Monitor are named `PM_{guid}_{IN|OUT}` and can be removed from Windows Defender Firewall at any time.
- Signature verification uses `WinVerifyTrust` — the same API used by Windows itself.

 Roadmap

- [x] Phase 1 — Process list + connection table
- [x] Phase 2 — Firewall rule engine (Block / Allow / Log)
- [x] Phase 3 — Live packet capture + protocol dissection
- [x] Phase 3b — TLS ClientHello / JA3 fingerprinting
- [x] Phase 3c — Authenticode trust + homoglyph detection
- [ ] Phase 4 — JA3 threat intelligence lookup (known malware hashes)
- [ ] Phase 4b — DNS query capture and hostname resolution overlay
- [ ] Phase 5 — Historical session recording and PCAP export


## 📄 License

Distributed under the MIT License. See `LICENSE` for more information.
<div align="center">


</div>
