Ansible AWS VPC Highly-Available Wordpress
----------------------
There's a blog post that I wrote to go along with this. [Check it out!]

The purpose of Ansible AWS VPC Highly-Available Wordpress(AAVHAW) is to create a fully operational AWS VPC infrastructure(subnets,routeing tables,igw etc), it will also create everything that need to be for creating EC2 and RDS instances (security key, security group, subnet group).

It will also create the Elastic Load Balancer and add the EC2 instance(s) automatically that were created using this playbook as well as creating the Route53 entry for this site and add the ELB alias to it. 

Beside that, this playbook will also run the essential role(updating and patching the OS, configuring NTP,creating users etc) and deploy the wordpress on them, that will be fault tolerant and highly available.

**NOTE:** The part of the play, 'webserver.yml', is not idempotent. Every time it is run, it will create a new EC2 instances.

### AAVHAW Playbook Tasks:

- Create 1 x VPC with 3 x VPC subnets(2 x public and 1 x private) in differrent AZ zones one AWS region
- Create the AWS key pair with the ansible host's login user's public key
- Create 1 x security group for each(Webservers,RDS and ELB)
- Provision 2 x EC2 instances(Ubuntu 14.04 LTS) in 2 different AZ
- Provision 1 x RDS instance in private subnet
- Launch and configure public facing VPC ELB (cross_az_load_balancing) and attach VPC subnets
- Register EC2 instances on ELB
- Install essential and webservers role on both instances
- Take the ELB dnsname and register/create dns entry in Route53

All informations about VPC, Webserver, RDS, ELB, Route53 are defined in their respective files (vpc.yml,webserver.yml,rds.yml,elb.yml,route53 etc) for both variables and tasks.

### Requirements:

- Ansible
- boto
- AWS admin access

### Tools Used:
```shell
ansible --version
ansible 1.9.2
  configured module search path = None


python
Python 2.7.9 (default, Apr  2 2015, 15:33:21) 
[GCC 4.9.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import boto
>>> boto.Version
'2.38.0'
>>>
```
### AWS credentials:

Ansible uses python-boto library to call AWS API, and boto needs AWS credentials in order to perform all the functions. There are many ways to configure your AWS credentials. The easiest way is to crate a .boto file under your user home directory:
```shell
vim ~/.boto
```
Then add the following:
```shell
[Credentials]
aws_access_key_id = <your_access_key_here>
aws_secret_access_key = <your_secret_key_here>
```

### To use this Role:

Edit the vars files inside the `aws/vars` directory as per your requirement, for example `vpc.yml` file inside the `aws/vars` directory:
```yaml
---
 # Variables for VPC
 vpc_name: rbgeek
 vpc_region: eu-west-1 # Ireland
 vpc_cidr_block: 172.25.0.0/16
 public_cidr_1: 172.25.10.0/24
 public_az_1: "{{ vpc_region }}a"
 public_cidr_2: 172.25.20.0/24
 public_az_2: "{{ vpc_region }}b"
 private_cidr: 172.25.30.0/24
 private_az: "{{ vpc_region }}c"

 # Please don't change the variables below, until you know what you are doing
 #
 # Subnets Defination for VPC
 vpc_subnets:
   - cidr: "{{ public_cidr_1 }}" # Public Subnet-1
     az: "{{ public_az_1 }}"
     resource_tags: { "Name":"{{ vpc_name }}-{{ public_az_1 }}-public-subnet" }
   - cidr: "{{ public_cidr_2 }}" # Public Subnet-2
     az: "{{ public_az_2 }}"
     resource_tags: { "Name":"{{ vpc_name }}-{{ public_az_2 }}-public-subnet" }
   - cidr: "{{ private_cidr }}" # Private Subnet
     az: "{{ private_az }}"
     resource_tags: { "Name":"{{ vpc_name }}-{{ private_az }}-private-subnet" }

 # Route table(s) for Subnets inside the VPC
 #
 # Routing Table for Public Subnet
 public_subnet_rt:
   - subnets:
       - "{{ public_cidr_1 }}"
       - "{{ public_cidr_2 }}"
     routes:
       - dest: 0.0.0.0/0
         gw: igw
```
After editing all the vars files as per requirements, run this command:
```shell
ansible-playbook -i hosts site.yml
```
### AWS Regions:

Please refer this AWS Region Chart for help

![Please refer this AWS Region Chart](http://s14.postimg.org/my8ap29sh/AWSRegions.jpg)
[Check it out!]:https://rbgeek.wordpress.com/2015/08/03/highly-available-wordpress-installation-inside-aws-vpc-using-ansible/
