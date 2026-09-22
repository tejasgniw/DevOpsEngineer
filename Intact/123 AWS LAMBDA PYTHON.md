# Lambda

## 1. AWS Lambda — 15-minute crash prep

Lambda = run code without managing servers.

```
Code
   ↓
Lambda Function
   ↓
AWS invokes it
   ↓
Execution Environment
   ↓
Your code runs
   ↓
Result / logs / downstream action
```

Typical DevOps use cases:

- S3 event → Lambda
- API Gateway → Lambda
- EventBridge → Lambda
- CloudWatch/EventBridge scheduled Lambda
- SQS → Lambda
- Step Functions → Lambda
- Automation/remediation
- Infrastructure operations
- Data processing


## 2. Lambda concepts you absolutely need

Runtime: Python is one of the Lambda runtimes.

```
def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": "Hello World"
    }
```

- event: Input received by Lambda.

e.g.

```
def lambda_handler(event, context):
    bucket = event["Records"][0]["s3"]["bucket"]["name"]
```

- context: Information about the invocation/runtime.

e.g.

```
context.function_name
context.memory_limit_in_mb
context.aws_request_id
```


## 3. Lambda execution role — VERY important

Lambda execution role = IAM role assumed by Lambda.

```
Lambda
   |
   | assumes
   ↓
IAM Execution Role
   |
   ├── s3:GetObject
   ├── s3:PutObject
   └── logs:CreateLogGroup
```

If Lambda needs to read S3, you don't put AWS credentials inside the Python code. You give Lambda an IAM execution role with the required permissions.

### Interview answer

**For Lambda, I use an IAM execution role with least-privilege permissions rather than embedding credentials in the function. The role would contain only the permissions required for the function, such as reading from S3 or writing logs.**


## 4. Lambda environment variables
Used for configuration:

```
ENVIRONMENT=prod
BUCKET_NAME=my-bucket
API_URL=https://example.com
```

Python:

```
import os

bucket = os.environ["BUCKET_NAME"]
```

Don't put secrets directly into environment variables unless appropriate controls are in place.

For sensitive values, think: **AWS Secrets Manager**


## 5. Lambda timeout + memory

- Timeout: Maximum execution duration.

If your Lambda takes longer than the configured timeout:

```
Lambda starts
     ↓
code executes
     ↓
timeout reached
     ↓
Lambda terminates
```

- Memory: Memory allocation also influences available CPU/network performance.

### Lambda is timing out. What do you check?

```
1. CloudWatch logs
2. Duration
3. Timeout configuration
4. Downstream dependencies
5. Network connectivity
6. VPC configuration
7. DNS
8. API/database latency
9. Memory allocation
10. Retries / concurrency
```


## 6. Lambda + VPC

Suppose Lambda needs to access a private RDS database.

```
Lambda
   |
   ↓
VPC
 ┌───────────────┐
 │ Private subnet│
 │      ↓        │
 │     RDS       │
 └───────────────┘
```

You need to think about:

- Subnets
- Security groups
- Route tables
- DNS
- NAT Gateway if outbound internet access is required
- Network ACLs where applicable

### Lambda worked before, but after putting it inside a VPC it cannot reach an external API. What do you check?

**I'd first verify whether the Lambda actually needs VPC access. If it does, I'd check its subnet routing and whether it has a route through a NAT Gateway for outbound internet access. I'd also check security groups, DNS resolution and the destination endpoint.**


## 7. Cold starts

An AWS Lambda cold start refers to the initial delay (latency) that occurs when a Lambda function is executed after being idle, during traffic spikes, or after code changes.

When Lambda needs a new execution environment:

```
Request
  ↓
Create execution environment
  ↓
Initialize runtime
  ↓
Load dependencies
  ↓
Execute function
```

That initialization can add latency.

**Ways to reduce cold-start impact**:

- Keep deployment packages small
- Reduce unnecessary dependencies
- Optimize initialization code
- Appropriate runtime choice
- Provisioned Concurrency when low latency is important


## 8. Lambda concurrency

```
1 request → 1 execution

100 simultaneous requests
        ↓
multiple Lambda executions
```

Lambda scales execution environments based on concurrency.

- Reserved concurrency: Limits/constrains concurrency for a function and can protect downstream systems.

- Provisioned concurrency: Keeps execution environments initialized to reduce cold-start latency or keep environments warm for predictable low latency.


## 9. Lambda retries

Different invocation models behave differently.

- Synchronous invocation

```
Synchronous
API Gateway → Lambda
```

Caller generally handles the response/error.

- Asynchronous invocation:

```
EventBridge → Lambda
             ↓
          failure
             ↓
          retry
```

### Interview answer

**I always consider the invocation model because retry and failure behavior depends on whether the invocation is synchronous, asynchronous, or through an event source mapping.**


## 10. Lambda monitoring

CloudWatch Logs: goes into CloudWatch Logs.

```
print()
logging
```

Important Lambda metrics:

- Invocations
- Errors
- Duration
- Throttles
- Concurrent executions
- Iterator age for relevant stream-based workloads

### Interview answer

**I would create CloudWatch alarms around errors, duration, throttling and other business-relevant indicators.**


## 11. Lambda deployment

```
Source code
    ↓
Package / build
    ↓
CI/CD
    ↓
Lambda
    ↓
Version
    ↓
Alias
```

Lambda supports:

- Versions
- Aliases
- Layers
- Deployment packages
- Container images

For production, you can use deployment strategies such as:

