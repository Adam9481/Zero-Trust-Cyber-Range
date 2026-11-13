## [2026-11-12] - Host-Level Micro-Segmentation & Baseline established

### Added
- **UFW Micro-segmentation implemented across Management infrastructure:**
  - `Blue Team VM`: Configured default deny incoming / allow outgoing to act as a secure jump box.
  - `Wazuh LXC`: Restricted inbound traffic strictly to ports `1514`, `1515`, and `55000`.
  - `Splunk LXC`: Restricted inbound traffic strictly to ports `8000` and `9997`.
- **Baseline Architecture Documentation:** Initialized `README.md` defining the macro/micro-segmentation strategy and VLAN rules.
- **Docker Engine Install on DFIR-IRIS LXC:**
 - Preparing to implement SOC ticketing system with DFIR-IRIS

### Changed
- Verified OPNsense perimeter rules: Detonation network explicitly blocks all RFC 1918 traffic prior to temporary internet access rules to prevent malware lateral movement.

### Security Decisions
- **Proxmox Hypervisor Firewall:** Left explicitly DISABLED on all nodes to prevent redundant packet filtering conflicts with OPNsense and UFW.
- **Red/Victim VMs:** Host firewalls intentionally left DISABLED on Kali Linux and Ubuntu Server to allow frictionless reverse shell callbacks, payload delivery, and exploit testing.
