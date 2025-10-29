# Automated Security Response with EventBridge & Lambda
Building Real-Time Threat Detection and Response for IAM Events in AWS

# Buisness Scenario
As a Security Engineer, you are tasked with reducing response time to suspicious IAM activity within our AWS environment. Previously, the security team manually investigated alerts,  a process prone to human error, inconsistent actions, and long response times (especially off-hours).

To solve this, I designed and implemented an automated security response system leveraging AWS EventBridge, Lambda, and SNS to detect and quarantine suspicious IAM user creation events in real time.

## By the end of this lab, I had:
- Developed Lambda functions for automated security responses
- Configured EventBridge rules to detect IAM CreateUser events
- Automated user quarantine via IAM policy enforcement
- Integrated SNS notifications for instant security alerts
- Tested and validated the full detection-to-response workflow

These steps demonstrate proficiency in AWS event-driven architecture, incident response automation, and cloud security engineering.

## Manual response has limitations:

- Delay: By the time someone sees an alert and responds, damage may already be done
- Consistency: Different people might respond differently to the same threat
- Coverage: Security teams can’t monitor 24/7 without automation
- Scale: During an active breach, there might be dozens of events happening simultaneously

My task is to build an automated security response system that can detect and immediately quarantine suspicious IAM user creation events.

# Lab Architecture
I’ll be building a system with these services:

- This system uses native AWS services for a fully managed, scalable, and serverless security automation pipeline:
- CloudTrail → Captures IAM API calls (CreateUser events)
- EventBridge Rule → Detects suspicious IAM activity
- Lambda Function → Executes automated quarantine and alert actions
- SNS Topic → Notifies the security team in real time
- IAM Policy → Applies deny-all permissions for quarantine enforcement

# Step 1: Subscribe to Security Alerts

- Navigate to SNS > Topics
- SNS Setup for Security Alerts
- Subscribed to the security-alerts topic for real-time notifications.
- Configured email-based subscriptions to ensure immediate visibility during incident response.
<img width="811" height="148" alt="image" src="https://github.com/user-attachments/assets/5732eb2f-24ad-4e6b-90bd-ad3e4c435fea" />

<img width="695" height="247" alt="image" src="https://github.com/user-attachments/assets/b9b3e36a-90da-45de-aeaf-41f8072906cb" />

<img width="618" height="198" alt="image" src="https://github.com/user-attachments/assets/26974379-4e64-42d7-9e97-5de0773f2529" />

Check my email and confirm the subscription 
<img width="538" height="213" alt="image" src="https://github.com/user-attachments/assets/f70f5b13-e9f3-4f76-8a23-b5c56dda2389" />

# Step 2: Create the Security Response Lambda Function
- Built a Python-based Lambda function to automatically:
- Parse incoming CloudTrail event data
- Quarantine newly created users by attaching a deny-all IAM policy
- Tag affected accounts with metadata (SecurityStatus: Quarantined)
- Send detailed SNS alerts with context and recommended actions

  ## Key Design Choices:
- Used environment variables for dynamic configuration (SNS topic ARN)
- Implemented error handling and idempotency for reliable execution
- Leveraged AWS-managed roles and permissions for least-privilege access

- Configure:
- Function name: security-response
- Runtime: Python 3.13
- Architecture: x86_64

<img width="626" height="298" alt="image" src="https://github.com/user-attachments/assets/db00b408-06b2-4e65-9316-f95845e498da" />

- Execution role: existing role

<img width="799" height="174" alt="image" src="https://github.com/user-attachments/assets/12eebc58-cc94-4c4c-969b-2f2d5d557553" />

- Create function

<img width="619" height="305" alt="image" src="https://github.com/user-attachments/assets/020cd01b-0b58-4206-985d-55f3254dfb97" />

<img width="794" height="366" alt="image" src="https://github.com/user-attachments/assets/69f730ec-0327-4029-adad-84c8d284bfc1" />

# The Security Response Code
 Check code file.

After pasting code in the code file, click on the “Deploy” button.
<img width="600" height="341" alt="image" src="https://github.com/user-attachments/assets/b28b41e9-2621-43c8-bca7-31609ef354be" />

# Step 3: Create the EventBridge Rule(EventBridge Rule Configuration)
Created a custom rule (detect-iam-createuser) to match:)

    {
      "detail-type": ["AWS API Call via CloudTrail"],
      "detail": {
        "eventSource": ["iam.amazonaws.com"],
        "eventName": ["CreateUser"]
      }
    }


Connected the rule to the Lambda target, enabling automated triggering on any IAM CreateUser event detected by CloudTrail.

Now you’ll create the rule that detects CreateUser events and triggers your Lambda function in EventBridge.

# EventBridge console > Rules (under “Buses”)

- Select the default event bus
- Create rule
- Configure:
- Name: detect-iam-createuser
- Description: Detects IAM CreateUser events for automated security response

Rule type: Rule with an event pattern

<img width="794" height="340" alt="image" src="https://github.com/user-attachments/assets/8737a062-e316-489a-aede-561d9d5584c5" />


