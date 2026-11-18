ASSET LOG

####### <!-- ANCHOR MARKER - ADD ALL NEW ASSET ENTRIES DIRECTLY BELOW THIS LINE -->

## ASSET:cloudquest 2026-11-19 → SSM Session Manager: shell access without SSH

**Source:** AWS Systems Manager Session Manager

SSM Session Manager provides interactive shell access to EC2 instances over HTTPS without opening port 22 or managing SSH key pairs. The SSM Agent installed on the EC2 instance makes an outbound WebSocket connection to the SSM service endpoint on port 443. Because the agent initiates the outbound connection, no inbound security group rule is needed on the instance.

When a session is started from the AWS CLI or console, commands travel from the client over HTTPS to the SSM endpoint, which relays them to the EC2 agent through the existing WebSocket channel. The protocol used for the shell session is WebSocket over HTTPS — SSH is not involved at any point in the flow.

```
Client ──HTTPS 443──► SSM endpoint (ssm.us-east-1.amazonaws.com)
                              │
                      AWS internal backbone
                              │
                       SSM Agent on EC2
                   (outbound WebSocket, no inbound port)
```

Authentication is IAM-based — the IAM identity of the person running the session determines whether access is permitted, replacing SSH key pair management entirely. If the SSM Agent stops or the instance loses outbound connectivity to the SSM endpoint, the channel drops and there is no fallback path unless port 22 is separately configured.

---

## ASSET:cloudquest 2026-11-19 → API Gateway: Layer 7 routing, resource/method segregation, Lambda proxy

**Source:** AWS API Gateway architecture discussions

API Gateway operates at OSI Layer 7. It receives HTTP/HTTPS requests and routes them based on application-layer content — URL path, HTTP method, headers, and query parameters. This distinguishes it from a Network Load Balancer (Layer 4), which routes by port number only and has no concept of HTTP paths or methods.

Each resource (path segment) and method (HTTP verb) combination is an independent routing unit with its own backend integration. The same path can route differently depending on the method — `GET /users` and `POST /users` can point to entirely different Lambda functions or services.

```
GET  /users        → Lambda: GetUsersFunction
POST /users        → Lambda: CreateUserFunction
GET  /orders       → HTTP integration → ECS service
POST /orders       → Lambda: CreateOrderFunction
```

Lambda proxy integration passes the complete HTTP request — headers, query parameters, path parameters, and body — to the Lambda function as a single JSON event without transformation. The function receives one structured object and must return a response in the specific format API Gateway expects. This is still Layer 7 work: constructing the JSON event requires fully parsing the HTTP request, which only Layer 7 can do.

API Gateway and Lambda are both Layer 7 services. Handing off between them does not break the OSI model — each hop between services traverses the full OSI stack independently. API Gateway → Lambda is an internal AWS invocation over the AWS backbone with its own TCP/IP/TLS underneath, resetting the stack for that leg of the journey.

---

## ASSET:cloudquest 2026-11-17 → S3 Static Website and CloudFront: TLS termination and two-connection model

**Source:** AWS CloudFront + S3 architecture

S3 website endpoint (`mybucket.s3-website-us-east-1.amazonaws.com`) only supports HTTP — TLS is not available on that endpoint. A client accessing S3 directly receives an unencrypted HTTP response. The S3 REST API endpoint supports HTTPS, but cannot serve a website with clean path routing.

CloudFront adds TLS by acting as the HTTPS termination point. The viewer connects to the CloudFront edge over HTTPS (port 443), and CloudFront connects to S3 over HTTP internally using the AWS backbone. These are two independent TCP connections — the viewer sees HTTPS end-to-end, but the CloudFront-to-S3 leg never crosses the public internet and uses HTTP.

```
Viewer ──HTTPS──► CloudFront edge (TLS terminates here, ACM cert)
                        │
                    HTTP internal
                        │
                   S3 website endpoint (HTTP only)
```

The ACM certificate used by CloudFront must be provisioned in `us-east-1` regardless of which region the CloudFront distribution or S3 bucket is in — CloudFront's certificate infrastructure is global but anchored in us-east-1.

OAC (Origin Access Control) restricts the S3 bucket so it only accepts requests from CloudFront. Without OAC, the S3 bucket URL remains publicly accessible, bypassing CloudFront's access controls and cache.

---

## ASSET:cloudquest 2026-11-17 → CloudFront domains: default domain vs CNAME branded domain

**Source:** AWS CloudFront distribution configuration

Every CloudFront distribution is assigned a default domain automatically — a subdomain of cloudfront.net in the format `d111111abcdef8.cloudfront.net`. This domain works immediately with no DNS configuration, uses an AWS-managed TLS certificate, and requires no ACM setup.

A CNAME branded domain (such as `cdn.example.com`) uses a DNS CNAME record to point the custom hostname to the CloudFront default domain. When a browser resolves `cdn.example.com`, DNS returns the CloudFront domain as an alias, which then resolves to a CloudFront edge IP. The browser makes the final HTTPS connection to that IP and presents `cdn.example.com` as the SNI hostname.

```
Browser resolves cdn.example.com
  → DNS CNAME: cdn.example.com → d111111abcdef8.cloudfront.net
  → DNS A record: d111111abcdef8.cloudfront.net → 13.32.0.1
  → HTTPS connection to 13.32.0.1, SNI: cdn.example.com
```

A CNAME branded domain requires an ACM certificate in us-east-1 that covers the custom hostname. The certificate must list the exact hostname presented in SNI. Without it, CloudFront cannot serve TLS for the branded domain.

---

## ASSET:cloudquest 2026-11-17 → CloudFront access control: signed URLs vs signed cookies

**Source:** AWS CloudFront signed URL and signed cookie mechanics

Both signed URLs and signed cookies restrict CloudFront content to authorised requests. The difference is how the credential is delivered and how broadly it applies.

A signed URL embeds the authorisation directly in the query string of a single URL — the signature, credential, and expiry are parameters appended to the path. Each signed URL grants access to exactly one object for a limited time. It is suited to individual asset downloads where each URL is generated per request.

A signed cookie delivers three separate cookies to the browser — `CloudFront-Policy`, `CloudFront-Signature`, and `CloudFront-Key-Pair-Id`. Once set, the browser sends these cookies on every subsequent request matching the cookie's scope (such as `/premium/*`). A single login event grants access to an entire path namespace for the duration of the cookie expiry. It is suited to streaming services or sections of a site where many assets share the same access policy.

```
Signed URL:    auth in query string → one object, one URL
Signed cookie: auth in browser cookies → entire /premium/* path
```

Both are issued by the application server using a CloudFront key pair and validated by CloudFront at the edge before serving. Neither requires any compute call per request once the credential is issued — validation is performed at the CDN layer.

---

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
