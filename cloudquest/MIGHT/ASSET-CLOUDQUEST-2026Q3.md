ASSET LOG

####### <!-- ANCHOR MARKER - ADD ALL NEW ASSET ENTRIES DIRECTLY BELOW THIS LINE -->

## ASSET:cloudquest 2026-11-05 -> BADGE EARNED: AWS Cloud Quest: Security - Trained

**Badge:** AWS Cloud Quest: Security - Training Badge
**Issued by:** Amazon Web Services Training and Certification
**Date issued:** November 5, 2026
**Credly URL:** https://www.credly.com/earner/earned/badge/94957dfe-68d7-46d8-9503-e5eba8e0a9db
**Skills:** AWS, AWS Cloud, Amazon Web Services (AWS), Cloud Security, Data Security, IAM, Security Architecture, Security Management, Serverless Security

---

## ASSET:cloudquest 2026-11-05 -> Cognito Pre-SignUp Lambda trigger DIY

**Task:** Add a Pre-SignUp Lambda trigger to the Cognito user pool to auto-confirm users (remove validation code process).

| Field | Value |
|---|---|
| User pool | `bookmark-app-userpool` |
| User pool ID | `us-east-1_3KUAT2O6K` |
| Trigger type | Pre sign-up Lambda trigger |
| Lambda function | `preSignHandlerLambdaFunction` |

**Result:** Trigger added successfully.

---

## ASSET:cloudquest 2026-11-05 -> AWS Glue crawler DIY: crawl Athena-output folder → new Data Catalog table

**Task:** Create a new Glue crawler pointing at the Athena query results folder and run it to produce a new Data Catalog table.

**Crawler config (matched practice crawler `logcrawler`):**
| Field | Value |
|---|---|
| Crawler name | `athena-output-crawler` |
| Data source | `s3://log-bucket-d864d830/Athena-output/` |
| IAM role | `glue-service-role-d864d830` |
| Target database | `logsdatabase` |
| Schedule | On demand |

**Dependency chain discovered:**
1. `logcrawler` had never been run → ran it (1 min 16s, 1 table created: `log_bucket_d864d830` in `logsdatabase`)
2. Ran Athena query `SELECT * FROM "logsdatabase"."log_bucket_d864d830" LIMIT 10` → populated `s3://log-bucket-d864d830/Athena-output/` with a CSV (query ID: `6882b5b9-a91e-4d9c-8b16-d5e189bee420`)
3. Re-ran `athena-output-crawler` → 2 tables created

**New Data Catalog tables (created by athena-output-crawler, 2nd run):**
| Table name | Classification | Database |
|---|---|---|
| `6882b5b9_a91e_4d9c_8b16_d5e189bee420_csv` | CSV | `logsdatabase` |
| `6882b5b9_a91e_4d9c_8b16_d5e189bee420` | UNKNOWN | `logsdatabase` |

**Primary answer to DIY (corrected — table name must contain 'athena'):**
- New table name: `athena_6882b5b9_a91e_4d9c_8b16_d5e189bee420_csv`
- Database: `logsdatabase`
- Fix: edited `athena-output-crawler` to add Table name prefix `athena_`, then re-ran (3rd run, 40s, 2 table changes)

**Key detail:** The existing `logcrawler` (practice section) uses `s3://log-bucket-d864d830` (bucket root). The DIY crawler targets the `/Athena-output/` subfolder specifically. Use the same IAM role and database to avoid permission issues.

---

## ASSET:cloudquest 2026-11-05 -> CloudWatch + Lambda + SNS DIY: email subscription to UnauthorizedExceptionNotification

**Task:** Add a confirmed email subscription to the SNS topic used in the practice section, so the topic has both a Lambda and an email subscriber.

**SNS topic:** `UnauthorizedExceptionNotification`
**Topic ARN:** `arn:aws:sns:us-east-1:741448922577:UnauthorizedExceptionNotification`

**Steps:**
1. SNS → Topics → UnauthorizedExceptionNotification → Create subscription
2. Protocol: Email / Endpoint: `jayreck996@gmail.com`
3. Click Create subscription (status: Pending confirmation)
4. Open confirmation email → click "Confirm subscription" link (check spam)

**Result:** Both subscriptions confirmed on the topic:
- Lambda: `arn:aws:lambda:us-east-1:741448922577:...` (pre-existing from practice)
- Email: `jayreck996@gmail.com` — subscription ID `46550afd-234c-4613-99ce-6a23904b647a`

