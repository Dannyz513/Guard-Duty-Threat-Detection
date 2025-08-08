# GuardDuty Threat Detection (AI-Powered Alerts)


## Architecture
![image alt](https://github.com/Dannyz513/Guard-Duty-Threat-Detection/blob/3399ae7308eb236d021d3c80a1343bf60262247c/GUARDDUTY%20ARCHITECTURE.png)

**Flow:** GuardDuty Finding → EventBridge Rule → Lambda (formats alert) → SNS (email/SMS)

### 2) Enable GuardDuty
Turn on GuardDuty in your home region. Wait a few minutes while it initializes.  
![image alt](https://github.com/Dannyz513/Guard-Duty-Threat-Detection/blob/7c392b1b0f7ca19f19ed213cd83d451d6481ce96/Guard%20duty%201.png)

### 3) Create an SNS Topic + Subscription
Create `GuardDuty-Threat-Alerts`. Subscribe your email and confirm the subscription.  
![image alt](https://github.com/Dannyz513/Guard-Duty-Threat-Detection/blob/c32058a9fb8c54fd0e87db721c6d40b25cf0e12c/sns%20arn.png)

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
![image alt](https://github.com/Dannyz513/Guard-Duty-Threat-Detection/blob/342c25fd4f7edec34543f2f9477a42e485e27be3/Eventbridge.png)

### 7) (Optional) Extend coverage
Add more finding types, multiple rules to the same Lambda, or additional SNS subscriptions (email, SMS, HTTP).  
![image alt](https://github.com/Dannyz513/Guard-Duty-Threat-Detection/blob/bf264c4bd717da146fd37d4d977d0b3301f3ca58/Guard%20duty%20final.png)
