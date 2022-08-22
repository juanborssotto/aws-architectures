# Important note
You should enable SSH agent forwarding with caution. When you set up agent forwarding, a socket file is created on the forwarding host, which is the mechanism by which the key can be forwarded to your destination. Another user on the system with the ability to modify files could potentially use this key to authenticate as you. See the SSH manual for more details.

* Never place your SSH private keys on the bastion instance. Instead, use SSH agent forwarding to connect first to the bastion and from there to other instances in private subnets. This lets you keep your SSH private key just on your computer.
* Configure the security group on the bastion to allow SSH connections (TCP/22) only from known and trusted IP addresses.
* Always have more than one bastion. You should have a bastion in each availability zone (AZ) where your instances are. If your deployment takes advantage of a VPC VPN, also have a bastion on premises.
* Configure Linux instances in your VPC to accept SSH connections only from bastion instances.
* Make sure the bastion host only has port 22 traffic from the IP address you need, not from the security groups of your other EC2 instances

References: https://aws.amazon.com/blogs/security/securely-connect-to-linux-instances-running-in-a-private-amazon-vpc/

# Steps

## Create the private instance key pair
Create an EC2 key pair, name it the same as the PrivateInstanceKeyPairName parameter value and save the .pem in your local machine

## Create the stack
```
aws cloudformation create-stack \
--stack-name BastionHostStack \
--template-body file://./templates/template.yaml \
--parameters \
  ParameterKey=BastionInstanceImageId,ParameterValue=ami-090fa75af13c156b4 \
  ParameterKey=BastionInstanceInstanceType,ParameterValue=t2.micro \
  ParameterKey=PrivateInstanceImageId,ParameterValue=ami-090fa75af13c156b4 \
  ParameterKey=PrivateInstanceInstanceType,ParameterValue=t2.micro \
  ParameterKey=DemoVpcCidrBlock,ParameterValue=10.0.0.0/16 \
  ParameterKey=PublicSubnetCidrBlock,ParameterValue=10.0.0.0/24 \
  ParameterKey=PrivateSubnetCidrBlock,ParameterValue=10.0.16.0/20 \
  ParameterKey=BastionInstanceAllowSSHTrafficFromCidr,ParameterValue=0.0.0.0/0 \
  ParameterKey=PublicSubnetAllowSSHTrafficFromCidr,ParameterValue=0.0.0.0/0 \
  ParameterKey=PrivateInstanceKeyPairName,ParameterValue=PrivateInstanceKeyPair
```

## Test the bastion host
From your local machine:
```
ssh-add your-key-pair.pem
ssh-add -L
ssh -v -A ec2-user@your-bastion-host-public-ip
```
From the bastion host:
```
ssh ec2-user@your-private-instance-private-ip
```

## Delete stack
```
aws cloudformation delete-stack \
--stack-name AutoScaleBasedOnQueueStack
```