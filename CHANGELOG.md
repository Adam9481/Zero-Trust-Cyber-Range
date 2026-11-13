## [2026-11-13] - DFIR-IRIS Implementation

### Added
- DFIR-IRIS
<details>
<summary>Click to expand</summary>

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
</details>   

### Changed
- 

### Security Decisions
- 
  
## [2026-11-12] - Host-Level Micro-Segmentation & Baseline established

### Added
- **UFW Micro-segmentation implemented across Management infrastructure:**
  - `Blue Team VM`: Configured default deny incoming / allow outgoing to act as a secure jump box.
  - `Wazuh LXC`: Restricted inbound traffic strictly to ports `1514`, `1515`, and `55000`.
  - `Splunk LXC`: Restricted inbound traffic strictly to ports `8000` and `9997`.
- **Baseline Architecture Documentation:** Initialized `README.md` defining the macro/micro-segmentation strategy and VLAN rules.
- **Docker Engine Install on DFIR-IRIS LXC:** Preparing to implement SOC ticketing system with DFIR-IRIS

### Changed
- Verified OPNsense perimeter rules: Detonation network explicitly blocks all RFC 1918 traffic prior to temporary internet access rules to prevent malware lateral movement.

### Security Decisions
- **Proxmox Hypervisor Firewall:** Left explicitly DISABLED on all nodes to prevent redundant packet filtering conflicts with OPNsense and UFW.
- **Red/Victim VMs:** Host firewalls intentionally left DISABLED on Kali Linux and Ubuntu Server to allow frictionless reverse shell callbacks, payload delivery, and exploit testing.
