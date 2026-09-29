# Standards & standardization

## What TrustBound is built on
- **NIST SP 800-207 — Zero Trust Architecture.** 7 tenets. TrustBound directly delivers the hardest: per-session access (#3), dynamic policy using device & environment posture (#4), continuous posture monitoring (#5), enforced authorization before every access (#6).
- **DoD Zero Trust Strategy (2022).** 7 pillars. TrustBound strengthens **Device** (TPM attestation), **User** (auth + presence), **Data** (presence-bound encryption), **Network/Environment** (geofence + UWB), **Visibility** (tamper-evident audit).
- **TCG TPM 2.0** — hardware root of identity & attestation.
- **IEEE 802.15.4z / FiRa** — secure UWB ranging (STS, Time-of-Flight).
- **Galileo OSNMA** — satellite navigation-message authentication.
- **Cryptography** from FIPS 140-3-validated modules (AES-GCM, SHA-256, HKDF) + Ed25519; **TLS 1.3**, **FIDO2/WebAuthn**.

## Legal aspects to plan for
- **Personal data:** location & presence data are personal data → EU **GDPR** (lawful basis, minimisation, **DPIA**); in Ukraine — the Law on Personal Data Protection.
- **State information systems (Ukraine):** Law on Information Protection in ICS; **SSSCZI** (Держспецзв'язку) requirements; building & attesting a **KSZI** (comprehensive information-protection system); **ND TZI** normative documents.
- **Product evaluation:** **Common Criteria (ISO/IEC 15408)**; crypto module — **FIPS 140-3**.
- **Workforce privacy & export control** (dual-use crypto/defence) must be reviewed.

## How to standardize inside a company
1. **Pilot + risk assessment** (and DPIA where needed).
2. **Internal standard / policy** — "Continuous Presence-Based Access Control": scope, roles, zones, key TTL, fail-secure & emergency (two-person) procedures.
3. **Integrate into an ISMS (ISO/IEC 27001)** — as a control that strengthens ISO/IEC 27002 access-control, physical-security and cryptography controls.
4. **SOPs + training** — device enrollment, beacon maintenance, incident response.
5. **Audit → certification** — internal audit, then ISO 27001 (ISMS), Common Criteria (product), FIPS 140-3 (crypto).
6. **Ukraine gov/defence** — align with SSSCZI, build & attest a KSZI; for NATO export align with STANAG / customer requirements.

> Informational overview, not legal advice — engage compliance & legal specialists for real certification.
