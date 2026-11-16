ASSET LOG

####### <!-- ANCHOR MARKER - ADD ALL NEW ASSET ENTRIES DIRECTLY BELOW THIS LINE -->

## ASSET:cloudquest 2026-11-16 → CloudFormation: S3 URL auto-population and Infrastructure Composer

**Source:** CloudFormation console deployment flow

**S3 URL auto-population:** When you upload a template through the CloudFormation console, it automatically copies the file to an S3 bucket in your account and fills in the S3 URL field — you don't need to manually construct the path. CloudFormation keeps this as a persistent copy throughout the stack's lifetime, using it as the source of truth for drift detection, updates, and rollbacks.

**Why the persistent S3 copy matters:** CloudFormation doesn't read the template once and discard it. It references that stored copy whenever it needs to compare the real state of your resources against the original definition — for drift detection, stack updates, and rollback scenarios.

**Infrastructure Composer:** The visual drag-and-drop designer built into the CloudFormation console. Displays your template as a diagram of connected AWS resources rather than raw YAML. The Edit in Infrastructure Composer button remains available throughout the deployment flow — you can return to the visual designer to fix a misconfigured property or tweak a resource right up until the moment you commit to creating the stack, without losing your place in the deployment wizard.

---

## ASSET:cloudquest 2026-11-16 → CloudFormation: rollback on failure behavior

**Source:** CloudFormation stack deployment behavior

**What it does:** CloudFormation deployments are all-or-nothing. If any single resource fails during stack creation — due to a syntax error, a timeout, or insufficient IAM permissions — CloudFormation automatically deletes every resource it successfully created before the failure. The environment is left exactly as it was before the deployment started.

**Why it matters:** Without automatic rollback, a failed deployment can leave behind orphaned resources (VPCs, subnets, load balancers, security groups) that are disconnected from anything useful but still incur costs or create open network paths. Finding and removing them manually is error-prone.

**Common failure causes:**
- Syntax errors making a property value invalid
- A resource timing out waiting to become available
- The deployment IAM role missing a required permission

**Disabling rollback for troubleshooting:** Rollback destroys the partial deployment state, removing the evidence needed to diagnose the failure. Disabling rollback before a deployment you expect may fail lets you inspect exactly what was created and what state each resource was in at the point of failure. After diagnosing, manually delete the partial stack and re-deploy with rollback re-enabled.

---
