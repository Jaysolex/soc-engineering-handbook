# Chapter 16 — NGFW Integration & External Dynamic Lists (EDL)

**Depends on:** Chapter 7 — Lists, Chapter 13 — Response Actions

```
Cortex XDR
    ↓
Detection & Investigation
    ↓
External Dynamic List (EDL)   ← YOU ARE HERE
    ↓
NGFW
    ↓
Network-Wide Enforcement
```

---

## Overview

Every chapter so far has covered what Cortex does *inside itself* — detection, investigation, automation, endpoint response. This chapter covers the boundary where Cortex hands off to a separate product: the **Next-Generation Firewall (NGFW)**.

Cortex XDR and the NGFW are not the same product, and understanding exactly where one stops and the other starts is what makes the EDL integration make sense. Get this boundary wrong and you'll either expect Cortex to do something only the firewall can do (block traffic in real time, network-wide) or expect the firewall to do something only Cortex can do (correlate an alert into an attack story).

## Learning Objectives

- What a Next-Generation Firewall is and how it differs from a traditional firewall
- Why NGFW and Cortex XDR are separate products with distinct responsibilities
- What an External Dynamic List (EDL) is and how it connects the two
- The end-to-end workflow from detection to organization-wide network block
- Why this integration matters operationally

---

## Traditional Firewall vs. NGFW

A traditional firewall mainly looks at **Source IP, Destination IP, Port, Protocol**.

```
Allow: 10.0.0.5 → 8.8.8.8, Port 443
```

It doesn't deeply inspect the traffic.

A **Next-Generation Firewall** understands the content and context of network traffic. It can inspect:

- Applications (e.g., Zoom, Dropbox, Teams)
- Users (Active Directory integration)
- URLs and web categories
- Malware
- Intrusion attempts (IPS)
- Threat Intelligence
- SSL/TLS encrypted traffic (SSL decryption)
- External Dynamic Lists (EDLs)

**Easy way to remember:**

```
Traditional Firewall
        │
Controls traffic using
      IP + Port
────────────────────────────
Next-Generation Firewall
        │
Controls traffic using
IP + Port + User + Application +
Threat Intelligence + Malware Detection +
IPS + URL Filtering + EDL
```

An NGFW is an advanced firewall that goes beyond basic IP and port filtering by inspecting applications, users, content, and threats — and it integrates with threat intelligence sources such as a Cortex EDL to automatically block known malicious IPs and domains across the network.

## Popular NGFW Vendors

- **Palo Alto Networks NGFW** — the one that integrates natively with Cortex
- Cisco Secure Firewall (formerly Firepower)
- Fortinet FortiGate
- Check Point Quantum
- Juniper SRX
- Sophos XGS

---

## Is the NGFW Inside Cortex?

**No.** The NGFW is a separate product from Cortex. Think of them as teammates.

```
           Palo Alto Ecosystem

      ┌────────────────────────┐
      │       Cortex XDR       │
      │ Detects & Investigates │
      └──────────┬─────────────┘
                 │
          Shares Threat Intel
            (EDL, APIs, etc.)
                 │
                 ▼
      ┌────────────────────────┐
      │    Palo Alto NGFW      │
      │ Blocks Network Traffic │
      └────────────────────────┘
```

### Their Roles

**Cortex XDR** is responsible for: detecting threats, creating Issues (Chapter 10), creating Cases (Chapter 12), running Playbooks (Chapter 3), running Scripts (Chapter 15), isolating endpoints (Chapter 14), threat hunting, and producing EDLs.

*Think: "Brain and Investigation Platform."*

**NGFW** is responsible for: inspecting network traffic, allowing or denying connections, blocking malicious IPs and domains, IPS (Intrusion Prevention), URL filtering, and application control.

*Think: "Network Security Guard."*

### Simple Analogy

Imagine a company building.

**🕵️ Cortex = the detective inside the building.** Investigates suspicious behavior, finds the criminal, and tells the security guard: *"Don't let this person back in."*

**👮 NGFW = the security guard at the front gate.** Decides who can enter or leave the building. Acts on what the detective tells it.

Neither role can substitute for the other — the detective can't physically stop someone at the gate, and the guard has no way to investigate what happened inside.

---

## How EDL Connects Them

```
Cortex XDR
    │
    ▼
Detects malicious IP
    │
    ▼
Adds IP to EDL
    │
    ▼
NGFW downloads EDL
    │
    ▼
NGFW blocks traffic
    │
    ▼
Attacker cannot communicate
```

### Example

Suppose your ransomware playbook identifies this C2 server: `185.100.87.14`.

Cortex adds the IP to the EDL. The NGFW downloads the updated list. Now every employee is protected:

```
Laptop A ─────X────► 185.100.87.14
Laptop B ─────X────► 185.100.87.14
Server 1 ─────X────► 185.100.87.14
Server 2 ─────X────► 185.100.87.14
```

The firewall blocks all communication to that malicious IP — for the whole organization, not just the originally infected endpoint.

### The Key Point: Cortex Does Not Directly Command the Firewall

Cortex doesn't directly tell the firewall "block this IP now." Instead, it updates the EDL feed, and the NGFW pulls from that feed on its own schedule:

```
Every few minutes...

     NGFW
       │
       ▼
Downloads latest EDL
       │
       ▼
Sees new malicious IP
       │
       ▼
Updates its block list
       │
       ▼
Begins blocking traffic
```

This is the same pull-based pattern as Jobs and Lists from Chapter 7 and Chapter 8 — Cortex doesn't push individually to every consumer; it updates one shared source of truth (the EDL), and every subscriber refreshes from it independently. That design is what lets one detection protect an entire fleet of firewalls without Cortex needing a direct connection to each one.

---

## Real Production Example — Ransomware Sigma Rule

Your ransomware Sigma rule detects communication with `185.100.87.14`.

```
 Sigma Rule
      │
      ▼
  Cortex XDR
      │
      ▼
    Issue
      │
      ▼
     Case
      │
      ▼
Threat Intel = Malicious
      │
      ▼
Add 185.100.87.14 to EDL
      │
      ▼
NGFW refreshes EDL
      │
      ▼
Blocks all outbound connections
to 185.100.87.14
      │
      ▼
Even clean machines cannot
communicate with that server
```

This is why EDL is so powerful: **one investigation can protect the entire organization**, not just the originally infected endpoint. It's also a direct extension of the Response Action target categories from Chapter 13 — EDL is the "Networks" target, and this chapter is the full mechanics of how that target actually gets enforced.

## Key Takeaways

- NGFW is a separate Palo Alto product from Cortex XDR — they integrate, but neither replaces the other.
- Cortex focuses on detection, investigation, automation, and endpoint response. NGFW enforces network security policy.
- An EDL (External Dynamic List) is the bridge between them: Cortex publishes confirmed malicious IPs/domains to the EDL, and the NGFW consumes that list on a refresh cycle to block traffic automatically.
- Cortex never issues a direct "block now" command to the firewall — the integration is pull-based, the same architectural pattern used by Jobs and Lists elsewhere in this handbook.
- Because the NGFW protects every device behind it, updating an EDL from one investigation extends protection organization-wide, not just to the compromised endpoint.