<img width="637" height="341" alt="image" src="https://github.com/user-attachments/assets/4da0e5c7-43ef-4aa3-8fd5-ad428baa0fae" />

The key fields we care about for security automation:

- eventName: “CreateUser”
- eventSource: “iam.amazonaws.com”

In Target 1:
Target type: AWS service
Select a target: Lambda function
Function: Select your security-response function
Execution role: (default) 
Create rule

<img width="638" height="328" alt="image" src="https://github.com/user-attachments/assets/20c0376d-e1d9-47b9-a72c-874716f1d33f" />

<img width="797" height="317" alt="image" src="https://github.com/user-attachments/assets/10e3d51b-f1a0-4eb5-b408-66ae1d6a4ff2" />

# Add EventBridge Trigger to Lambda

<img width="662" height="386" alt="image" src="https://github.com/user-attachments/assets/4e985415-28af-4889-88fd-e66d870d2888" />



# Step 4: Test Your Automation
Time to see your security automation in action!

# Attack Simulation & Testing
Open CloudShell

Run the attack simulation:

    aws lambda invoke \
        --function-name $(aws lambda list-functions --query "Functions[?contains(FunctionName, 'attack-simulation')].FunctionName" --output text) \
        --payload '{}' \
        response.json && cat response.json

Response:

<img width="745" height="179" alt="image" src="https://github.com/user-attachments/assets/6c2db570-d6bb-48a0-89de-09301a69de72" />

     "StatusCode": 200,
        "ExecutedVersion": "$LATEST"
    }
    {"statusCode": 200, "body": "{\"message\": \"Attack simulation completed successfully\", \"username\": \"service-oekwze\", \"created_at\": \"2025-09-26T12:47:30.183373\"}"}~ $ 
    
Executed a pre-deployed “attack simulation” Lambda to mimic unauthorized user creation.

- Observed the following automated chain:
- CloudTrail captured the event
- EventBridge matched and triggered Lambda
- Lambda quarantined the user via IAM policy
- SNS sent detailed incident notification

# Step 5: Verification:

-Confirmed user quarantine status in IAM
- Validated log entries in CloudWatch
- Received contextualized email alerts for the incident

# Check if the Suspicious User was Created
Navigate to IAM > Users

<img width="659" height="200" alt="image" src="https://github.com/user-attachments/assets/b8b20dfd-287e-48c2-abcc-607137d0462a" />

<img width="464" height="359" alt="image" src="https://github.com/user-attachments/assets/29dd5e85-2247-47f8-ac23-5955e8a42258" />

# Results & Impact

- Response Time: Reduced from minutes/hours to under 2 minutes (post-CloudTrail event delivery).
- Human Error: Eliminated inconsistencies in manual response workflows.
- Operational Coverage: Achieved 24/7 automated detection and containment.
- Scalability: Solution works across multiple accounts and can easily extend to other AWS security events (e.g., unauthorized API calls, access key creation, or S3 public ACL changes).

# Troubleshooting Insights

While implementing the solution, I diagnosed and resolved common AWS event-driven integration issues, including:
- EventBridge rule pattern mismatches
- Lambda invocation permission errors
- Role-based execution failures between services

This reinforced my ability to debug distributed AWS systems and ensure robust, production-ready automation.

<img width="764" height="233" alt="image" src="https://github.com/user-attachments/assets/e8eaba4f-299f-45a8-b0cc-c841c644bdeb" />

The above metrics showed that EventBridge successfully detected the user creation and triggered my Lambda function. 

# Check the Lambda Function Logs
- The EventBridge event that triggered the function
- Details about the user creation event
- Confirmation that the quarantine policy was applied
- Success message for the SNS notification

<img width="549" height="318" alt="image" src="https://github.com/user-attachments/assets/3062c4ca-3554-4293-bdce-72b4ecd76a96" />

Check if user was Quarantine

<img width="638" height="137" alt="image" src="https://github.com/user-attachments/assets/570ce004-2525-4782-9daa-f8ada6c40870" />

<img width="657" height="150" alt="image" src="https://github.com/user-attachments/assets/459468a8-74e9-40ec-b8e4-c5d86e291894" />


Check Email

<img width="648" height="380" alt="image" src="https://github.com/user-attachments/assets/d821cc49-1d2a-4e1f-b405-a592416e5455" />

# Conclusion

This project showcases my ability to design, implement, and operationalize automated cloud security responses using native AWS services.

## It highlights a strong understanding of:

- Event-driven architectures
- Security orchestration and response (SOAR)
- Serverless automation pipelines
- Operational excellence in cloud environments

By transforming manual incident handling into a real-time, automated defense mechanism, I demonstrated both technical depth and the ability to deliver business impact through scalable security automation.

# Infrastructure as Code (IaC)

After completing the lab in the AWS Console, I developed a Terraform template to automate deployment across environments (Check my repo).
This template encapsulates:
- EventBridge rule creation
- Lambda deployment
- IAM roles and policies
- SNS topic and subscriptions

The IaC approach enables consistent, version-controlled, and auditable security automation deployments.










     



