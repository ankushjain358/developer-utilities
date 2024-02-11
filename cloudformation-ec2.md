## CloudFormation for EC2 in Public Subnet 

- [CloudFormation Template](#cloudformation-template)
- [How to deploy](#how-to-deploy)

## CloudFormation Template
```yaml
AWSTemplateFormatVersion: "2010-09-09"
Parameters:
  KeyName:
    Type: "AWS::EC2::KeyPair::KeyName"
    Description: "Name of an existing EC2 KeyPair to enable SSH access to the instance"

  InstanceType:
    Type: "String"
    Description: "EC2 instance type for the new instance"
    Default: "t2.micro"

  AMIId:
    Type: "AWS::EC2::Image::Id"
    Description: "ID of the Amazon Machine Image (AMI) to launch"

Resources:
  MyVPC:
    Type: "AWS::EC2::VPC"
    Properties:
      CidrBlock: "10.0.0.0/16"
      EnableDnsSupport: true
      EnableDnsHostnames: true

  MyPublicSubnet:
    Type: "AWS::EC2::Subnet"
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: "10.0.0.0/24"
      MapPublicIpOnLaunch: true  # Enable public IP for instances in this subnet

  MyInternetGateway:
    Type: "AWS::EC2::InternetGateway"
    Properties: {}

  MyAttachGateway:
    Type: "AWS::EC2::VPCGatewayAttachment"
    Properties:
      VpcId: !Ref MyVPC
      InternetGatewayId: !Ref MyInternetGateway

  MyPublicRouteTable:
    Type: "AWS::EC2::RouteTable"
    Properties:
      VpcId: !Ref MyVPC

  MyDefaultRoute:
    Type: "AWS::EC2::Route"
    DependsOn: MyAttachGateway
    Properties:
      RouteTableId: !Ref MyPublicRouteTable
      DestinationCidrBlock: "0.0.0.0/0"
      GatewayId: !Ref MyInternetGateway

  MySubnetAssociation:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties:
      SubnetId: !Ref MyPublicSubnet
      RouteTableId: !Ref MyPublicRouteTable

  MySecurityGroup:
    Type: "AWS::EC2::SecurityGroup"
    Properties:
      GroupDescription: "Enable SSH and RDP access to EC2 instance"
      VpcId: !Ref MyVPC
      SecurityGroupIngress:
        - IpProtocol: "tcp"
          FromPort: 22
          ToPort: 22
          CidrIp: "0.0.0.0/0"  # Allow SSH from any IP
        - IpProtocol: "tcp"
          FromPort: 3389
          ToPort: 3389
          CidrIp: "0.0.0.0/0"  # Allow RDP from any IP

  MyInstance:
    Type: "AWS::EC2::Instance"
    Properties:
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyName
      ImageId: !Ref AMIId  # Use the provided AMI ID
      NetworkInterfaces:
        - AssociatePublicIpAddress: true
          DeviceIndex: 0
          GroupSet:
            - !Ref MySecurityGroup
          SubnetId: !Ref MyPublicSubnet
      CreditSpecification:
        CPUCredits: 'unlimited' # This is only required for T family burstable instances

```

## How to deploy
Use below command to deploy the above Cfn template.
> Replace `\` with `^` when running below multiline command in Windows.
```
aws cloudformation create-stack \
  --stack-name EC2-Stack \
  --template-body file://C:/Users/username/Desktop/EC2/template.yml \
  --parameters ParameterKey=KeyName,ParameterValue=<key-name> ParameterKey=InstanceType,ParameterValue=<instance-type> ParameterKey=AMIId,ParameterValue=<ami-id> \
  --capabilities CAPABILITY_IAM
```
Replace the following placeholders with the actual values:
- `<key-name>` replace it with your existing key.
- `<instance-type>` - replace it with the instacen type *i.e. t2.micro*
- `<ami-id>` - repalce it with Application Machine Image ID *i.e. ami-00d59001b2335bdea*