**Grader check:** validates that the same SNS topic has a confirmed email subscription AND a valid Lambda function subscription.

---

## ASSET:cloudquest 2026-11-04 -> Service Catalog DIY: deploy three-tier AppStack as catalogUser

**Task:** Deploy the AppStack product as IAM user `catalogUser`, using the Base-Networking stack as the network layer.

**Key parameter:** When launching AppStack, the `NetworkStackName` parameter must be set to the CloudFormation stack name created by the Base-Networking provisioned product — not the provisioned product name itself. Format: `SC-<account-id>-pp-<id>`.

**Lab values:**
- Account ID: `741448922577`
- NetworkStackName: `SC-741448922577-pp-swvbx3cfdsy7y`
- AppStack provisioned product ID: `pp-ippgq4wvtliu2`
- AppStack product ID: `prod-gg4juvu6esgee` (Owner: Dev team)
- Base-Networking product ID: `prod-srfhfrgenq3w6` (Owner: InfoSec Department)

**Result:** Both `NetworkTemplate` (Base-Networking v1) and `AppStack` (App-v1.0) provisioned products reached **Available** status.

**Portfolio name visibility:** The portfolio name is NOT visible from the `catalogUser` end-user view — it only appears in the admin console under Administration → Portfolios. The launch constraint on AppStack is named `3TierApp`. To find the portfolio name via CLI: `aws servicecatalog list-portfolios-for-product --product-id prod-gg4juvu6esgee`.

---

## ASSET:cloudquest 2026-11-04 -> API Gateway DIY: Cognito authorizer + labFunction-diy on /lab GET

**Task:** Configure the existing `/lab` GET method in API Gateway to use `labFunction-diy` as the Lambda integration and `labAuthorizer` (Cognito) as the authorizer, then deploy to the `prod` stage.

**Mistake to avoid:** Do NOT create a new `/diy` resource. The grader tests the existing `/lab` path. The fix is to edit the integration request on the existing `/lab` GET method and change the Lambda function from `labFunction-practice` to `labFunction-diy`.

**Steps:**
1. API Gateway → REST API → Resources → `/lab` → GET → Integration Request → Edit
2. Change Lambda function to `labFunction-diy`
3. Method Request → Authorization → set to `labAuthorizer` (Cognito user pool authorizer)
4. Deploy API → Stage: `prod`

**Cognito authorizer:** `labAuthorizer` — pool ID `048dwu - 3rdbQnY1x` (token source: Authorization header)

**Invoke URL:** `https://6qdypmh6kf.execute-api.us-east-1.amazonaws.com/prod`

**Grader endpoint tested:** `https://6qdypmh6kf.execute-api.us-east-1.amazonaws.com/prod/lab`

---

## ASSET:cloudquest 2026-11-02 -> AWS Skill Builder subscription details

**Plan:** AWS Skill Builder Individual Annual Subscription (free, does not renew)

**Period:** Sep 15, 2026 – Sep 14, 2027

**Status:** Active

**Price:** Free (complimentary annual subscription — not the standard $449/year paid tier)

**Note:** This subscription grants full access to Cloud Quest and all individual subscriber content. It will not auto-renew at expiry.

## ASSET:cloudquest 2026-10-28 -> Security groups: stateful behaviour explained

**Rule:** Security groups are **stateful** — they track active network sessions.

**What this means:**
- If your instance sends an outbound request, the security group remembers the session
- The inbound *response* to that request is automatically allowed — no inbound rule required
- Only new inbound connections from outside need an explicit allow rule

**Contrast with NACLs (stateless):** NACLs treat every packet independently — you must explicitly allow both directions.

| Traffic | Stateful SG behaviour |
|---|---|
| Outbound request you initiated | Allowed (outbound rule) |
| Inbound response to your request | Auto-allowed (state tracking) |
| New inbound connection from outside | Blocked unless explicit inbound rule |

---

## ASSET:cloudquest 2026-10-28 -> IAM policy JSON: ec2_app_policy with S3/EC2/DynamoDB permissions

