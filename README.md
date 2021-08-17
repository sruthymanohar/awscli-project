#  Launching an EC2 instance using AWS CLI and attachment of EBS volume

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

## _Description_

 The AWS Command Line Interface (AWS CLI) is an open source tool that enables you to interact with AWS services using commands in your command-line shell. Here is a simple document to create an EC2 instance and attachment of additional EBS using AWS CLI.

## _Resources_

- Creation of Security group
- Creation of Key pair
- Creation of an EC2 instance
- List the  instances in an aws account
- Add a tag to your instance
- Find the public and private IP address of an aws instance
- ssh into the EC2 instance and installation of a webserver
-  Attachment of additional 2GB  EBS voulme to the document root of the EC2 instance


## Prerequisites for this project
- AWS CLI on your system 
- Need an IAM user access with attached policies for the creation of EC2  instance
- Knowledge to the  requirements of EC2 and Ebs creation

## Installtion of AWS CLI 

_https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html_



## IAM Role Creation

Here I have configured an IAM user with Administrator policy. 

You can also, refer following url for IAM creation.

_https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create.html_

```sh
C:\Users\user1>aws configure
AWS Access Key ID [****************]: **************
AWS Secret Access Key [****************2TiW]: ******************
Default region name [ap-south-1]: ap-south-1 
Default output format [json]: json
```


## Creation of Security group

A security group is an instance level firewall. You can set rules to limit access to your instance. In this case we are creating a security group with open ports 22 (SSH protocol), 80 (http port), and 443 (http secure port).

The below command creates a security group with name CLI-example
  ```sh
   aws ec2 create-security-group --group-name CLI-example --description "this is a sample security group with open ports http,ssh,https"
```
   > {
   >    "GroupId": "sg-06cb2deda82f27ed0"
   > }
 

Next, we are opening port 22, 80 and 443  for inbound connections in security group.

```sh
> aws ec2 authorize-security-group-ingress --group-name CLI-example --protocol tcp --port  22 --cidr 0.0.0.0/0

Results::

{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-010ee581955d52fd4",
            "GroupId": "sg-06cb2deda82f27ed0",
            "GroupOwnerId": "600002467561",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}

For  80 and 443 run the above command by replacing port number
```

To describe the security policy, you can use the following command

```sh
aws  ec2  describe-security-groups --group-name CLI-example
```
## Creation of Key Pair

The aws ec2 command stores the public key and outputs the private key for you to save to a file.

The following command creates private key "Key-example" and stores in pem  file "Key-example"

```sh
aws ec2 create-key-pair --key-name Key-example --query 'KeyMaterial' --output text > Key-example.pem
```

## Amazone Machine Image

You can view a list of all Linux AMIs in the current AWS Region by using the following command in the AWS CLI.

```sh 
aws ssm get-parameters-by-path --path /aws/service/ami-amazon-linux-latest --query "Parameters[].Name"
```
Also, refer the following documention
_https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html_

## Instance Type

t2.micro is the only instance type available in the free tier.

## Ec2 Creation

###### The following command is used to create an instance with a security group, AMI and key pair.

```sh
aws ec2 run-instances --image-id resolve:ssm:/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --instance-type t2.micro --key-name Key-example --security-group-ids sg-06cb2deda82f27ed0
```

###### You can list your instances withaws ec2 describe-instances

```sh
aws ec2 describe-instances
```
###### You can set the instance name using following command
```sh
aws ec2 create-tags --resources i-08c2d839031be070f --tags Key=Name,Value=CLI_erver
```
###### Inorder to find the public IP address run the following command
```sh
aws ec2 describe-instances --instance-ids i-08c2d839031be070f --query "Reservations[0].Instances[0].PublicIpAddress"
```

## EBS volume creation and attachment

You can use the following command to create an additonal 2GB EBS volume on avilabilty zone ap-south-1b

```sh
aws ec2 create-volume --volume-type gp2 --size 2 --availability-zone ap-south-1b
```

To attach the additonal EBS volume to your instance run the following command

```sh
aws ec2 attach-volume --volume-id vol-0390ad0c4ca6a29b1 --instance-id i-0782ca4d61f8f3f39 --device /dev/sdf
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

Now the Website is  loading 

![alt text](https://github.com/sruthymanohar/awscli-ec2-creation/blob/main/Capture.PNG)



  
