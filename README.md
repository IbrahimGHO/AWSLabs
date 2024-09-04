
# CloudFormation Lab

## Objective
Create a Virtual Private Cloud (VPC) with a public subnet, an Internet Gateway, a Security Group, and an EC2 instance with an Apache web server.

## Parameters
```yaml
Parameters:
  InstanceProfileName:
    Description: Name of the IAM instance profile to attach to the EC2 instance
    Type: String
    Default: MyInstanceProfile
```

## Resources
### VPC
Objective: Create a Virtual Private Cloud (VPC) with a specific CIDR block.

```yaml
Resources:
  VPC:
    Type: 'AWS::EC2::VPC'
    Properties:
      CidrBlock: '10.0.0.0/16'
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: NV-VPC

```

### Create an Internet Gateway
Objective: Create an Internet Gateway (IGW) and attach it to the VPC to enable Internet access
```yaml

  InternetGateway:
    Type: 'AWS::EC2::InternetGateway'
    Properties:
      Tags:
        - Key: Name
          Value: MyInternetGateway

  VPCGatewayAttachment:
    Type: 'AWS::EC2::VPCGatewayAttachment'
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway
```

### Create a Public Subnet
Objective: Create a public subnet within the VPC that will house the EC2 instance.

```yaml

  PublicSubnet:
    Type: 'AWS::EC2::Subnet'
    Properties:
      VpcId: !Ref VPC
      CidrBlock: '10.0.1.0/24'
      MapPublicIpOnLaunch: true
      AvailabilityZone: !Select [ 0, !GetAZs '' ]
      Tags:
        - Key: Name
          Value: NVMyPublicSubnet
```

### Create a Route Table and Public Route
Objective: Create a Route Table and add a route that directs traffic to the Internet Gateway.
```yaml
  RouteTable:
    Type: 'AWS::EC2::RouteTable'
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: NV-MyRouteTable

  PublicRoute:
    Type: 'AWS::EC2::Route'
    DependsOn: VPCGatewayAttachment
    Properties:
      RouteTableId: !Ref RouteTable
      DestinationCidrBlock: '0.0.0.0/0'
      GatewayId: !Ref InternetGateway

  SubnetRouteTableAssociation:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref RouteTable
```

### Create a Security Group
Objective: Create a Security Group that allows SSH and HTTP traffic to the EC2 instance.

```yaml
  SecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: Allow SSH and HTTP
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: '0.0.0.0/0'
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: '0.0.0.0/0'
        - IpProtocol: icmp
          FromPort: -1
          ToPort: -1
          CidrIp: '0.0.0.0/0'
      Tags:
        - Key: Name
          Value: MySecurityGroup
```


### 6: Launch an EC2 Instance
Objective: Launch an EC2 instance within the public subnet, attach the Security Group, and install Apache server using user data.

```yaml 
  EC2Instance:
    Type: 'AWS::EC2::Instance'
    Properties:
      InstanceType: t2.micro
      ImageId: ami-00beae93a2d981137
      IamInstanceProfile: !Ref InstanceProfileName
      NetworkInterfaces:
        - AssociatePublicIpAddress: true
          DeviceIndex: 0
          SubnetId: !Ref PublicSubnet
          GroupSet:
            - !Ref SecurityGroup
      Tags:
        - Key: Name
          Value: MyEC2Instance
      UserData:
        Fn::Base64: |
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl start httpd
          systemctl enable httpd
          echo "Hello World from $(hostname -f)" > /var/www/html/index.html
```

## Outputs
Objective: Retrieve important outputs such as the Public IP of the EC2 instance

```yaml
Outputs:
  VPCId:
    Description: The ID of the VPC
    Value: !Ref VPC

  PublicIP:
    Description: Public IP of the EC2 instance
    Value: !GetAtt EC2Instance.PublicIp
```