**Policy:** `ec2_app_policy` — required permissions for Compliance_App_Role validator.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeAccountAttributes",
                "ec2:DescribeInstances",
                "s3:ListBucket",
                "s3:PutObject",
                "s3:GetObject",
                "s3:DeleteObject",
                "dynamodb:PartiQLSelect"
            ],
            "Resource": "*"
        }
    ]
}
```

**DIY task steps:**
1. Add `s3:DeleteObject` to `ec2_app_policy` (edit existing policy)
2. Create IAM role `Compliance_App_Role` → attach `ec2_app_policy` + permissions boundary `secPolBoundaries`
3. Attach role to `Compliance-Server` EC2 instance via Actions → Security → Modify IAM role

---

## ASSET:cloudquest 2026-10-27 -> AWS Config remediation: ec2-instance-detailed-monitoring-enabled

**Lab tasks:**
1. Create SSM Automation document (`MyAutomationDoc`) using provided YAML
2. Deploy Config rule `ec2-instance-detailed-monitoring-enabled`
3. Edit remediation action → set to `MyAutomationDoc` (not the pre-existing `Lab-TagProdInstances`)
4. Select HR-WebServer in noncompliant resources → Remediate

**Key issue:** When deploying the Config rule, it defaulted to `Lab-TagProdInstances` as the remediation action. That document only adds tags — it does NOT enable detailed monitoring. Must manually edit the remediation action to use the new `MyAutomationDoc`.

**SSM document YAML (use as-is — do not replace the Lambda ARN account ID):**
```yaml
schemaVersion: '0.3'
parameters:
  InstanceId:
    type: String
    description: ID of the instance to enable detailed monitoring
mainSteps:
  - name: enableDetailedMonitoring
    action: aws:invokeLambdaFunction
    isEnd: true
    inputs:
      InvocationType: Event
      Payload: |
        {"instanceId": "{{ InstanceId }}"}
      FunctionName: arn:aws:lambda:us-east-1:582969442417:function:labFunction-DIY
```

**Hint reminder:** Also check `required-tags` rule — HR-WebServer may need remediation there too.

---

## ASSET:cloudquest 2026-10-27 -> Multi-Region KMS key: creation + automatic rotation

**Task:** Create symmetric multi-Region KMS key with automatic rotation enabled.

**Console path:** KMS → Customer managed keys → Create key

| Field | Value |
|---|---|
| Key type | Symmetric |
| Key usage | Encrypt and decrypt |
| Regionality | **Multi-Region key** |

**Enable rotation:** Key detail page → Key rotation tab → Enable automatic key rotation → Save

**Key ID format:** Multi-Region keys have prefix `mrk-` (e.g. `mrk-6d8ac2fe63964df69e566b7853d3b3c2`)

**Note:** Cannot convert an existing single-Region key to multi-Region — must create new.

**Validator tip:** Submit the full ARN if the key ID alone is rejected:
`arn:aws:kms:us-east-1:<account-id>:key/mrk-<uuid>`

---

## ASSET:cloudquest 2026-10-27 -> KMS envelope encryption: openssl bad decrypt fix

**Task:** Encrypt a file using a KMS-generated data key, then decrypt it.

**Root cause of "bad decrypt" error:** Two issues:
1. **Typo** — encryption used `-k fileb://datakeyPlainText.tx` (missing `t`), decryption used `.txt` → different passphrase strings
2. **Misuse of `-k` flag** — OpenSSL's `-k` takes a literal passphrase string, NOT a file path. `fileb://` is AWS CLI syntax; OpenSSL ignores it as a path and uses the whole string as the passphrase

**Quick fix (recover existing encrypted file):**
```bash
openssl enc -d -aes256 -in encryptedSecret.txt -out decryptedSecret.txt -k "fileb://datakeyPlainText.tx"
# Must match exact string used during encryption (including the typo)
```

**Correct approach for future:**
```bash
# Encrypt
openssl enc -e -aes256 -pbkdf2 -in samplesecret.txt -out encryptedSecret.txt -pass file:datakeyPlainText.txt
# Decrypt
openssl enc -d -aes256 -pbkdf2 -in encryptedSecret.txt -out decryptedSecret.txt -pass file:datakeyPlainText.txt
```

**Key difference:** `-pass file:filename` (OpenSSL syntax) reads the file; `-k fileb://filename` uses the string literally.

## ASSET:cloudquest 2026-10-23 -> EC2 key pairs: SSH public key placement at instance launch

**Concept:** When launching an EC2 instance, you can specify a key pair. On first boot, AWS places the public key into `~/.ssh/authorized_keys` on the Linux instance.

**How it works:**
- Key pair = public key (stored in AWS) + private key (you keep locally)
- At launch → public key written to `~/.ssh/authorized_keys` automatically
- You connect via SSH using your private key — no password needed

**Key lessons:**
- The key pair must be specified **at launch** — it cannot be added this way after the instance is running
- `~/.ssh/authorized_keys` is the standard Linux file SSH checks to allow connections
- Private key stays on your machine and is never sent to AWS

