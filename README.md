# Automated-Security-Response-with-EventBridge-and-Lambda
 Build an automated security response system that can detect and immediately quarantine suspicious IAM user creation events.
# Buisness Scenario
You're a security engineer who has been tasked with implementing automated incident response for your AWS environment. The security team has been manually responding to suspicious IAM activity which is time consuming and error prone. Your mission is to build an automated security response system that can detect and immediately quarantine suspicious IAM user creation events.
# Overview & Scenario
A security engineer who just implemented CloudWatch monitoring for IAM activities from our prior lab.

The dashboards are working great, but there’s a problem: you’re still relying on humans to see the alerts and respond. What happens at 2 AM when a threat actor starts creating users? What about during weekends or holidays?

## Manual response has limitations:

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

<img width="794" height="366" alt="image" src="https://github.com/user-attachments/assets/69f730ec-0327-4029-adad-84c8d284bfc1" />

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

This policy denies all actions on all resources, effectively quarantining the user until an administrator can investigate.

Lambda function creation continued
Go to the Configuration tab > and then Environment variables
Click Edit and add one environment variable:
Key: SNS_TOPIC_ARN
Value: You should have this value in your notes from earlier, but if not get this from SNS > Topics (copy the ARN of your security alerts topic)

<img width="818" height="170" alt="image" src="https://github.com/user-attachments/assets/9efba63c-19f0-4eef-9a38-1635ffc81a1a" />

What this is doing is attaching an environment variable to the Lambda function so that your code can access this key/value pair whenever it needs it.

Environment variables are super helpful and commonly used, but never ever store secret values here since they are easily accessible and in plaintext. Instead, we’d want to use something like Secrets Manager or Parameter Store in SSM for secret values.

# Step 3: Create the EventBridge Rule

Now you’ll create the rule that detects CreateUser events and triggers your Lambda function in EventBridge.

Navigate to EventBridge > Rules (under “Buses”)

Select the default event bus (if it’s not already selected)

Click Create rule

Configure:

Name: detect-iam-createuser

Description: Detects IAM CreateUser events for automated security response

Make sure to “Enable the rule on the selected event bus” if it’s not already toggled

Rule type: Rule with an event pattern

<img width="794" height="340" alt="image" src="https://github.com/user-attachments/assets/8737a062-e316-489a-aede-561d9d5584c5" />


Now we need to build our event pattern.

For the event source, we want “AWS events or EventBridge partner events“”
For the Event pattern section, select “Custom pattern (JSON editor)“”
Paste this event pattern:

<img width="637" height="341" alt="image" src="https://github.com/user-attachments/assets/4da0e5c7-43ef-4aa3-8fd5-ad428baa0fae" />

By the way, we’re not just randomly coming up with this pattern. When someone creates an IAM user, CloudTrail captures this event and this is what a CreateUser event looks like:

<img width="811" height="296" alt="image" src="https://github.com/user-attachments/assets/452103fd-d41b-4bce-9ab0-da2cc5fddfe3" />

The key fields we care about for security automation:

eventName: “CreateUser”
eventSource: “iam.amazonaws.com”

So that’s what the EventBridge pattern needs to capture.

Alright, let’s click on Next.

In Target 1:
Target type: AWS service
Select a target: Lambda function
Function: Select your security-response function
Execution role: Leave as “Create a new role for this resource” (default) – this will let AWS automatically create a service role with the correct permissions for EventBridge to invoke your Lambda function

Click Skip to Review and create
Click Create rule
<img width="638" height="328" alt="image" src="https://github.com/user-attachments/assets/20c0376d-e1d9-47b9-a72c-874716f1d33f" />

<img width="797" height="317" alt="image" src="https://github.com/user-attachments/assets/10e3d51b-f1a0-4eb5-b408-66ae1d6a4ff2" />

# Add EventBridge Trigger to Lambda
When using an existing execution role, you might need to manually add the trigger to the Lambda function:

Navigate back to Lambda > Functions > security-response
Click Add trigger (either in the diagram or under Configuration -> Triggers)
Configure the trigger:
Trigger configuration: Set EventBridge (CloudWatch Events) as the source
Rule: Select detect-iam-createuser (the rule you just created)
Click Add

<img width="662" height="386" alt="image" src="https://github.com/user-attachments/assets/4e985415-28af-4889-88fd-e66d870d2888" />

You should now see the EventBridge trigger listed in your Lambda function’s triggers section.

Also, if you go to Configuration -> Permissions and scroll down until you get to “Resource-based policy statements” you will now see a resource-based policy allowing the EventBridge rule to trigger this function. That plus the execution role from earlier are what grants permission.

<img width="781" height="363" alt="image" src="https://github.com/user-attachments/assets/e228b56b-98e2-4ce1-bc63-7684d94220c5" />

After that, we’re ready to move on!

# Step 4: Test Your Automation
Time to see your security automation in action!

# Run the Attack Simulation
Open CloudShell (icon in the top of the AWS Console near search)

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
    
This command will execute the attack Lambda function that was pre-deployed in the environment and that we saw in the console earlier. This function will create a suspicious IAM user, which should trigger your automation within 5-15 minutes (due to CloudTrail delivery delay to EventBridge).

Remember that CloudTrail is not real-time, which means we could wait up to 15 minutes for the workflow to execute, though usually it’s only a couple of minutes. While we wait, feel free to proceed with the lab guide to make sure everything worked properly. Go ahead and close CloudShell too as we no longer need it.


