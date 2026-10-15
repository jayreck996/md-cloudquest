ASSET LOG

####### <!-- ANCHOR MARKER - ADD ALL NEW ASSET ENTRIES DIRECTLY BELOW THIS LINE -->

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