---

## ASSET:cloudquest 2026-10-20 -> IAM console disable + S3 upload via CLI (fnuser)

**Lab tasks:**
1. Disable AWS Management Console access for `fnuser-<suffix>`
2. Connect to LabInstance via Session Manager
3. Upload `Zassy.png` to `fnuser1/` folder in `corp-fnuser-<suffix>` bucket using AWS CLI

**Grader checks:**
- `get_login_profile` API → confirms console access disabled for fnuser
- `s3 head_object` → confirms `Zassy.png` exists at `fnuser1/Zassy.png` in corp-fnuser bucket

**Session findings:**
- Bucket name: `corp-fnuser-63104c60`
- `Zassy.png` already present at `/home/ec2-user/Zassy.png` (78,322 bytes)
- Instance role (`Lab-ManagedInstanceRole`) has no `s3:ListAllMyBuckets` — `aws s3 ls` fails until fnuser credentials configured
- fnuser has access to `corp-fnuser-63104c60` but not `corp-hruser-63104c60` (AccessDenied as expected)

**CLI credential setup (required — instance role insufficient):**
```bash
aws configure
# AWS Access Key ID: AKIA2NNOIYFK4GYNQKNG   ← lab-scoped fnuser key
# Region: us-east-1 / Output: json
```

**Upload command:**
```bash
aws s3 cp ~/Zassy.png s3://corp-fnuser-63104c60/fnuser1/Zassy.png
```

**Key lessons:**
- Must `sudo su - ec2-user` first (Session Manager starts as ssm-user)
- Instance role ≠ fnuser permissions — configure fnuser IAM keys explicitly
- `fnuser1/` is a key prefix (folder), not a pre-existing S3 folder — the upload creates it
- Disable console access via IAM → Users → fnuser → Security credentials → Console sign-in → Disable

---

## ASSET:cloudquest 2026-10-16 -> Security group: allow port 3306 from web server to DB server

**Lab task:** Allow web server (subnet `10.10.0.0/24`, SG `WebServerSecurityGroup`) to connect to DB server (subnet `10.10.2.0/24`, SG `DbServerSecurityGroup`) over TCP port 3306 (MySQL/Aurora).

**Fix applied:** Added inbound rule to `DbServerSecurityGroup`:
- Type: MySQL/Aurora
- Protocol: TCP
- Port: 3306
- Source: `10.10.0.0/24` (web server subnet)

**Error hit first:** Tried using `WebServerSecurityGroup` (`sg-04f1531061448cb5c`) as the source, but got:
> "You may not specify a referenced group id for an existing IPv4 CIDR rule."

**Root cause:** An existing inbound rule on `DbServerSecurityGroup` for port 3306 already used a CIDR source. AWS won't let you mix CIDR and SG-reference sources — you must pick one type per rule.

**Resolution:** Used CIDR source `10.10.0.0/24` instead of the SG reference. Lab test passed — status changed to Connected.

---

## ASSET:cloudquest 2026-10-16 -> AWS Cloud Quest: Generative AI Architect badge COMPLETED

**Badge earned:** AWS Cloud Quest: Generative AI Architect - Training Badge (Amazon Web Services Training and Certification)

**Lab: Moderating AI Chat App Conversations — final working solution**

**Task 1 — ECS Desired tasks: 1 → 2**
- Service: `streamlitservice` (Streamlit-cluster, us-east-1)
- Account: `582969442417` (fresh lab session)
- Updated via ECS Update service console — Desired tasks 1 → 2 ✅

**Task 2 — Lambda DIY code (exact Kiro-provided snippet — use this verbatim):**
```python
# DIY section - Add unsafe responses to the bad_prompts table
if guard_result.startswith("unsafe"):
    dynamodb.put_item(
        TableName="bad_prompts",
        Item={
            "prompt_id":   {"S": random_id},
            "prompt_content": {"S": prompt},
            "guard_resp":  {"S": guard_result},
        },
    )
```
- Uses low-level `dynamodb` client (already defined as `boto3.client("dynamodb")`)
- Uses `random_id`, `prompt`, `guard_result` — all already in scope in `lambda_handler`
- DynamoDB attributes: `prompt_id` (PK), `prompt_content`, `guard_resp`
- **Do NOT use the Kiro expanded pattern** (with `user_id`, `session_id`, `assessments`) — grader expects exactly these 3 attributes

