### AWS Playbook Tasks:

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