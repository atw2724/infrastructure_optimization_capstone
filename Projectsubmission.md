# DevOps Capstone Infrastructure Optimization Submission


## Section A: Creation of the Cluster
1.	Log onto AWS practice lab to retrieve Access Key, Secret Key, and Security Token
2.	Open AWS Web Console and Create a Directory
3.	Export Access Key, Secret Key, and Security Token for Terraform to use
```
export AWS_ACCESS_KEY_ID="ASIAZZTBKIIAKXATQNGN"
export AWS_SECRET_ACCESS_KEY="GG9CJdpUWUaQ48DJnKgsdXXPwCL1fHYeRv9CIsUk"
export AWS_SESSION_TOKEN="***************"
echo $AWS_ACCESS_KEY_ID
echo $AWS_SECRET_ACCESS_KEY
echo $AWS_SESSION_TOKEN
```

4.	Create aws.tf to define region
```
    provider "aws" {
    region        = "us-east-1"
    }
```

## Section B: Provisioning Automation Using Terraform

1.	Verify Terraform is available
```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ terraform version

Terraform v1.1.6
on linux_amd64

Your version of Terraform is out of date! The latest version
is 1.2.3. You can update by downloading from https://www.terraform.io/downloads.html

```
![image](https://user-images.githubusercontent.com/72522796/175634712-6b921233-2f82-43b2-965d-af8d5f44e3ce.png)

2.	Initialize Terraform

```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v4.20.0...
- Installed hashicorp/aws v4.20.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```
![image](https://user-images.githubusercontent.com/72522796/175635064-e5c7694e-e8c4-452e-92ee-2954da1a4837.png)

3.	Create main.tf to define aws instance and keypair resources 

```
cat main.tf

resource "aws_instance" "rhel" {
  ami           = "ami-052efd3df9dad4825"
  instance_type = "t2.micro"
  key_name      = "${aws_key_pair.generated_key.key_name}"
  tags = {
    Name        = "terraform_instance"
  }
}

output "myEC2IP" {
  value = "${aws_instance.ubuntu.public_ip}"
}

resource "tls_private_key" "example" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

resource "aws_key_pair" "generated_key" {
  key_name   = "mykey2"
  public_key = tls_private_key.example.public_key_openssh

provisioner "local-exec" { # Create "myKey.pem" to your computer!!
    command = "echo '${tls_private_key.example.private_key_pem}' > ./myKey.pem"
  }
}
```
![image](https://user-images.githubusercontent.com/72522796/175636886-0568ef95-6524-4aa9-b67f-6d2134fe5ca9.png)

4.	Run Terraform Plan

```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ terraform plan
```
![image](https://user-images.githubusercontent.com/72522796/175643556-26553f65-e84e-4aa4-afae-cbb1ac5eb016.png)

5.	Run Terraform Apply to deploy EC2 Instance

```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ terraform apply
```
![image](https://user-images.githubusercontent.com/72522796/175644072-e3b4da0d-21e6-4064-9c93-270b629f9750.png)

6.	Verify Instance and Keypair Creation

![image](https://user-images.githubusercontent.com/72522796/175644328-4798d9de-fa46-4466-ba4a-fa8e3f500e83.png)

7.	Update main.tf to add security group

![image](https://user-images.githubusercontent.com/72522796/175644480-b5891ef8-8c08-4e56-88f5-fbb2e992eeaf.png)

8.	Run Terraform Apply and change permissions to verify ability to SSH

```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$  chmod 400 myKey.pem
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$  ssh -i myKey.pem ec2-user@44.206.230.44
```
![image](https://user-images.githubusercontent.com/72522796/175644684-62d28b21-f746-4f84-9f34-dfba54994604.png)

9.	Scale the instances by modifying main.tf with updated count, updated name, and updated value

```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ vi main.tf
resource "aws_instance" "ubuntu" {
  ami           = "ami-052efd3df9dad4825"
  count = 3
  instance_type = "t2.micro"
  key_name      = "${aws_key_pair.generated_key.key_name}"
  vpc_security_group_ids = [aws_security_group.ab_sg.id]
  tags = {
    Name        ="terraform_instance${count.index+1}"
  }
}

output "myEC2IP" {
  value = "${aws_instance.ubuntu.*.public_ip}"
}

```

10.	Run Terraform Apply again to provision the additional instances

```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ terraform apply
```
![image](https://user-images.githubusercontent.com/72522796/175647097-6a1ee61e-e783-45cb-b48c-4828ab1e1ffa.png)

## Section C: Kubernetes Installation on the Master


