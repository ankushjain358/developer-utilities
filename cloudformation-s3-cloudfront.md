# CloudFormation for CloudFront to S3 with OAC
This CloudFormation template allows you to provision a very general purpose stack comprising of an S3 bucket, a CloudFront distribution. 

In this setup, the public access to the S3 bucket is disabled, and CloudFront accesses the original S3 bucket via OAC.

## CloudFormation Template
```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: This CloudFormation template allows you to provision a very general purpose stack comprising of an S3 bucket, a CloudFront distribution.
Parameters:
  BucketName:
    Type: String
    Description: Enter S3 bucket name
Resources:
  CloudFrontOriginAccessControl:
    Type: AWS::CloudFront::OriginAccessControl
    Properties:
      OriginAccessControlConfig:
        Name: !Sub "OAC_${AWS::AccountId}_${AWS::StackName}"
        OriginAccessControlOriginType: s3
        SigningBehavior: always
        SigningProtocol: sigv4
  CloudFrontDistribution:
    Type: AWS::CloudFront::Distribution
    Metadata:
      Comment: CachePolicyId is the id of AWS managed policy - CachingOptimized
    Properties:
      DistributionConfig:
        Enabled: true
        DefaultRootObject: index.html
        PriceClass: PriceClass_100
        DefaultCacheBehavior:
          CachePolicyId: 658327ea-f89d-4fab-a63d-7e88639e58f6
          TargetOriginId: S3BucketOrigin
          ViewerProtocolPolicy: redirect-to-https
        Origins:
          - DomainName: !GetAtt "Bucket.DomainName"
            Id: S3BucketOrigin
            OriginAccessControlId: !Ref "CloudFrontOriginAccessControl"
            S3OriginConfig:
              OriginAccessIdentity: ""
  Bucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
    Properties:
      BucketName: !Ref BucketName
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - BucketKeyEnabled: true
            ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
  BucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref Bucket
      PolicyDocument:
        Version: "2008-10-17"
        Statement:
          - Sid: AllowCloudFrontServicePrincipalReadOnly
            Action:
              - s3:GetObject
            Effect: Allow
            Principal:
              Service: cloudfront.amazonaws.com
            Resource: !Sub "${Bucket.Arn}/*"
            Condition:
              StringEquals:
                AWS:SourceArn: !Sub "arn:aws:cloudfront::${AWS::AccountId}:distribution/${CloudFrontDistribution.Id}"
Outputs:
  CloudFrontDistributionUrl:
    Value: !Sub "https://${CloudFrontDistribution.DomainName}"
    Description: URL of the CloudFront distribution
```
## How to deploy
Use below commands to deploy the above Cfn template.
> Replace \ with ^ when running below multiline command in Windows.

### Method 1
`aws cloudformation create-stack` is a fire-and-forget method. It does not wait for the stack operation to finish.
```bash
aws cloudformation create-stack \
  --stack-name <stack-name> \
  --template-url https://cdn.coderjony.com/cloudformation/s3-cloudfront.yaml \
  --parameters ParameterKey=BucketName,ParameterValue=<bucket-name> \
  --capabilities CAPABILITY_IAM \
  --region us-east-1
```

### Method 2
`aws cloudformation deploy` is a synchronous method that waits for the stack operation to complete, whether it succeeds or fails.
```bash
aws cloudformation deploy \
  --stack-name <stack-name> \
  --template-file <template-filepath> \
  --parameter-overrides BucketName=<bucket-name> \
  --region us-east-1
```

## Optional - Add a Custom Domain to CloudFront distribution with an ACM SSL certificate
Refer [How to add a Custom Domain to CloudFront distribution with SSL](https://github.com/ankushjain358/developer-utilities/blob/main/add-a-custom-domain-to-cloudfront-distribution-with-ssl.md) for more detail.

## Clean up
```bash
aws cloudformation delete-stack --stack-name <stack-name>
```
