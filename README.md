#  Deployment of EC2 instance and EBS volume using AWSCLI

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

## _Description_

Here we are going to create an AWS instance without Aws console .How is that possibile ?. I just tell you the project baseline, here  we want to create a basic infrastructure for a website where there is no access to AWS console. The only details that we have is an  IAM user with progrmatic access. Initially, we are creating a VPC (172.16.0.0/16) in ap-south-1 region  with 3 public subnets. Then the  next step is the creation of EC2 instance, which includes selecting an appropriate AMI, Instance type, security groups, key pairs, etc...In this  project we are using http as our webserver. There was an additional requirement which was to create and attach additional volume (Ebs)to the document root of the Ec2 webserver. All this steps are done via command line interface. I have provided herewith a detailed command summary that I have executed in the project.


## _Resources_

- VPC
- Subnets
- Internet gateway
- Route table
- Security group
-  Key pair
-  EC2 instance
-  Additional Ebs volume


## Prerequisites for this project
- AWS CLI on your system 
- Need an IAM user access with attached policies for the creation of EC2  instance
- Knowledge to the  requirements of Vpc, Ec2, Ebs, IAM 

## Installtion of AWS CLI 
Installation of AWCLI depends on the Operating systems that installed on your system. Please check the following official documentation to install the awscli on your system.

_https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html_



## STEP 1 - IAM Role Creation

Initially, I have configured an IAM user with the available access key ID and AWS secret key.

```sh
AWS Access Key ID [None]: **************
AWS Secret Access Key [None]: ***********
Default region name [None]: ap-south-1
Default output format [None]: json

```
## STEP 2 -Creation of VPC, Subnets, Internet Gateway, Route tables


Here, I have created a vpc with IP range 172.16.0.0/16

```sh
aws ec2 create-vpc --cidr-block 172.16.0.0/16
```

Output

```sh 
{
    "Vpc": {
        "CidrBlock": "172.16.0.0/16",
        "DhcpOptionsId": "dopt-770fa21c",
        "State": "pending",
        "VpcId": "vpc-0b7c1bfe7fb631fd3",
        "OwnerId": "
        ```
        600002467561",
        "InstanceTenancy": "default",
        "Ipv6CidrBlockAssociationSet": [],
        "CidrBlockAssociationSet": [
            {
                "AssociationId": "vpc-cidr-assoc-07536c084d86449c9",
                "CidrBlock": "172.16.0.0/16",
                "CidrBlockState": {
                    "State": "associated"
                }
            }
        ],
        "IsDefault": false
    }
}
```
#### Assigning name tag for newly created VPC

```sh
aws ec2 create-tags --resources vpc-0b7c1bfe7fb631fd3 --tags Key=Name,Value=AWS-CLI-VPC
```

#### Subnet creation

Here we are partitiong the previously created VPC "vpc-0b7c1bfe7fb631fd3" to 3 subnets.

```sh 
aws ec2 create-subnet --vpc-id vpc-0b7c1bfe7fb631fd3 --cidr-block 172.16.0.0/18

aws ec2 create-subnet --vpc-id vpc-0b7c1bfe7fb631fd3 --cidr-block 172.16.64.0/18

aws ec2 create-subnet --vpc-id vpc-0b7c1bfe7fb631fd3 --cidr-block 172.16.64.0/18
```

The next steps is the conversion of this subnets to public. 

#### Creation of  internet gateway  with name tag "igw-awscli".

```sh 
aws ec2 create-internet-gateway