# Step 5: Verify Your Automation Worked
Note: CloudTrail events typically take 5-15 minutes to be delivered to EventBridge, so be patient! Let’s check what happened by following the automation chain:

Check if the Suspicious User was Created
Navigate to IAM > Users
Look for a user with a suspicious name (like admin-abc123, backup-def456, system-ghi789, etc…)

# Check if the Suspicious User was Created
Navigate to IAM > Users

<img width="659" height="200" alt="image" src="https://github.com/user-attachments/assets/b8b20dfd-287e-48c2-abcc-607137d0462a" />

<img width="464" height="359" alt="image" src="https://github.com/user-attachments/assets/29dd5e85-2247-47f8-ac23-5955e8a42258" />


If you see the user, great! The attack simulation worked. We’ll come back to this, but now let’s see if your security automation responded.

# Check EventBridge Rule Activity
Navigate to EventBridge > Rules
Click on your detect-iam-createuser rule
Go to the Metrics tab
Look for:
MatchedEvents: Should show 1 or more matches
SuccessfulInvocations: Should show 1 or more successful triggers
Invocations: Should show the rule was triggered

<img width="764" height="233" alt="image" src="https://github.com/user-attachments/assets/e8eaba4f-299f-45a8-b0cc-c841c644bdeb" />

If you see these metrics, EventBridge successfully detected the user creation and triggered your Lambda function. If not, give it another couple of minutes and refresh. You should eventually see the metrics, and once you do, proceed to step 3.

# Check the Lambda Function Logs
Navigate to Lambda > Functions > security-response
Go to Monitor and review the metrics graphcs
Then, click on View CloudWatch Logs
Click on the most recent log stream (the first one)

You should see:

The EventBridge event that triggered the function
Details about the user creation event
Confirmation that the quarantine policy was applied
Success message for the SNS notification

<img width="549" height="318" alt="image" src="https://github.com/user-attachments/assets/3062c4ca-3554-4293-bdce-72b4ecd76a96" />

Check if the User was Quarantined
Go back to the suspicious user you found in Step 1:

Click on the user in IAM
Check the Permissions tab – you should see the AutomatedSecurityQuarantine managed policy attached
Check the Tags tab – you should see quarantine-related tags like:
SecurityStatus: Quarantined
QuarantineReason: Automated response to suspicious user creation
QuarantineTime: timestamp (notice the time difference between both timestamps)

<img width="638" height="137" alt="image" src="https://github.com/user-attachments/assets/570ce004-2525-4782-9daa-f8ada6c40870" />

<img width="657" height="150" alt="image" src="https://github.com/user-attachments/assets/459468a8-74e9-40ec-b8e4-c5d86e291894" />

If you see these, your Lambda function successfully quarantined the user!

Check Your Email
You should receive a detailed security alert with:

Information about the suspicious user creation
Details about the automated response
Next steps for investigation

<img width="648" height="380" alt="image" src="https://github.com/user-attachments/assets/d821cc49-1d2a-4e1f-b405-a592416e5455" />

What’s really nice about this alert is that it a) removes all the noise and only focuses on the threat, and b) gives you step-by-step what you should do. You could even link to an internal playbook in your docs, for example.

# Recapping What Just Happened
Your automation system worked like this:

Attack Simulation → Created a suspicious IAM user
CloudTrail → Captured the CreateUser API call
EventBridge → Detected the event pattern and triggered your rule
Lambda Function → Received the event and executed your security response
IAM Policy → Applied quarantine to prevent user actions
SNS → Sent alert to your email
This entire process happens automatically once CloudTrail events reach EventBridge (typically 5-15 minutes), providing rapid containment of potential threats.

# Troubleshooting EventBridge issues
As we start to wrap up, I want to share a few tips/tricks if you try to apply EventBridge in your own environments and encounter issues:

EventBridge shows no matches: a) Wait longer (up to 15 minutes) for CloudTrail events to show up, or b) make sure your event pattern is correct. Even just a single typo or something missing will lead to this issue
User created but not quarantined: a) Make sure the Lambda function is executing. If not, read on to the next one. b) Check Lambda function logs for errors
No Lambda logs/execution but EventBridge shows matched events: This indicates EventBridge is detecting events but failing to invoke Lambda. Make sure you selected “Create a new role for this resource” when setting up the EventBridge target, rather than using a pre-existing execution role. This is critical because AWS will create a service-role whereas if you create the role yourself it would just be a regular role. It might look like it would work but it will fail to execute the Lambda function.
EventBridge shows failed invocations: This is the same issue as the prior bullet point. Check CloudWatch metrics for your EventBridge rule and if you see failed invocations usually it’s because of permission issues with the execution role, and/or the resource-based policy in Lambda.

# Download the Terraform template that deploys this
Now that you’ve learned how to do this via the AWS console, it’s time for you to deploy this in your own AWS environments. I’ve made it super easy for you to do this with a Terraform template that you can download here. This template is fully customizable and in fact I would expect you to customize it after successfully deploying it in one of your sandbox accounts and testing it. You’ll want to customize it to your exact organization’s needs, and this template makes that super easy to do.

# Conclusion
Nice work! You’ve successfully built an automated security response system.

This is just one example of security automation. The same patterns can be applied to detect and respond to many other security events. Beyond this lab, I would encourage you to think about different scenarios where you could apply this in your personal or org accounts without negatively impacting production or colleagues. I’ll leave you with a few ideas…













     



