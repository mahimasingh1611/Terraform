# Terraform
notes of terraform

TERRAFORM:

Terraform allows us to create reusable code that can deploy identical set of infrastructure in a repeatable fashion.

Terraform apply: It will read the code that we have written and it will deploy the code in the infrastructure.

terraform supports thousands of providers(AWS, Azure, GCP)
Once we learn terraform core concept, we can write code to create and manage infrastructure across all the providers
Terraform concept remains same irrespective of the platform.

CHOOSING THE RIGHT IAC TOOL
terraform, ansible, heat, chef,puppet , saltstack

IAC is of 2 types: 1) configuration Management 2) Infrastructure Orchestration
We should prefer infrastructure Orchestration tool like cloudformation, terraform etc

Configuration Management tool is used to maintain desired configuration of systems (inside servers)
Example: All server should have Antivirus installed with version 10.0.2

Infrastructure Orchestration is primarily used to create and manage infrastructure environments
Example: Create 2 server with 4 GB RAM ,2 vCPU. Each server should have firewall rules to allow SSH connection from office IPs.

(In Infrastructure Orchestration we are dealing with raw infrastructure not the configuration inside the server)

We can integrate IAC tool(terraform) with configuration management(Ansible) so after the raw infrastructure got created we can create inside the server.
terraform integrate with configuration management tool

EXAMPLE:

resource "aws_instance" "myec2" {
    ami = "ami-00c39f71452c08778"
    instance_type = "t2.micro"
}

#AUTHENTICATION AND AUTHORIZATION
AUTHENTICATION is a process who the user is. It will check the user exist or not
AUTHORIZATION is a process to check that you have the access or not

Terraform needs access credentials with relevant permissions to create and manage the environments

Commands to be used in Terraform:

terraform init: it will download all the plugins that we need to run the code. It will downloaded to terraform directory.
terraform plan: It will show which resources we will be creating
terraform apply: to apply the plan

provider "aws" {
	region : "us-east-1"
	access_key : "bsmnbsmbmjjg"
	private_key : "2kuy8272h2ye8idiuqkhqhdoiqdhi"
}

resource "aws_instance" "myec2" {
	ami = "ami-00c39f71452c08778"
    	instance_type = "t2.micro"
}

resource block describe one or more infrastructure

If you wanna create 2 instances then we need to take the different name of the ec2-instance otherwise it will throw error.

#PROVIDERS MAINTAINERS

Official--> Owned and maintain by Hashicorp,
Partner---> Owned and maintain by Technology company that maintain direct partnership with Hashicorp
Community--> Owned and maintain by individual contributers

Namespaces are used to help users identify the organisation or publisher responsible for the integration

when we need to use required_providers in terraform that are not hashicorp maintained use a new syntax in the required_providers nested block inside the
terraform configuration block

terraform {
required_providers {
digitalocean = {
source = "digitalocean/digitalocean"
	}
}
}

#We will make GitHub repository using terraform

terraform {
required_providers {
digitalocean = {
source = "integration/github"
version = "~> 5.0"
	}
}
}

provider "github"{
token = "sbbwsdjkabkjaxbajhhvsvjquwsyiuqu9quq"
}

resource "github_repository" "example"{
name = "example"
description = "My awesome codebase"
visibility = "public"
}


#TERRAFORM DESTROY
It will destroy all the resources that are created within the folder
command: terraform destroy

If we wanna destroy specific resources then we can use target flag
command: terraform destroy -target aws_instance.myec2

We can also comment out the code so we can destroy that resources.

#STATE FILE
In state file all the resources track is there. If any resources we delete the information that is present in that state file will also be deleted.
Exact data about resources will be stored in state file.

#DESIRED AND CURRENT STATE
It's primary function is to create, modify, and destroy infrastructure resources to match the desired state described in a Terraform configuration.
Terraform tries to ensure that the deployed infrastructure is based on the desired state. If there is a difference between the two, terraform plan present 
a description of the changes necessary to achieve the desired state. 

If in the tf code the it is mentioned the tf instance should be tf.micro but someone from console change it to tf.medium. If we ran the terraform plan again then we can get the tf.micro state again.

#PROVIDER VERSIONING
We should set the version explicitly otherwise terraform will download the latest plugins and it will create the error

How to define the version?
>=1.0, greater than equal to 
<=1.0, less than equal to
~>2.0, Any range in the 2.X range
>=2.0 <=2.30, Any version between 2.10 and 2.30 

Terraform dependency lock file allows us to lock to a specific version of the provider
If a particular provider already has a selection recorded in the lock file, terraform will always reselect that version for installation, even if a new version has become available
You can override that behaviours by adding the --upgrade option when you run terraform init

#TERRAFORM REFRESH
we do not use terraform refresh because terraform plan do it internally. If we do it manually then it might create mess to the state file.

#AUTHENTICATION CONFIGURATION
.aws/config and .aws/credentials--> default path to search for credentials
USERPROFILE/AWS/CONFIG on windows
We can download AWS CLI for storing the credentials on the location

These 4 services we will use:
firewall
EC2
AWS Users
IP Address

When we need to check which ports are opening for that server
command: netstat -ntlp
Whenever we want to connect to the server 
curl public_ip:80
Firewall control both inbound and outbounds connections to and from the server.
Inbound rule: Inbound rules generally controls whomcan connect to the server.
Outbound rule: Which remote system server can connect to

INBOUND AND OUTBOUND RULE:

provider "aws"{
region = "us-east-1"
}

resource "aws_security_group" "allow_tls"{
name = "terraform-firewall"
description = "Managed from Terraform"
}

resource "aws_vpc_security_group_ingress_rule" "allow_tls_ipv4"{
security_group_id = aws_security_group.allow_tls.id
cidr_ipv4 = "0.0.0.0/0"
from_port = 80
ip_protocol = "tcp"
to_port = 80
}

resource "aws_vpc_security_group_egress_rule" "allow_all_traffic_ipv4"{
security_group_id = aws_security_group.allow_tls.id
cidr_ipv4 = "0.0.0.0/0"
ip_protocol = "-1"
}
