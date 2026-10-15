ASSET LOG

####### <!-- ANCHOR MARKER - ADD ALL NEW ASSET ENTRIES DIRECTLY BELOW THIS LINE -->

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

