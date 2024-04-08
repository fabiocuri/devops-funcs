This repo contains all the code for the exercises of the DevOps bootcamp.

Author: Fabio Curi

**Exercise 1**

```
aws configure list
cat ~/.aws/credentials
aws iam create-user --user-name fabio
aws iam create-group --group-name devops
aws iam add-user-to-group --user-name fabio --group-name devops
aws iam get-group --group-name devops
aws iam create-access-key --user-name fabio > key.txt
aws iam create-login-profile --user-name fabio --password MyTestPassword123
aws iam list-policies | grep ChangePassword
aws iam attach-user-policy --user-name fabio --policy-arn "arn:aws:iam::aws:policy/IAMUserChangePassword"
aws iam list-policies | grep EC2FullAccess
aws iam list-policies | grep VPCFullAccess
aws iam attach-group-policy --group-name devops --policy-arn "arn:aws:iam::aws:policy/AmazonEC2FullAccess"
aws iam attach-group-policy --group-name devops --policy-arn "arn:aws:iam::aws:policy/AmazonVPCFullAccess"
aws iam list-attached-group-policies --group-name devops
```

**Exercise 2**

```
aws configure
```

**Exercise 3**

```
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query Vpc.VpcId --output text
aws ec2 create-subnet --vpc-id vpc-0f61f35a20689ce58 --cidr-block 10.0.1.0/24
aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
aws ec2 attach-internet-gateway --vpc-id vpc-0f61f35a20689ce58 --internet-gateway-id igw-042a28c61aa063d8f
aws ec2 create-route-table --vpc-id vpc-0f61f35a20689ce58 --query RouteTable.RouteTableId --output text
aws ec2 create-route --route-table-id rtb-05d16c02e69e3e34d --destination-cidr-block 0.0.0.0/0 --gateway-id igw-042a28c61aa063d8f
aws ec2 describe-route-tables --route-table-id rtb-05d16c02e69e3e34d
aws ec2 associate-route-table  --subnet-id subnet-0e24fc84f2cb6e0fc --route-table-id rtb-05d16c02e69e3e34d
aws ec2 create-security-group --group-name SSHAccess --description "Security group for SSH access" --vpc-id vpc-0f61f35a20689ce58
aws ec2 authorize-security-group-ingress --group-id sg-09b032542907348e7 --protocol tcp --port 22 --cidr 0.0.0.0/0
```

**Exercise 4**

```
aws ec2 create-key-pair --key-name WebServerKeyPair --query "KeyMaterial" --output text > WebServerKeyPair.pem
chmod 400 WebServerKeyPair.pem
aws ec2 run-instances --image-id ami-0f7204385566b32d0 --count 1 --instance-type t2.micro --key-name WebServerKeyPair --security-group-ids sg-09b032542907348e7 --subnet-id subnet-0e24fc84f2cb6e0fc --associate-public-ip-address
```

**Exercise 5**

```
ssh -i "WebServerKeyPair.pem" ec2-user@3.68.69.227
sudo yum update -y && sudo yum install -y docker
sudo systemctl start docker 
sudo usermod -aG docker ec2-user
```

**Exercise 6**

Added the following docker compose file.

```
version: '3.8'
services:
    nodejs-app:
      image: ${IMAGE}
      ports:
        - 3000:3000
```

**Exercise 7**

Created the server-cmds.sh file and Jenkinsfile.
The following repo has a code similar to what I developed for the npm exercise: https://gitlab.com/fabiocuri/aws-exercise

**Exercise 8**

```
aws ec2 authorize-security-group-ingress --group-id sg-09b032542907348e7 --protocol tcp --port 3000 --cidr 0.0.0.0/0
```

**Exercise 9**

Added the following part to Jenkinsfile:

```
          when {
            expression {
              return env.BRANCH_NAME == "main"
            }
          }

```
