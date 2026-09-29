# TrustBound

**Continuous spatial & hardware authentication for Zero Trust architectures.**

> The physical perimeter is the new cryptographic boundary — we don't trust a device until we can mathematically prove *exactly where it is*.

🌐 Live site & interactive demo: **https://trustbound.forum** · ✉️ **trustbound67@gmail.com**
🇬🇧 English (this branch `en`) · 🇺🇦 Ukrainian: branch [`uk`](../../tree/uk)

---

## The problem

Identity & Access Management verifies you **once, at login** — then goes blind to the device's physical movement. With valid credentials, a live session and an intact TPM, an attacker can carry an unlocked laptop out of a secure zone and keep full access. Manual revocation takes hours to days; a single R&D leak costs millions.

## The idea

Access is not a door you open once — it's a **stream you must keep earning**. Every ~3–5 seconds the device must re-prove **three independent facts**:

| Layer | Technology | Proves |
|-------|-----------|--------|
| Hardware root of trust | **TPM 2.0** (PCR, TPM Quote, EK/AK) | Same, untampered device |
| Macro-location | **GNSS / Galileo** (+ OSNMA) | In the allowed building / geofence |
| Micro-positioning | **UWB** (IEEE 802.15.4z, ToF, STS) | Physically in this room, now |

## Architecture

```mermaid
%%{init: {'theme':'dark','themeVariables':{'fontSize':'16px','clusterBkg':'#0a1526','clusterBorder':'#33507a','lineColor':'#6b86ad','edgeLabelBackground':'#0a1526'}}}%%
flowchart LR
  subgraph DEV[" On the device"]
    direction TB
    TPM[" TPM 2.0<br/>device integrity"]:::dev
    GEO[" GNSS / Galileo<br/>location"]:::dev
    UWB[" UWB anchors<br/>signed in-room ranges"]:::dev
    AG[" TrustBound Agent<br/>collects &amp; signs evidence"]:::agent
    TPM --> AG
    GEO --> AG
    UWB --> AG
  end
  subgraph SRV[" TrustBound backend"]
    direction TB
    S[" Server<br/>verifies &amp; issues keys"]:::srv
    GW[" Gateway<br/>enforces the key"]:::srv
    R[" Protected<br/>resources"]:::res
    S -->|"K_epoch · valid ~3–5 s"| GW
    GW --> R
  end
  AG ==>|"① evidence, signed Ed25519"| S
  S -.->|"② nonce + random beacon quorum"| AG
  classDef dev fill:#12213c,stroke:#4f8cff,color:#eaf2ff;
  classDef agent fill:#173a63,stroke:#7fb0ff,color:#eaf2ff,stroke-width:2px;
  classDef srv fill:#0b332d,stroke:#35d0ba,color:#eaf2ff;
  classDef res fill:#3a2a10,stroke:#f4b740,color:#ffedc2;
```

## How verification works (every ~3–5 s)

```mermaid
%%{init: {'theme':'dark','themeVariables':{'actorBkg':'#12213c','actorBorder':'#4f8cff','actorTextColor':'#eaf2ff','signalColor':'#6b86ad','signalTextColor':'#cfe0f5','noteBkgColor':'#0e2038','noteTextColor':'#cfe0f5','noteBorderColor':'#33507a','labelBoxBkgColor':'#0b332d','labelBoxBorderColor':'#35d0ba','labelTextColor':'#eaf2ff'}}}%%
sequenceDiagram
  autonumber
  participant S as Server
  participant A as Agent · device
  Note over S,A: new epoch every ~3–5 seconds
  S->>A: nonce + challenge (random 3 of 5 beacons)
  A->>A: gather TPM Quote · GPS fix · signed UWB ranges
  A->>S: Ed25519-signed evidence bundle
  S->>S: verify TPM · location · UWB quorum · nonce
  alt  all checks pass
    S-->>A: derive K_epoch (HKDF) → access granted ~3–5 s
  else  any check fails — left the room, spoof, tamper
    S-->>A: DENY → no key · data stays encrypted (AES-256-GCM)
  end
```

Leave the room → the beacons can't confirm presence → the next key `K_epoch` is never derived → access dies in seconds and resources stay encrypted (AES-256-GCM).

## Two core innovations

- **Random Witness Quorum** — the server randomly picks a subset of UWB beacons each round (e.g. 3 of 5). You can't pre-forge what you can't predict; to fool it you'd need every beacon at once.
- **Presence-Bound Key Stream** — short-lived keys derived via **HKDF-SHA256** *only* when TPM + GEO + UWB quorum + a fresh server challenge all pass. No proof → no key.

## Technology

- **Crypto:** Ed25519 (signatures), HKDF-SHA256 + HMAC-SHA256 (key stream), AES-256-GCM (sealing), TLS 1.3 mTLS; PQC-ready (ML-KEM / ML-DSA).
- **Hardware:** TPM 2.0, GNSS/Galileo receiver, UWB radio (IEEE 802.15.4z / FiRa), beacons with Secure Element keys.
- **Software:** client agent + central server — Python (FastAPI), hot paths in Go/Rust, C/C++ for TPM (TSS/tpm2-tools) and UWB; deployed in **Docker**; tamper-evident audit (hash chain).

## Attack resistance

| Attack | Why it fails |
|--------|--------------|
| 🚪 Device theft | Out of room → no UWB quorum → no key |
| 🛰 GPS spoofing | UWB needs signed in-room replies; Galileo OSNMA flags forgery |
| 🧬 Boot/TPM tampering | PCR ≠ golden values → TPM Quote fails |
| 📡 Beacon jamming | Random k-of-n quorum → must jam all; else fail-secure |
| ⏪ Replay / MITM | Fresh nonce + epoch, mTLS, Ed25519 signatures |

## The website — trustbound.forum

A multi-page bilingual (UA/EN) product site with a **live interactive demo** (`/program/`): drag a laptop around a room and watch access revoke in real time, with **real in-browser cryptography** (HKDF · Ed25519 · AES-GCM). Pages: Home, Benefits, **Tech** (full deep-dive), Implementation (the demo). Served over HTTPS (Let's Encrypt) on an Ubuntu/Nginx VPS.

## Standards & standardization

Built on open standards and mapped onto Zero Trust models — see **[docs/STANDARDS.md](docs/STANDARDS.md)**.

```mermaid
%%{init: {'theme':'dark','themeVariables':{'fontSize':'15px','lineColor':'#6b86ad'}}}%%
flowchart LR
  P1["<b>Phase 1 · Months 1–6</b><br/> MVP on UWB dev boards<br/> Integrate TPM 2.0 API<br/> Latency optimisation"]:::a
  P2["<b>Phase 2 · Months 6–12</b><br/> Proof of Concept in a lab<br/> Linux / Windows client daemon"]:::b
  P3["<b>Phase 3 · Year 2</b><br/> Independent crypto audit<br/> State certification (SSSCZI)<br/> First paid commercial pilot"]:::c
  P1 ==> P2 ==> P3
  classDef a fill:#12213c,stroke:#4f8cff,color:#eaf2ff;
  classDef b fill:#0b2f3a,stroke:#52d6ff,color:#eaf2ff;
  classDef c fill:#0b332d,stroke:#35d0ba,color:#eaf2ff;
```

## Team

Students of the Igor Sikorsky Kyiv Polytechnic Institute — major **Cybersecurity**, CTF team **LeetSh4d0ws**.

---
*Status: early R&D / pre-seed. Concept + working demo. Grounded in NIST SP 800-207 and the DoD Zero Trust Strategy.*
