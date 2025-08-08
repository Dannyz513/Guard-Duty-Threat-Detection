# GuardDuty Threat Detection (AI-Powered Alerts)

> **Repo name**: `guard duty threat detection`  
> This project builds a real-time threat detection and alerting pipeline on AWS using **Amazon GuardDuty**, **EventBridge**, **AWS Lambda**, and **Amazon SNS**. It mirrors common “AI-driven threat intel” pipelines by leveraging GuardDuty’s ML-based detections and automating notifications.

## What this repo shows recruiters
- Wire up a production-style **security alert pipeline** on AWS
- Understand **GuardDuty findings**, **EventBridge rules**, **Lambda handlers**, and **SNS alerts**
- Test end-to-end using **sample GuardDuty findings**

## Architecture
![Architecture Diagram – placeholder](images/01_architecture_placeholder.png)

**Flow:** GuardDuty Finding → EventBridge Rule → Lambda (formats alert) → SNS (email/SMS)

## Prerequisites
- AWS account with admin privileges for setup
- AWS CLI configured (`aws configure`)
- An email address for SNS subscription

## Step-by-step (6–7 steps)
### 1) Enable CloudTrail
Enable a management events trail (console or CLI) so GuardDuty has activity context.  
_Screenshot placeholder:_ ![Step 1 – CloudTrail](images/step1_cloudtrail_placeholder.png)

### 2) Enable GuardDuty
Turn on GuardDuty in your home region. Wait a few minutes while it initializes.  
_Screenshot placeholder:_ ![Step 2 – GuardDuty](images/step2_guardduty_placeholder.png)

### 3) Create an SNS Topic + Subscription
Create `GuardDuty-Threat-Alerts`. Subscribe your email and confirm the subscription.  
_Screenshot placeholder:_ ![Step 3 – SNS](images/step3_sns_placeholder.png)

### 4) Create the Lambda function
Create `GuardDuty-Automated-Response` (Python 3.x). Attach **AWSLambdaBasicExecutionRole**, add **sns:Publish** permission to your topic, and set env var `SNS_TOPIC_ARN=<your topic ARN>`.  
_Screenshot placeholder:_ ![Step 4 – Lambda](images/step4_lambda_placeholder.png)

**Inline policy JSON (update ARN):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:REGION:ACCOUNT_ID:GuardDuty-Threat-Alerts"
    }
  ]
}
```

### 5) Create the EventBridge Rule
Target the Lambda on GuardDuty findings (example filters `Trojan:EC2/BlackholeTraffic`):
```json
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"],
  "detail": {
    "type": ["Trojan:EC2/BlackholeTraffic"]
  }
}
```
_Screenshot placeholder:_ ![Step 5 – EventBridge](images/step5_eventbridge_placeholder.png)

### 6) Test with sample findings
Replace `YOUR_DETECTOR_ID` and run:
```bash
aws guardduty create-sample-findings   --detector-id YOUR_DETECTOR_ID   --finding-types "Trojan:EC2/BlackholeTraffic"
```
You should receive an SNS email alert.  
_Screenshot placeholder:_ ![Step 6 – Alert](images/step6_alert_placeholder.png)

### 7) (Optional) Extend coverage
Add more finding types, multiple rules to the same Lambda, or additional SNS subscriptions (email, SMS, HTTP).  
_Screenshot placeholder:_ ![Step 7 – Extensions](images/step7_extensions_placeholder.png)

## Files
```
guard duty threat detection/
├─ src/
│  └─ GuardDuty-Automated-Response.py
├─ images/
│  ├─ 01_architecture_placeholder.png
│  ├─ step1_cloudtrail_placeholder.png
│  ├─ step2_guardduty_placeholder.png
│  ├─ step3_sns_placeholder.png
│  ├─ step4_lambda_placeholder.png
│  ├─ step5_eventbridge_placeholder.png
│  ├─ step6_alert_placeholder.png
│  └─ step7_extensions_placeholder.png
├─ artifacts/
├─ README.md
└─ .gitignore
```

## Quick deploy (manual, console-first)
1. Create SNS topic + subscribe your email  
2. Create Lambda and paste code from `src/GuardDuty-Automated-Response.py`  
3. Add `SNS_TOPIC_ARN` env var; grant `sns:Publish` to topic  
4. Create EventBridge rule using the JSON above and set Lambda as target  
5. Generate sample findings and confirm email

## Notes
- GuardDuty already uses ML; this repo demonstrates detection-to-action wiring.
- For production: least-privilege IAM, DLQ, retries, SSM/Secrets for config.

## License
MIT
