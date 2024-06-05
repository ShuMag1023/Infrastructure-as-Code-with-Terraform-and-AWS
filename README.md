# Infrastructure-as-Code-with-Terraform-and-AWS
Let's set up an Infrastructure as Code (IaC) project using Terraform to provision and manage resources on AWS. This project will involve creating a Terraform configuration to provision an AWS EC2 instance and deploy a simple web application.


Let's set up an Infrastructure as Code (IaC) project using Terraform to provision and manage resources on AWS. This project will involve creating a Terraform configuration to provision an AWS EC2 instance and deploy a simple web application.

### Project Outline

1. **Setup GitHub Repository:**
   - Create a GitHub repository for the Terraform configuration files.

2. **Write Terraform Configuration:**
   - Write Terraform scripts to provision an AWS EC2 instance and deploy a simple web application.

3. **Jenkins Pipeline for Deployment:**
   - Configure a Jenkins pipeline to run the Terraform scripts.

### Detailed Steps

#### Step 1: Setup GitHub Repository

1. Create a new repository on GitHub (e.g., `terraform-aws-iac`).
2. Initialize the repository with the necessary Terraform configuration files.

#### Step 2: Write Terraform Configuration

Create Terraform configuration files to provision an AWS EC2 instance and deploy a web application.

**Directory Structure:**
```
terraform-aws-iac/
├── main.tf
├── variables.tf
├── outputs.tf
└── userdata.sh
```

**main.tf:**
```hcl
provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"  # Amazon Linux 2 AMI
  instance_type = "t2.micro"

  key_name = var.key_name

  user_data = file("userdata.sh")

  tags = {
    Name = "WebServer"
  }

  vpc_security_group_ids = [aws_security_group.web_sg.id]
}

resource "aws_security_group" "web_sg" {
  name_prefix = "web-sg"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

output "instance_ip" {
  value = aws_instance.web.public_ip
}
```

**variables.tf:**
```hcl
variable "key_name" {
  description = "The name of the SSH key pair"
  type        = string
}
```

**outputs.tf:**
```hcl
output "web_instance_public_ip" {
  description = "The public IP address of the web instance"
  value       = aws_instance.web.public_ip
}
```

**userdata.sh:**
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd

echo "Hello, World from $(hostname -f)" > /var/www/html/index.html
```

#### Step 3: Jenkins Pipeline for Deployment

Create a Jenkins pipeline to automate the deployment process using Terraform.

**Jenkinsfile:**
```groovy
pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-username/terraform-aws-iac.git'
            }
        }
        stage('Init Terraform') {
            steps {
                sh 'terraform init'
            }
        }
        stage('Plan Terraform') {
            steps {
                sh 'terraform plan -var "key_name=your-key-pair-name"'
            }
        }
        stage('Apply Terraform') {
            steps {
                sh 'terraform apply -var "key_name=your-key-pair-name" -auto-approve'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
```

### Configuration Steps

1. **AWS Credentials:**
   - Store your AWS credentials (`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`) in Jenkins using the Credentials Plugin.

2. **SSH Key Pair:**
   - Ensure you have an AWS SSH key pair. The key pair name should be specified in the Terraform variable `key_name`.

3. **Create a Jenkins Pipeline Job:**
   - Open Jenkins and create a new pipeline job.
   - In the job configuration, set the pipeline definition to "Pipeline script from SCM".
   - Configure the SCM to use your GitHub repository and point to the `Jenkinsfile`.

4. **Run the Pipeline:**
   - Trigger the Jenkins job manually or set up a webhook to trigger the job on every push to the repository.

### Conclusion

By following these steps, you will have created an IaC setup using Terraform to provision AWS resources and automated the deployment using Jenkins. This project demonstrates key DevOps practices such as infrastructure provisioning, configuration management, and continuous deployment.
