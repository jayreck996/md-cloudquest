ASSET LOG

####### <!-- ANCHOR MARKER - ADD ALL NEW ASSET ENTRIES DIRECTLY BELOW THIS LINE -->

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

