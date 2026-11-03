what is iam (identity and access management)

- users 
  - Mapped to a physical user
- groups 
  - A group of users
  - can not have another group
- policies
  - a Json file contains permission for users and groups
  - Version
  - Statement
    - Action, Resource, Effect
- Security
  - Enable MFA
  - Tools
    - IAM Credential Report (Account level)
      - last login/ password update?
    - Access Advisor (user level)
      - when and what service used
- How to manage IAM
  - console
    - cloud console
  - Access Key
  - CLI
  - SDK
- Role for AWS Service
  - need to create IAM for aws services to act on my behalf
  - eg. create IAM for EC2 Service 
    - create role -> select service
- Best Practice 
  - grant minimum privilege 
### policies
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "StatementID",
      "Effect": "Allow | Deny",
      "Principal": "who",
      "Action": "what",
      "Resource": "which resource",
      "Condition": "when"
    }
  ]
}

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3FullAccessSpecificBucket",
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    }
  ]
}
```