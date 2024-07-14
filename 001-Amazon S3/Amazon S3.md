Cloudformation template for the cross-account s3 access:
```
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  # IAM Role in Account A
  CrossAccountRole:
    Type: "AWS::IAM::Role"
    Properties: 
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement: 
          - Effect: "Allow"
            Principal: 
              AWS: !Sub arn:aws:iam::${AccountBID}:user/${AccountBUserName}
            Action: "sts:AssumeRole"
      Policies: 
        - PolicyName: "CrossAccountS3Access"
          PolicyDocument: 
            Version: "2012-10-17"
            Statement: 
              - Effect: "Allow"
                Action: 
                  - "s3:GetObject"
                  - "s3:PutObject"
                  - "s3:ListBucket"
                Resource: 
                  - !Sub arn:aws:s3:::${S3BucketName}
                  - !Sub arn:aws:s3:::${S3BucketName}/*

  # S3 Bucket Policy
  S3BucketPolicy:
    Type: "AWS::S3::BucketPolicy"
    Properties:
      Bucket: !Ref S3BucketName
      PolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: "Allow"
            Principal:
              AWS: !GetAtt CrossAccountRole.Arn
            Action:
              - "s3:GetObject"
              - "s3:PutObject"
              - "s3:ListBucket"
            Resource:
              - !Sub arn:aws:s3:::${S3BucketName}
              - !Sub arn:aws:s3:::${S3BucketName}/*

Parameters:
  AccountBID:
    Type: String
    Description: "The AWS Account ID for Account B."
  AccountBUserName:
    Type: String
    Description: "The user name in Account B who will assume the role."
  S3BucketName:
    Type: String
    Description: "The name of the S3 bucket to grant access to."

Outputs:
  CrossAccountRoleArn:
    Description: "The ARN of the cross-account IAM role."
    Value: !GetAtt CrossAccountRole.Arn

```


**How to Use This Updated Template?**  
Save the Template: Save the updated YAML content into a file, e.g., cross-account-s3-access.yaml.

**Login to AWS Management Console:**  

- Open the AWS Management Console and log in to your Account A.  
- In the AWS Management Console, navigate to the CloudFormation service.  

**Create a Stack:**  
- Click on "Create stack" and choose "With new resources (standard)".
- 
**Upload the Template:**  
- In the "Specify template" section, choose "Upload a template file".
- Click "Choose file" and select the cross-account-s3-access.yaml file you saved earlier.
- Click "Next".

**Specify Stack Details:**    

- Stack name: Enter a name for your stack, e.g., CrossAccountS3Access.
- Parameters: Enter the Account B ID, Account B User Name, and the S3 bucket name.
- AccountBID: Enter the AWS Account ID of Account B.
- AccountBUserName: Enter the user name in Account B who will assume the role.
- S3BucketName: Enter the name of the S3 bucket in Account A.
- Click "Next".

 
**Configure Stack Options:**  
 - Configure any additional stack options if needed (e.g., tags, IAM role, etc.).
 - Click "Next".

**Review:**  
- Review the stack details and ensure everything is correct.
- Acknowledge that CloudFormation might create IAM resources with custom names.
- Click "Create stack".

  
**Wait for Stack Creation:**  
- CloudFormation will now create the resources defined in the template. This might take a few minutes.
- You can monitor the progress on the "Events" tab of the stack.  

**Completion:**  
- Once the stack is created, you should see the status as CREATE_COMPLETE.
- The output section will show the ARN of the cross-account IAM role created.
- This updated CloudFormation template includes a parameter for the user in Account B who will assume the role, ensuring that the access is granted to the specific user.  