aws ec2 create-tags --resources igw-045fbb708b900abce --tags Key=Name,Value=igw-awscli
```

#### Attached internet gateway to VPC "AWS-CLI-VPC" ID: vpc-0b7c1bfe7fb631fd3

```sh
aws ec2 attach-internet-gateway --vpc-id vpc-0b7c1bfe7fb631fd3 --internet-gateway-id igw-045fbb708b900abce
```

#### Created a custom route table for the VPC.

```sh 
aws ec2 create-route-table --vpc-id vpc-0b7c1bfe7fb631fd3
```
Output:

```sh 
{
    "RouteTable": {
        "Associations": [],
        "PropagatingVgws": [],
        "RouteTableId": "rtb-0fe7412c018349f87",
        "Routes": [
            {
                "DestinationCidrBlock": "172.16.0.0/16",
                "GatewayId": "local",
                "Origin": "CreateRouteTable",
                "State": "active"
            }
        ],
        "Tags": [],
        "VpcId": "vpc-0b7c1bfe7fb631fd3",
        "OwnerId": "600002467561"
```

Pointed the route table to the IGW with all traffic (0.0.0.0/0) and also a nametag is added for the route table.

```sh 
aws ec2 create-route --route-table-id rtb-0fe7412c018349f87  --destination-cidr-block 0.0.0.0/0 --gateway-id igw-045fbb708b900abce

aws ec2 create-tags --resources rtb-0fe7412c018349f87 --tags Key=Name,Value=route-aws-cli
```

Output:

```sh 
{
    "Return": true
}

```

The next step is attaching the  route table to subnets so that traffic from that subnet is routed to the internet gateway.

The below command list the subnets under the VPC.

```sh 
aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-0b7c1bfe7fb631fd3" --query "Subnets[*].{ID:SubnetId,CIDR:CidrBlock}"
```

Ouptut:

```sh 
[
    {
        "ID": "subnet-0a8106e1bfedfd92f",
        "CIDR": "172.16.0.0/18"
    },
    {
        "ID": "subnet-0156c169f2d9f146a",
        "CIDR": "172.16.128.0/18"
    },
    {
        "ID": "subnet-04ad53e10e7a89db0",
        "CIDR": "172.16.64.0/18"
    }
]
```
Then accordingly update subnet to associate with the custom route table.

```sh 
aws ec2 associate-route-table  --subnet-id subnet-0a8106e1bfedfd92f --route-table-id  rtb-0fe7412c018349f87
aws ec2 associate-route-table  --subnet-id subnet-0156c169f2d9f146a --route-table-id  rtb-0fe7412c018349f87
aws ec2 associate-route-table  --subnet-id subnet-04ad53e10e7a89db0 --route-table-id  rtb-0fe7412c018349f87
```

Then the next step is  enabling  of subnet's public IPv4 addressing behavior so that an instance launched into the subnet automatically receives a public IP address.

```sh 
aws ec2 modify-subnet-attribute --subnet-id subnet-0a8106e1bfedfd92f --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --subnet-id subnet-0156c169f2d9f146a --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --subnet-id subnet-04ad53e10e7a89db0 --map-public-ip-on-launch
```


## STEP 3 -Creation of security groups, key pair, Instance creation


#### Creation of Security group

A security group is an instance level firewall.  In this project I created a security group with open ports 22 (SSH protocol), 80 (http port), and 443 (http secure port).

```sh
   aws ec2 create-security-group --group-name sec-aws-cli --description "this is a sample security group with open ports http,ssh,https" --vpc-id vpc-0b7c1bfe7fb631fd3
   aws ec2 create-tags --resources sg-07e05244d8b25d4ff --tags Key=Name,Value=sec-aws-cli
```
  
 Output:
 
 ```sh 
 {
    "GroupId": "sg-07e05244d8b25d4ff"
}
```

Next, we are opening port 22, 80 and 443  for inbound connections in security group.

```sh
aws ec2 authorize-security-group-ingress --group-id sg-07e05244d8b25d4ff  --protocol tcp --port 22 --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress --group-id sg-07e05244d8b25d4ff  --protocol tcp --port 80 --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress --group-id sg-07e05244d8b25d4ff  --protocol tcp --port 443 --cidr 0.0.0.0/0

```

#### Creation of Key Pair

The aws ec2 command stores the public key and outputs the private key for you to save to a file.

The following command creates private key "Key-example" and stores in pem  file "Key-example"

```sh
aws ec2 create-key-pair --key-name awsclikeymaterial --query 'keyMaterial' --output text > awsclikeymaterial.pem
```

#### Amazone Machine Image

You can view a list of all Linux AMIs in the current AWS Region by using the following command in the AWS CLI.

```sh 
aws ssm get-parameters-by-path --path /aws/service/ami-amazon-linux-latest --query "Parameters[].Name"
```
Also, refer the following documention
_https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html_

#### Instance Type

t2.micro is the only instance type available in the free tier.

#### Ec2 Creation


```sh

aws ec2 run-instances --image-id resolve:ssm:/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2  --count 1 --instance-type t2.micro --key-name awsclikeymaterial --security-group-ids sg-061e92f7484adc098 --subnet-id subnet-0a8106e1bfedfd92f

```

###### You can list your instances withaws ec2 describe-instances

```sh
aws ec2 describe-instances
```
###### You can set the instance name using following command
```sh
aws ec2 create-tags --resources i-05c2e7e8239d1b183 --tags Key=Name,Value=aws-cli-server
```
###### Inorder to find the public IP address run the following command
```sh
aws ec2 describe-instances --instance-ids i-05c2e7e8239d1b183 --query "Reservations[0].Instances[0].PublicIpAddress"
```

## EBS volume creation and attachment

You can use the following command to create an additonal 2GB EBS volume on avilabilty zone ap-south-1a

```sh
aws ec2 create-volume --region ap-south-1 --availability-zone ap-south-1a --size 2 --volume-type gp2
```

To attach the additonal EBS volume to your instance run the following command

```sh
aws ec2 attach-volume --volume-id vol-01eaa179007f7edc4 --instance-id i-05c2e7e8239d1b183 --device /dev/sdf
```

> Now we can connect to our EC2 
> instance over the Internet.

```sh 
ssh -i  ec2-user@*.*.*.*
```
## Installtion and setup of webserver and mounting the document root  to the additonal EBS volume

Here, we are going to install an  httpd service on our EC2 instance and attaching it's document root the additonal EBS volume

```sh 
[root@ip-172-31-6-205 ~]# yum install httpd -y ; systemctl start httpd.service; systemctl enable httpd.service
[root@ip-172-31-6-205 ~]# lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk 
└─xvda1 202:1    0   8G  0 part /
xvdf    202:80   0   2G  0 disk 
└─xvdf1 202:81   0   2G  0 part /var/www/html
```

## Hosting a sample website

Here, I have uploaded a sample html  code on document root of the domain and pointed my website to the public IP address of EC2 instance.

```sh 
[root@ip-172-31-6-205 html]# ll
total 44
drwxr-xr-x 2 apache apache  4096 Aug 17 21:12 2102_constructive
-rw-r--r-- 1 apache apache   453 Oct 19  2018 ABOUT THIS TEMPLATE.txt
drwxr-xr-x 2 apache apache  4096 Jul  4  2018 css
drwxr-xr-x 2 apache apache  4096 Jul  3  2018 img
-rw-r--r-- 1 apache apache 14437 Dec 16  2018 index.html
drwxr-xr-x 2 apache apache  4096 Jul  3  2018 js
drwxr-xr-x 3 apache apache  4096 Jul  3  2018 slick
drwxr-xr-x 2 apache apache  4096 Jul  3  2018 webfonts
[root@ip-172-31-6-205 html]#
```

## Conclusion



  