```
Version 1
   ↓
Alias
   ↓
Production
```

Then deploy:

```
Version 2
   ↓
Test
   ↓
Gradually shift traffic
```

This gives you rollback capability.


## 12. Terraform + Lambda

Since Terraform is your strength, connect the two.

```
resource "aws_lambda_function" "example" {
  function_name = "my-function"
  runtime       = "python3.x"
  handler       = "lambda_function.lambda_handler"
  role          = aws_iam_role.lambda_role.arn
}
```

### Interview answer

**I would manage the Lambda function, IAM role, environment configuration, networking and supporting resources through Terraform so that the configuration is version-controlled and repeatable across environments.**



# Python

## 13. Basics

### Variables

```
name = "prod"
count = 10
enabled = True
```

### Lists

```
servers = ["web1", "web2", "web3"]

for server in servers:
    print(server)
```

### Dictionaries

```
server = {
    "name": "web1",
    "environment": "prod",
    "status": "running"
}

print(server["name"])
```

### Functions

```
def restart_service(service):
    print(f"Restarting {service}")

restart_service("nginx")
```

### Exception handling

```
try:
    result = do_something()
except Exception as e:
    print(f"Error: {e}")
```

Better:

```
try:
    result = do_something()
except FileNotFoundError:
    print("File not found")
```

Don't blindly catch every exception if you can handle specific exceptions.


## 14. JSON — extremely important for AWS

```
import json

data = json.loads('{"name": "app", "env": "prod"}')

print(data["name"])
```

### Convert Python → JSON:

```
json.dumps(data)
```

Lambda events are commonly represented as Python dictionaries after the event is passed to the handler.


## 15. Environment variables

```
import os

environment = os.getenv("ENVIRONMENT", "dev")
```


## 16. Calling APIs

### How would you automate an API call with Python?

```
import requests

response = requests.get(
    "https://api.example.com/resources",
    timeout=10
)

response.raise_for_status()

data = response.json()
```

Know:

```
response.status_code
response.json()
response.raise_for_status()
```

**I would always use a timeout and handle exceptions rather than allowing the script to hang indefinitely.**


## 17. Running system commands

```
import subprocess

result = subprocess.run(
    ["kubectl", "get", "pods"],
    capture_output=True,
    text=True,
    check=True
)

print(result.stdout)
```

**It allows Python automation to interact with system commands while giving me control over stdout, stderr, return codes and failures.**


## 19. Five Python questions to rehearse

### Q1. List vs dictionary?

**A list is an ordered collection accessed by index, while a dictionary stores key-value pairs and is useful when I need to access data by a meaningful key.**

### Q2. What is exception handling?

**It's a way to handle runtime errors gracefully using try/except, so an automation script can fail predictably, log the issue and take appropriate recovery actions.**

### Q3. How do you read an environment variable?

```
import os

value = os.getenv("MY_VARIABLE")
```

### Q4. How do you call an API?

**I'd typically use the requests library, set a timeout, check the HTTP status, parse the response, and handle exceptions.**

### Q5. How do you automate a shell command using Python?

**I'd use subprocess, capture stdout/stderr, check the return code, and handle failures appropriately.**


## 20. The most important combined question

### How would you build a Python Lambda that processes files uploaded to S3?

```
S3 upload
   ↓
S3 Event
   ↓
Lambda
   ↓
Python
   ↓
Process file
   ↓
Output → S3 / database / downstream service
```

**I would configure an S3 event notification to trigger the Lambda. The Python handler would receive the event, extract the bucket and object key, retrieve the object using the Lambda execution role, process it, and write the result to the required destination.**

**I would manage the Lambda, IAM role, S3 configuration and supporting infrastructure through Terraform. I'd use CloudWatch Logs and metrics for monitoring and configure appropriate error handling and retries. For production, I'd also consider idempotency, permissions, timeout, memory, concurrency and failure handling.**


## 21. Rapid Lambda interview questions

Before your interview, say these answers out loud.

### "Lambda is timing out. What do you do?"

**Check CloudWatch logs and duration, identify where it's spending time, check downstream dependencies and networking, verify timeout/memory, and investigate VPC/DNS/security-group issues if applicable.**

### "Lambda can't access S3."

**Check the Lambda execution role and S3 IAM permissions, bucket policy, encryption/KMS permissions if applicable, and verify we're accessing the correct bucket/region.**

### "Lambda can't access an external API after being placed in a VPC."

**Check subnet routing, NAT Gateway, security groups, DNS and network connectivity.**

### "How do you secure Lambda?"

**Least-privilege IAM, encryption, Secrets Manager/SSM for sensitive configuration, VPC where required, controlled deployment permissions, logging/monitoring and avoiding credentials in code.**

### "How do you deploy Lambda with Terraform?"

**Manage the function, IAM role, configuration, networking and supporting resources through Terraform, keep the configuration in Git, and deploy through CI/CD.**

### "How do you troubleshoot Lambda?"

Use:

Logs → Metrics → Configuration → IAM → Network → Dependencies → Recent changes

Memorize that sequence.


## Your honest positioning if they ask about your Lambda depth

### How much Lambda experience do you have?

**I've worked with Lambda as part of AWS cloud and infrastructure platforms, including integrating it with services such as S3 and Step Functions and managing the supporting infrastructure through Terraform. My strongest area is the DevOps and cloud side rather than being a dedicated Lambda application developer. However, I'm comfortable with Lambda architecture, IAM, networking, configuration, monitoring, deployment and troubleshooting, and I understand the Python programming model.**