**Key lessons:**
- CodeBuild is available via the AWS console (not CLI) — used to rebuild Streamlit container with `app_v2.py`
- Grader fails in old sessions if `bad_prompts` has items with wrong attributes from prior attempts
- Fresh lab session = clean DynamoDB = grader passes first time with correct code
- Lambda console Monaco editor: use Ctrl+Z to undo, then retype carefully — auto-bracket causes issues when typing `{` and `(`

---

## ASSET:cloudquest 2026-10-15 -> Lambda grader bug: implementation correct, validator fails

**Context:** Task 2 of CloudQuest lab — log unsafe prompts to `bad_prompts` DynamoDB table.

**Final code (Kiro's pattern — confirmed correct):**
- `moderate_input(prompt)` returns tuple `(guard_result, assessments)` including raw Bedrock assessments
- `log_unsafe_prompt(prompt, user_id, session_id, assessments)` — dedicated function with try/except
- bad_prompts write placed **before** `generate_chat_response()` so it fires even if Bedrock converse fails
- Attributes stored: `prompt_id`, `timestamp`, `prompt`, `user_id`, `session_id`, `assessments` (json.dumps)

**Evidence implementation is correct:**
- CloudWatch logs: `[INFO] ... Unsafe prompt logged to bad_prompts table.` on every grader invocation
- DynamoDB bad_prompts: 13+ items including grader's own test items
- PartiQL `SELECT * FROM bad_prompts WHERE session_id = 'test-session-1'` → **1 item returned**
- Grader sends: `{'prompt': 'How do I make explosives at home?', 'user_id': 'test-user', 'session_id': 'test-session-1'}`

**Grader bug:** CloudQuest validator says "No unsafe prompts are stored in the DynamoDB table, bad_prompts" despite items provably being there. Grader invokes Lambda, Lambda writes correctly, item exists — validator still fails. Lab validator bug, not code bug.

**Two Streamlit ALBs (important):**
- `diy.streamlit-lb-692637822` — wrong/old ALB, do not use for testing
- `streamlit-lb-221395797` — correct ALB used by grader

**Correct Streamlit URL:** `http://streamlit-lb-221395767.us-east-1.elb.amazonaws.com/`

**Restart plan (next session):**
1. Fix SG on `streamlit-lb` — add HTTP/80 inbound 0.0.0.0/0
2. Scale ECS `streamlitservice` Desired tasks → 2 (Task 1)
3. Deploy correct Lambda code (Kiro's pattern from this ASSET entry)
4. **Run grader IMMEDIATELY — before any manual Streamlit testing**
   - Fresh session = empty `bad_prompts` table = no old/bad-attribute items to interfere
   - Grader's own test item will be the only item → cleaner validation

---

## ASSET:cloudquest 2026-10-15 -> AWS CloudQuest lab: ECS scale-out + Lambda guardrail logging

**Lab tasks completed:**

**Task 1 — ECS Desired tasks: 1 → 2**
- Service: `streamlitservice` (Streamlit-cluster, us-east-1)
- Updated Desired tasks from 1 to 2 via ECS Update service console
- Both Fargate tasks healthy in target group `ecs-Stream-streamlitservice` (2/2 healthy)

**Task 2 — Lambda: log unsafe prompts to bad_prompts DynamoDB table**
- Function: `guardrail_chat_function`
- DIY section added inside `lambda_handler`, directly after `guard_result = moderate_input(prompt)` (before `generate_chat_response`)
- Code written:
  ```python
  if guard_result.startswith("unsafe"):
      dynamodb.put_item(
          TableName="bad_prompts",
          Item={
              "prompt_id": {"S": random_id},
              "prompt_content": {"S": prompt},
          },
      )
  ```
- `bad_prompts` partition key: `prompt_id (String)` — confirmed via DynamoDB console

**Key lesson — DIY block placement matters:**
First attempt placed the block after `generate_chat_response(prompt)`. That function also invokes Bedrock guardrails via `converse()` and can return/error before the DIY section runs. Fix: move `put_item` to immediately after the `moderate_input` call.

**Security group fix (blocker):**
- `streamlit-lb` was using the default VPC SG (`sg-07a14a27e7b5a513e`) which only allowed self-referencing traffic
- Added inbound rule: HTTP / port 80 / 0.0.0.0/0 to allow internet access to the Streamlit app

**Still pending:** CloudQuest lab grader still returning "No unsafe prompts stored" despite items present in `bad_prompts` table — may need Kiro CLI suggested format or a fresh test after re-deploy.

