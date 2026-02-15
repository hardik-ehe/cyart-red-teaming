# Practical 1  Threat Hunting with Open-Source Tools

## Objective
Detect suspicious PowerShell activity using Sigma rules and Elastic Security.

## Tools Used
- Elastic Security
- Security Onion
- Sigma Rules

## Workflow
1. Ingest Windows logs into Elastic Security.
2. Create a Sigma rule for PowerShell execution.
3. Execute a test PowerShell command in Windows VM.
4. Query Event ID 4688 for validation.
5. Document findings.

## Outcome
Successfully detected PowerShell execution events and validated detection logic.
