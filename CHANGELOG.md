<details>
<summary><strong>[2026-11-24] - DISA STIG Baseline & OpenSCAP Integration</strong></summary>

### Added
- **Ubuntu 24.04 LTS Victim VM:** Provisioned a headless Ubuntu 24.04.2 Server VM on VLAN 11 with OpenSSH and `openscap-scanner` installed.
- **Baseline DISA STIG Audit:** Executed an initial remote compliance scan from the Blue Team VM (Kali Purple) using `oscap-ssh` against the `xccdf_org.ssgproject.content_profile_stig` profile, generating `ubuntu2404-arf.xml` and `ubuntu2404-stig-report.html` for triage.
- **Clean Baseline Snapshot:** Captured Proxmox snapshot `clean-2404-openscap` immediately after locking down the firewall to enable safe rollbacks during upcoming STIG remediation exercises.

<details>
<summary><strong>View Scanning & Setup Commands</strong></summary>

```bash
# 1. Install scanner engine on Ubuntu 24.04 Victim VM (during temporary OPNsense maintenance window)
sudo apt update && sudo apt install -y openscap-scanner

# 2. Verify STIG profile availability on Blue Team VM (Kali Purple)
mkdir -p ~/scans/openscap && cd ~/scans/openscap
oscap info /usr/share/xml/scap/ssg/content/ssg-ubuntu2404-ds.xml | grep -i stig

# 3. Execute remote DISA STIG audit over SSH
oscap-ssh --sudo adam@<ubuntu_ip> 22 xccdf eval \
  --profile xccdf_org.ssgproject.content_profile_stig \
  --results-arf ubuntu2404-arf.xml \
  --report ubuntu2404-stig-report.html \
  /usr/share/xml/scap/ssg/content/ssg-ubuntu2404-ds.xml
```

</details>

### Changed
- **Target OS Pivot (26.04 to 24.04 LTS):** Replaced the initial Ubuntu 26.04 victim VM with Ubuntu 24.04 LTS after verifying that Kali's `ssg-ubuntu2604-ds.xml` datastream was an early 118 KB stub lacking a complete DISA STIG benchmark.

### Security Decisions
- **Temporary Provisioning Window:** Briefly disabled the OPNsense outbound block rule on VLAN 11 to pull `openscap-scanner` onto a clean, uncompromised VM, then immediately restored air-gapped isolation and snapshotted the baseline. This avoids manual offline dependency resolution while ensuring the VM never touches the internet once testing begins.
- **Offline Remote Scanning:** Executed `oscap-ssh` from the Blue Team VM without `--fetch-remote-resources` so compliance auditing operates reliably across an air-gapped VLAN.

</details>

<details>
<summary><strong>[2026-11-13] - DFIR-IRIS Implementation</strong></summary>

### Added
- **DFIR-IRIS:** Deployed the incident response platform on Ubuntu LXC (10.0.20.40).

<details>
<summary><strong>View Deployment Commands</strong></summary>

```bash
# 1. Clone stable v2 release
git clone https://github.com/dfir-iris/iris-web.git
cd iris-web
git checkout v2.4.24

# 2. Configure production secrets
cp .env.model .env
nano .env
# Added Bitwarden-generated DB passwords and OpenSSL-generated salts.
# Set IRIS_HOSTNAME=10.0.20.40

# 3. Generate TLS Certificates for Management IP
mkdir -p certificates/web_certificates
openssl req -x509 -newkey rsa:2048 -sha256 -days 365 -nodes \
    -keyout certificates/web_certificates/iris_dev_key.pem \
    -out certificates/web_certificates/iris_dev_cert.pem \
    -subj "/CN=10.0.20.40" \
    -addext "subjectAltName=IP:10.0.20.40"
chmod 600 certificates/web_certificates/iris_dev_key.pem

# 4. Build and Deploy
docker compose build
docker compose up -d
```

</details>

### Changed
- **VLAN Routing:** Resolved a silent packet drop issue by adding the missing Management VLAN tag on the Proxmox LXC network bridge.
- **Access Control:** Configured host-level firewall rules on the DFIR-IRIS LXC to restrict inbound web interface (HTTPS/443) and SSH traffic exclusively to the Blue Team VM IP address.
  
### Security Decisions
- **Production Secrets:** Opted against default testing credentials. Generated cryptographic salts via OpenSSL and unique database passwords stored exclusively in a self-hosted Bitwarden vault.
- **Local Authentication:** Configured IRIS for local DB auth to maintain independence from external identity providers during range operations.
- **Bastion Host Isolation:** Mandated that DFIR-IRIS is only accessible via the Blue Team VM acting as a secure jump box. This enforces a strict administrative chokepoint, prevents direct access from the broader network, and accurately mirrors enterprise zero-trust architecture.

</details>

<details>
<summary><strong>[2026-11-12] - Host-Level Micro-Segmentation & Baseline Established</strong></summary>

### Added
- **UFW Micro-segmentation implemented across Management infrastructure:**
  - `Blue Team VM`: Configured default deny incoming / allow outgoing to act as a secure jump box.
  - `Wazuh LXC`: Restricted inbound traffic strictly to ports `1514`, `1515`, and `55000`.
  - `Splunk LXC`: Restricted inbound traffic strictly to ports `8000` and `9997`.
- **Baseline Architecture Documentation:** Initialized `README.md` defining the macro/micro-segmentation strategy and VLAN rules.
- **Docker Engine Install on DFIR-IRIS LXC:** Preparing to implement SOC ticketing system with DFIR-IRIS.

### Changed
- **Verified OPNsense perimeter rules:** Detonation network explicitly blocks all RFC 1918 traffic prior to temporary internet access rules to prevent malware lateral movement.

### Security Decisions
- **Proxmox Hypervisor Firewall:** Left explicitly DISABLED on all nodes to prevent redundant packet filtering conflicts with OPNsense and UFW.
- **Red/Victim VMs:** Host firewalls intentionally left DISABLED on Kali Linux and Ubuntu Server to allow frictionless reverse shell callbacks, payload delivery, and exploit testing.

</details>
