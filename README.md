# Aktivitet 4 – Systemintegration, AAA & SSO

This repository contains the practical implementation for **Activity 4** of the Network Technology course, focusing on AAA (Authentication, Authorization, Accounting) and Single Sign-On (SSO) in a simulated network environment.

A full project report in Swedish is available in the repository as a PDF.

---

## Project Overview

The goal of this project was to:

- Implement an AAA solution using FreeRADIUS and test different user roles
- Configure SSO between two systems using Keycloak
- Integrate RADIUS authentication directly into Keycloak
- Analyze how system integration affects security, usability, and troubleshooting
- Reflect on incident management procedures and user advisory

---

## Architecture

​```
┌─────────────────────────────────────────────────────┐
│                  Proxmox VE Host                    │
│                                                     │
│  ┌──────────────────┐    ┌──────────────────────┐   │
│  │  VM1             │    │  VM2                 │   │
│  │  FreeRADIUS      │    │  Keycloak 26.4.0     │   │
│  │  192.168.1.111   │    │  192.168.1.112       │   │
│  │  Ubuntu Server   │    │  Ubuntu Server       │   │
│  │  22.04           │    │  22.04               │   │
│  └──────────────────┘    └──────────────────────┘   │
│                                                     │
│  ┌──────────────────────────────────────────────┐   │
│  │  VM3  –  Client                              │   │
│  │  192.168.1.113  –  Ubuntu Desktop 22.04      │   │
│  │  app-one (port 8081)  ·  app-two (port 8082) │   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
​```

---

## Technologies Used

| Component | Technology | Version |
|---|---|---|
| Virtualisation | Proxmox VE | 8.x |
| AAA Server | FreeRADIUS | 3.2.8 |
| Identity Provider / SSO | Keycloak | 26.4.0 |
| RADIUS plugin for Keycloak | keycloak-radius-plugin | 1.6.1 |
| Web applications | Python / Flask | 3.x |
| Authentication protocol | OpenID Connect (OIDC) | — |
| Network auth protocol | RADIUS | RFC 2865 |
| Operating system (servers) | Ubuntu Server | 22.04 LTS |
| Operating system (client) | Ubuntu Desktop | 22.04 LTS |

---

## Repository Structure

​```
.
├── app_one.py          # Discover Sicily – tourist portal web app (port 8081)
├── app_two.py          # La Tavola Siciliana – restaurant web app (port 8082)
└── report.pdf          # Full project report (Swedish)
​```

---

## Web Applications

Two Flask web applications were built to demonstrate the SSO flow. Both delegate authentication to Keycloak via OIDC.

**App One – Discover Sicily** (`port 8081`)
A tourist portal for Sicily. Login is required to access personalised itineraries and exclusive tour bookings.

**App Two – La Tavola Siciliana** (`port 8082`)
A Sicilian restaurant website. Login unlocks the full members menu and the table reservation system.

Both apps demonstrate SSO: logging into one automatically authenticates the user in the other without re-entering credentials.

---

## SSO Login Flow

​```
User opens app-one
        │
        ▼
app-one redirects to Keycloak
        │
        ▼
User enters credentials on Keycloak login page
        │
        ▼
Keycloak verifies credentials and issues JWT token
        │
        ▼
User is redirected back to app-one (callback URL)
        │
        ▼
User is authenticated in app-one ✓
        │
        ▼
User opens app-two → Keycloak recognises active session
        │
        ▼
User is authenticated in app-two without re-entering credentials ✓
                    SSO verified
​```

---

## AAA with FreeRADIUS

FreeRADIUS handles AAA for network-level authentication. Three users were configured with different roles:

| Username | Role |
|---|---|
| admin | Network Administrator |
| user1 | Standard User |
| guest | Guest |

Passwords are stored as MD5 hashes. The shared secret was changed from the default `testing123` to a stronger value, and BlastRADIUS protection (CVE-2024-3596) was enabled via `require_message_authenticator = true`.

---

## Keycloak RADIUS Integration

Keycloak was extended with the [keycloak-radius-plugin](https://github.com/vzakharchenko/keycloak-radius-plugin), making it act as a RADIUS server on ports 1812 (auth) and 1813 (accounting). This means Keycloak serves as a centralised identity provider for both:

- **Web applications** via OpenID Connect
- **Network devices** via RADIUS

---

## Prerequisites (to reproduce)

- Proxmox VE with at least 8 GB RAM available
- Three Ubuntu VMs as described in the architecture section
- Python 3.x with `pip` on the client VM
- Java 21 on the Keycloak VM

---

## Notes

This environment is configured for **lab/development purposes only**. It uses HTTP instead of HTTPS and an embedded H2 database in Keycloak. A production deployment would require TLS certificates, an external database (e.g. PostgreSQL), and high availability configuration.
