---
title: IEEE 802.1X Port-Based Network Access Control
version: 1.0.0
status: Supported
last_updated: 2026-02-02
ieee_reference: IEEE 802.1X-2020
---
## 802.1X Architecture

### Component Overview

```mermaid
graph LR
    subgraph ENDPOINT["Endpoint Layer"]
        SUP["Supplicant<br/>(Client software)"]
    end

    subgraph NETWORK["Network Layer"]
        AUTH["Authenticator<br/>(Switch/AP)"]
    end

    subgraph BACKEND["Backend Layer"]
        RADIUS["Authentication/NAC Server<br/>(RADIUS)"]
        AD["Directory Service"]
        CA["Certificate Authority"]
    end

    SUP <-->|"EAP over LAN<br/>(EAPoL)"| AUTH
    AUTH <-->|"RADIUS<br/>(UDP 1812/1813)"| RADIUS
    RADIUS <--> AD
    RADIUS <--> CA
    
