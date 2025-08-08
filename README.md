# GuardDuty Threat Detection (AI-Powered Alerts)


## Architecture
![image alt](https://github.com/Dannyz513/Guard-Duty-Threat-Detection/blob/3399ae7308eb236d021d3c80a1343bf60262247c/GUARDDUTY%20ARCHITECTURE.png)

**Flow:** GuardDuty Finding → EventBridge Rule → Lambda (formats alert) → SNS (email/SMS)

### 2) Enable GuardDuty
Turn on GuardDuty in your home region. Wait a few minutes while it initializes.  
![image alt](https://github.com/Dannyz513/Guard-Duty-Threat-Detection/blob/bbf55978fefd7acce7f48a2c741ab0bf94b51f37/Guard%20duty.png)

### 3) Create an SNS Topic + Subscription
Create `GuardDuty-Threat-Alerts`. Subscribe your email and confirm the subscription.  
_Screenshot placeholder:_ ![Step 3 – SNS](images/step3_sns_placeholder.png)

### 4) Create the Lambda function
Create `GuardDuty-Automated-Response` (Python 3.x). Attach **AWSLambdaBasicExecutionRole**, add **sns:Publish** permission to your topic, and set env var `SNS_TOPIC_ARN=<your topic ARN>`.  
![image alt](https://github.com/Dannyz513/Guard-Duty-Threat-Detection/blob/bbf55978fefd7acce7f48a2c741ab0bf94b51f37/Guard%20duty.png)

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
