# Proxmox Cyber Range & Automated SOC Pipeline

## Overview
This repository documents the baseline architecture of a self-hosted Proxmox Cyber Range. The environment is engineered from scratch to replicate an enterprise-level Security Operations Center (SOC). The primary objective is to develop an end-to-end telemetry pipeline (Wazuh to Splunk to DFIR-IRIS) for incident response, culminating in a human-in-the-loop Retrieval-Augmented Generation (RAG) triage assistant to combat alert fatigue.

## Network Architecture & Segmentation
The lab utilizes Proxmox virtual network interfaces routed through an OPNsense firewall to enforce strict traffic boundaries. 

*   **vmbr0 (Home Network / WAN):** External connectivity to the physical home network. To the virtual firewall, this serves as the untrusted outside interface.
*   **vmbr1 (Segmented Network / LAN):** The internal lab network trunk line sitting behind OPNsense.

Within OPNsense, traffic is strictly micro-segmented using the following VLAN tags, matching the third octet to the VLAN ID for clean subnet routing:
*   **VLAN 20 (Management Interface - `10.0.20.0/24`):** Houses the core security stack.
*   **VLAN 12 (Attacker Interface - `10.0.12.0/24`):** Dedicated to offensive operations.
*   **VLAN 11 (Vulnerable Interface - `10.0.11.0/24`):** Hosts target machines for attack simulations.
*   **VLAN 10 (Detonation Interface - `10.0.10.0/24`):** Isolated environment for safely inspecting malware PCAP files without risk of Remote Code Execution (RCE).
*   **VLAN 30 (Apps Interface - `10.0.30.0/24`):** Designated for the future agentic AI RAG pipeline.
*   **Tailscale Interface:** Configured for secure remote VPN access into the lab environment.

## Core Infrastructure & Telemetry Pipeline
All systems are deployed on virtual machines or Linux Containers (LXC) within Proxmox.

*   **Management (VLAN 20):**
    *   **Kali Purple (Blue Team VM):** The sole Bastion host providing secure web GUI and SSH ingress to the SIEM and case management platforms.
    *   **Wazuh Manager (LXC):** Receives endpoint telemetry via TCP 1514.
    *   **Splunk SIEM (LXC):** Aggregates data forwarded from Wazuh Manager via the Splunk Universal Forwarder on TCP 9997.
    *   **DFIR-IRIS (LXC):** Case management hub for clustering related alerts and documenting investigations via text-driven War Rooms.
*   **Offensive & Target Assets:**
    *   **Kali Red Team (VM - VLAN 12):** Executes attacks across the segmented network.
    *   **Ubuntu Server Victim (VM - VLAN 11):** The primary target running a Wazuh Agent to generate security telemetry during Red Team attacks.
    *   **Canary (VM - VLAN 10):** Isolated malware detonation VM.

## Operational Security (OPSEC) Principles
To mirror real-world enterprise standards, this deployment enforces strict access controls:
1.  **Host-Level Micro-Segmentation:** UFW is configured across the environment to deny all incoming traffic by default. Services like the DFIR-IRIS web interface (port 443) and SSH (port 22) are explicitly whitelisted only for the Bastion Host IP.
2.  **Zero-Knowledge Secrets:** No default testing credentials are used. Passwords and API tokens are generated using OpenSSL cryptographic salts and stored exclusively within a zero-knowledge Bitwarden vault.
3.  **Role-Based Access Control (RBAC):** Multi-tenant controls are enforced within the application layer, separating default administrative access from standard Analyst accounts.

Additional Details Displayed in CHANGELOG.md
