# Automated-Security-Response-with-EventBridge-and-Lambda
 Build an automated security response system that can detect and immediately quarantine suspicious IAM user creation events.
# Buisness Scenario
You're a security engineer who has been tasked with implementing automated incident response for your AWS environment. The security team has been manually responding to suspicious IAM activity which is time consuming and error prone. Your mission is to build an automated security response system that can detect and immediately quarantine suspicious IAM user creation events.
# Overview & Scenario
A security engineer who just implemented CloudWatch monitoring for IAM activities from our prior lab.

The dashboards are working great, but there’s a problem: you’re still relying on humans to see the alerts and respond. What happens at 2 AM when a threat actor starts creating users? What about during weekends or holidays?

Manual response has limitations:

• Delay: By the time someone sees an alert and responds, damage may already be done

• Consistency: Different people might respond differently to the same threat

• Coverage: Security teams can’t monitor 24/7 without automation

• Scale: During an active breach, there might be dozens of events happening simultaneously

Your mission is to build an automated security response system that can detect and immediately quarantine suspicious IAM user creation events.

# Learning Objectives
By the end of this lab, I was able to:

• Create Lambda functions for automated security responses

• Configure EventBridge rules to detect specific security events

• Implement user quarantine through IAM policy automation

• Set up SNS notifications for security incident alerting

• Test and validate automated security response workflows

# Lab Architecture
I’ll be building a system with these services:

• EventBridge Rule: Detects IAM CreateUser events from CloudTrail

• Lambda Function: Automatically quarantines suspicious users and sends alerts

• SNS Topic: Sends notifications about security events and responses

• CloudTrail: Captures API events for EventBridge to process (pre-deployed)

• Attack Simulation: A Lambda function to generate test events (pre-deployed)

# Step 1: Subscribe to Security Alerts
First, let’s set up notifications so you can see when your automation triggers.

• Navigate to SNS > Topics

• Click on the topic named security-alerts
<img width="811" height="148" alt="image" src="https://github.com/user-attachments/assets/5732eb2f-24ad-4e6b-90bd-ad3e4c435fea" />

• Grab the ARN value and store it in your notes – we’ll need it later

• Click Create subscription

•Configure:

•Protocol: Email

• Endpoint: Enter your email address

<img width="695" height="247" alt="image" src="https://github.com/user-attachments/assets/b9b3e36a-90da-45de-aeaf-41f8072906cb" />


• Click Create subscription
<img width="618" height="198" alt="image" src="https://github.com/user-attachments/assets/26974379-4e64-42d7-9e97-5de0773f2529" />

Check your email and confirm the subscription -> if you don’t do this you will not receive notifications. Note that it can take a minute to receive the confirmation email from AWS.

<img width="538" height="213" alt="image" src="https://github.com/user-attachments/assets/f70f5b13-e9f3-4f76-8a23-b5c56dda2389" />

# Step 2: Create the Security Response Lambda Function

Now you’ll create the Lambda function that automatically responds to security events.

# Deploy the Lambda Function

Navigate to Lambda > Functions (ignore the existing attack-simulation function for now)

•Click Create function

• Configure:
• Function name: security-response (please use this exact name or you will experience permissions issues in our lab)
• Runtime: Python 3.13
• Architecture: x86_64

<img width="626" height="298" alt="image" src="https://github.com/user-attachments/assets/db00b408-06b2-4e65-9316-f95845e498da" />

• Execution role: “Change default execution role” -> Use an existing role -> security-response-lambda-role (if you don’t select this role it will fail to create)

<img width="799" height="174" alt="image" src="https://github.com/user-attachments/assets/12eebc58-cc94-4c4c-969b-2f2d5d557553" />

• Click Create function

<img width="619" height="305" alt="image" src="https://github.com/user-attachments/assets/020cd01b-0b58-4206-985d-55f3254dfb97" />

You should see a code editor under the Code tab that has the following, and this is what we’re going to entirely replace:
import json

    def lambda_handler(event, context):
        # TODO implement
        return {
            'statusCode': 200,
            'body': json.dumps('Hello from Lambda!')
        }

<img width="596" height="207" alt="image" src="https://github.com/user-attachments/assets/d391142b-35fc-4ed8-a5df-db457ec15d73" />

# The Security Response Code
Here’s the Python code for the security response function: check code file.

After pasting code in the code file, click on the “Deploy” button.
<img width="600" height="341" alt="image" src="https://github.com/user-attachments/assets/b28b41e9-2621-43c8-bca7-31609ef354be" />

# Understanding the Code
This function performs four key actions:

Event Parsing: Extracts details from the EventBridge event containing CloudTrail data

User Quarantine: Attaches a pre-deployed deny-all IAM policy to prevent any actions by the suspicious user

User Tagging: Adds metadata tags to track the quarantine action and timing

Alert Notification: Sends detailed information to the security team via SNS

The quarantine policy (AutomatedSecurityQuarantine) has been pre-deployed for you. This simulates deploying a security quarantine policy to every AWS account within our organization that can 

then be used by automation scripts, and that can be managed from a central location via IaC. It would look something like this:

The quarantine policy (AutomatedSecurityQuarantine) has been pre-deployed in this lab. This simulates deploying a security quarantine policy to every AWS account within our organization that can then be used by automation scripts, and that can be managed from a central location via IaC. It would look something like this:

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Sid": "AutomatedSecurityQuarantine",
          "Effect": "Deny",
          "Action": "*",
          "Resource": "*"
        }
      ]
    }



     



