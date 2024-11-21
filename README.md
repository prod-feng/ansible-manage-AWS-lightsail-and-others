# ansible-manage-AWS-lightsail-and-others
Rocky Linux 9.4. miniforge/3-24. Python3.12

## 1. Install ansible, using miniforge:
   ```
   pip install ansible
   ```
## 2. Install awscli2
   ```
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   ./aws/install --bin-dir /home/feng/aws/bin --install-dir /home/feng/aws/aws-cli --update
   ```
   This install will also include the boto3 package too. If you use other way to install awscli, you might need to manually "pip install boto3".

## 3. On the AWS webportal, set the IAM account: policy, user, group, etc. 

   This is very important, through which you can can use awscli and ansible to interactively communicate with AWS.
   
   On the top right of the dashboard, click the icon with your name and down arrow, choose the "Security Credentials" menu to open the "Identity and Access Management" webpage.

## 3.1 Create a new Policy. Choose the actions you need. I chose all Read actions, for Write action, for test purpose, I only choose:
   ```
   CreateInstances
   CreateKeyPair
   DeleteInstance
   DeleteKeyPair
   ImportKeyPair
   RebootInstance
   ```
   When add new Policy, choose the service you want for this work: Lighsail. Or others, like S2, EC2, etc.

   After done, you can click "JASON" to cross check all options you have chosen.

## 3.2 Add a new gourp, say "lightsail-admin", and  bind the above policy to it.

## 3.3 Add a new user and get the keys for Lightsail access:

   like "feng", and make it be a memeber of the aboce group.
   
   During this step, AWS will generate a ## csv ### file(e.g. feng_accessKeys.csv) that you need to download and save, it contains the most important secret key and access key you will need for the next steps.

## 3.4 Configure the local awscli:
   ```
   /home/feng/aws/bin/aws configure
   ```
   open the csv file you donwloaded, and type the keys there(Make sure you choose the right region, and later the zone, like us-east-2 and us-east-2a):

   ```
    AWS Access Key ID [****************FSMV]: 
    AWS Secret Access Key [****************HOYa]: 
    Default region name [us-east-2]: 
    Default output format [None]: 
   ```

   Now you should be able to use awscli to connect to your AWS/Lightsail.
```
   /home/feng/aws/bin/aws --debug lightsail get-instance --instance-name mytestweb
```

## 3.5 Generate SSH key on local machine, and import it to the new Lightsail instance, so you can use ssh command to login later.
```
ssh-keygen -q -t rsa -b 2048 -N '' -f ~/.ssh/myweb

/home/feng/aws/bin/aws lightsail import-key-pair --key-pair-name myweb --public-key-base64 file://~/.ssh/myweb.pub
```
## 4. Ansible configuration.

## 4.1 Add a hosts file with content:
   ```
[local]
  localhost
   ```
## 4.2 Add a YAML file:

```
###lightsail.yaml
 - hosts: local
   connection: local
   tasks:
     - lightsail:
         state: present
         name: mywebtest
         region: us-east-2
         zone: us-east-2a
         blueprint_id: ubuntu_22_04
         bundle_id: micro_2_0
         key_pair_name: myweb
         wait_timeout: 500
       register: myweb
     - debug: msg="{{ myweb.instance.name }}'s IP is {{ myweb.instance.public_ip_address }}."
       when: myweb.instance.public_ip_address is defined
     - lightsail:
         state: absent
         name: mywebtest
         region: us-east-2
       tags: [ 'never', 'destroy' ]
```

## 4.3 Now run the ansible:
```
$ ansible-playbook -i hosts lightsail.yaml
```
From the output, you will find the public IP of this instance:

## 4.4 SSH to the node, or else

```
ssh -i ~/.ssh/myweb  ubuntu@xx.xxx.xx.xx
```

Done.
