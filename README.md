This repo contains all the code for the exercises of the DevOps bootcamp.

Author: Fabio Curi Paixao 

E-mail: fcuri91@gmail.com

**Exercise 1**

```
import boto3

subnets = boto3.client('ec2').describe_subnets()
for subnet in subnets["Subnets"]:
    if subnet["DefaultForAz"]:
        print(subnet["SubnetId"])
```

**Exercise 2**

```
import boto3

iam = boto3.client('iam')
iam_users = iam.list_users()

last_active_user = iam_users["Users"][0]

for iam_user in iam_users["Users"]:
    print(iam_user["UserName"])
    print(iam_user["PasswordLastUsed"])
    
    if last_active_user["PasswordLastUsed"] < iam_user["PasswordLastUsed"]:
        last_active_user = iam_user

print("Last active user info:")
print(last_active_user["UserId"], last_active_user["UserName"], last_active_user["PasswordLastUsed"])
```

**Exercise 3**

```
from distutils import command
import boto3
import time
import paramiko
import requests
import schedule

ec2_resource = boto3.resource('ec2')
ec2_client = boto3.client('ec2')

image_id = 'xxxxx'
key_name = 'boto3-server-key'
instance_type = 't2.small'

ssh_private_key_path = '~/Downloads/boto3-server-key.pem' 
ssh_user = 'ec2-user'
ssh_host = ''

response = ec2_client.describe_instances(
    Filters=[
        {
            'Name': 'tag:Name',
            'Values': [
                'my-server',
            ]
        },
    ]
) 

instance_already_exists = len(response["Reservations"]) != 0 and len(response["Reservations"][0]["Instances"]) != 0
instance_id = ""

assert instance_already_exists == True
#this makes sure that the VM exists

instance = response["Reservations"][0]["Instances"][0]
instance_id = instance["InstanceId"]

ec2_instance_fully_initialised = False

while not ec2_instance_fully_initialised:

    statuses = ec2_client.describe_instance_status(
        InstanceIds = [instance_id]
    )
    if len(statuses['InstanceStatuses']) != 0:
        ec2_status = statuses['InstanceStatuses'][0]

        ins_status = ec2_status['InstanceStatus']['Status']
        sys_status = ec2_status['SystemStatus']['Status']
        state = ec2_status['InstanceState']['Name']
        ec2_instance_fully_initialised = ins_status == 'ok' and sys_status == 'ok' and state == 'running'

# Install DOCKER
commands_to_execute = [
    'sudo yum update -y && sudo yum install -y docker',
    'sudo systemctl start docker',
    'sudo usermod -aG docker ec2-user',
    'docker run -d -p 8080:80 --name nginx nginx'
]

ssh = paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect(hostname=ssh_host, username=ssh_user, key_filename=ssh_private_key_path)

for command in commands_to_execute:
    stdin, stdout, stderr = ssh.exec_command(command)
    print(stdout.readlines())

ssh.close()

# Open port 8080 to be accessible in browser

sg_list = ec2_client.describe_security_groups(
    GroupNames=['default']
)

port_open = False
for permission in sg_list['SecurityGroups'][0]['IpPermissions']:
    if 'FromPort' in permission and permission['FromPort'] == 8080:
        port_open = True

if not port_open:
    sg_response = ec2_client.authorize_security_group_ingress(
        FromPort=8080,
        ToPort=8080,
        GroupName='default',
        CidrIp='0.0.0.0/0',
        IpProtocol='tcp'
    )

# Scheduled function to check nginx application status and reload if not OK 5x in a row
app_not_accessible_count = 0

def restart_container():
    print('Restarting the application...')
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    ssh.connect(hostname=ssh_host, username=ssh_user, key_filename=ssh_private_key_path)
    stdin, stdout, stderr = ssh.exec_command('docker start nginx')
    print(stdout.readlines())
    ssh.close()
    # reset the count
    global app_not_accessible_count
    app_not_accessible_count = 0
    
    print(app_not_accessible_count)


def monitor_application():
    global app_not_accessible_count
    try:
        response = requests.get(f"http://{ssh_host}:8080")
        if response.status_code == 200:
            print('Application is running successfully!')
        else:
            print('Application Down. Fix it!')
            app_not_accessible_count += 1
            if app_not_accessible_count == 5:
                restart_container()
    except Exception as ex:
        print(f'Connection error happened: {ex}')
        print('Application not accessible at all')
        app_not_accessible_count += 1
        if app_not_accessible_count == 5:
            restart_container()
        return "test"
    
schedule.every(10).seconds.do(monitor_application)  

while True:
    schedule.run_pending()
```

**Exercise 4**

```
import boto3
from operator import itemgetter

ecr_client = boto3.client('ecr')
for repo in ecr_client.describe_repositories()['repositories']:
    print(repo['repositoryName'])

repo_name = "debruits-app"
images = ecr_client.describe_images(
    repositoryName=repo_name
)

image_tags = []

for image in images['imageDetails']:
    image_tags.append({
        'tag': image['imageTags'],
        'pushed_at': image['imagePushedAt']
    })

images_sorted = sorted(image_tags, key=itemgetter("pushed_at"), reverse=True)
for image in images_sorted:
    print(image)

```

**Exercise 5**

The Jenkinsfile is attached to this repo.
