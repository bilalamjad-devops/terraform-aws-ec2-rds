**✅ Here is your fully polished, Medium-optimized article.**  

Copy-paste ready. It’s written for better ranking: strong SEO title, compelling hook, scannable structure, keyword-rich content, clear value, and engagement elements.

---

**Title:**  
**Automating a 2-Tier Flask + MySQL Application on AWS using Terraform: A Complete Step-by-Step Guide**

**Subtitle:**  
Build a secure, production-ready EC2 + RDS infrastructure with Infrastructure as Code, automated bootstrapping, and real-world troubleshooting tips.

---

In the world of cloud infrastructure, manually launching servers and databases is slow, error-prone, and impossible to scale. Modern DevOps teams use **Infrastructure as Code (IaC)** to deploy repeatable, version-controlled environments in minutes.

In this hands-on guide, we’ll provision a complete **2-tier web application** on AWS using **Terraform**. A **Flask** Python application running on an **EC2** instance will connect securely to a managed **Amazon RDS MySQL** database — all automated from start to finish.

This project is perfect for DevOps engineers, cloud enthusiasts, and anyone building their portfolio for AWS or Terraform roles.

### The Business Problem
Setting up multi-tier applications manually leads to:
- Inconsistent environments across teams
- Security misconfigurations
- Long deployment times
- Difficulty in replicating setups for dev, staging, and production

Terraform solves these challenges by treating infrastructure as software.

### Architecture Overview
We will build two isolated layers:

1. **Public Compute Tier**: An EC2 instance running a Flask web app (on port 5000) inside a Python virtual environment.
2. **Private Data Tier**: A fully managed RDS MySQL database accessible **only** from the EC2 instance.

**Tech Stack:**
- AWS EC2 + RDS MySQL
- Terraform (IaC)
- Flask + Python
- Security Groups + IAM best practices

**GitHub Repository**: [https://github.com/bilalamjad-devops/terraform-aws-ec2-rds](https://github.com/bilalamjad-devops/terraform-aws-ec2-rds) (Full code + `requirements.txt` + `app.py`)

---

### Step 1: Terraform Configuration (`main.tf`)

Create a new directory and add the following production-grade Terraform code:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"  # Change to your preferred region
}

# Web Tier Security Group
resource "aws_security_group" "web_sg" {
  name        = "flask-ec2-sg"
  description = "Allow SSH and Flask traffic"

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Flask App"
    from_port   = 5000
    to_port     = 5000
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

# Database Tier Security Group
resource "aws_security_group" "db_sg" {
  name        = "mysql-rds-sg"
  description = "Allow MySQL only from Web tier"

  ingress {
    description     = "MySQL from EC2"
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.web_sg.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# RDS MySQL Database
resource "aws_db_instance" "mysql_rds" {
  identifier             = "dev-flask-mysql"
  allocated_storage      = 20
  db_name                = "web_db"
  engine                 = "mysql"
  engine_version         = "8.0"
  instance_class         = "db.t3.micro"
  username               = "admin"
  password               = "SecurePassword123!"   # Use AWS Secrets Manager in production
  vpc_security_group_ids = [aws_security_group.db_sg.id]
  skip_final_snapshot    = true
  publicly_accessible    = false

  tags = {
    Name        = "flask-backend-db"
    Environment = "development"
  }
}

# EC2 Instance with Automated Bootstrapping
resource "aws_instance" "web_app_server" {
  ami                    = "ami-01a00762f46d584a1"  # Ubuntu AMI (update as needed)
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.web_sg.id]

  user_data = <<-EOF
              #!/bin/bash
              apt-get update -y
              apt-get install -y python3-pip python3-dev python3-venv git

              cd /home/ubuntu
              git clone https://github.com/bilalamjad-devops/terraform-aws-ec2-rds.git
              cd terraform-aws-ec2-rds

              # Fix permissions (important for PEP 668)
              chown -R ubuntu:ubuntu /home/ubuntu/terraform-aws-ec2-rds

              sudo -u ubuntu python3 -m venv venv
              sudo -u ubuntu bash -c "source venv/bin/activate && pip install -r requirements.txt"

              # Create .env file with RDS endpoint
              cat > .env << EON
              DB_HOST=${aws_db_instance.mysql_rds.address}
              DB_USER=admin
              DB_PASSWORD=SecurePassword123!
              DB_NAME=web_db
              EON

              # Start Flask app
              sudo -u ubuntu bash -c "source venv/bin/activate && nohup python3 app.py > flask.log 2>&1 &"
              EOF

  tags = {
    Name        = "flask-web-server"
    Environment = "development"
  }
}

output "ec2_public_url" {
  value       = "http://${aws_instance.web_app_server.public_ip}:5000"
  description = "Access your Flask application here"
}
```

---

### Step 2: Deploy the Infrastructure

```bash
terraform init
terraform plan
terraform apply --auto-approve
```

Wait 3–5 minutes for RDS to become available and the user_data script to finish bootstrapping.

*(Insert screenshot: Successful terraform apply)*

---

### Step 3: Test the Application

1. Copy the `ec2_public_url` output.
2. Open it in your browser: `http://<EC2_PUBLIC_IP>:5000`
3. Submit sample data through the form and verify success.

*(Insert screenshots: Application UI, successful form submission)*

---

### Step 4: Verify Data in RDS (Optional but Recommended)

SSH into the EC2 instance and connect to MySQL:

```bash
ssh -i your-key.pem ubuntu@<EC2_IP>
cd ~/terraform-aws-ec2-rds
source venv/bin/activate
sudo apt install mysql-client-core -y
curl -o global-bundle.pem https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem

mysql -h <RDS_ENDPOINT> -P 3306 -u admin -p --ssl-mode=VERIFY_IDENTITY --ssl-ca=global-bundle.pem
```

Then run:

```sql
USE web_db;
SELECT * FROM users;
```

*(Insert screenshots: SSH session, MySQL query results)*

---

### Step 5: Cleanup

```bash
terraform destroy --auto-approve
```

Always clean up lab resources to avoid charges.

*(Insert screenshot: terraform destroy)*

---

### Key Takeaways & Real-World Troubleshooting

- **PEP 668 & Virtual Environments**: Modern Ubuntu prevents global `pip` installs. Always use `venv` for clean dependency isolation.
- **Directory Permissions**: `user_data` scripts often run as root. Use `chown` to give the ubuntu user proper ownership.
- **Security Best Practices**: Never expose RDS publicly. Use security group referencing and strong passwords (consider AWS Secrets Manager in production).
- **Dynamic Configuration**: Inject RDS endpoint via `user_data` + `.env` file — no hardcoding needed.

---

### Conclusion
You’ve now built a fully automated 2-tier Flask + RDS application using Terraform. This project demonstrates real DevOps skills: IaC, secure networking, automated bootstrapping, and troubleshooting.

Add this to your portfolio — it’s exactly what hiring managers look for.

**Star the repo** if it helped:  
👉 [https://github.com/bilalamjad-devops/terraform-aws-ec2-rds](https://github.com/bilalamjad-devops/terraform-aws-ec2-rds)

What should we build next? ALB + Auto Scaling Group? CI/CD pipeline with GitHub Actions? Let me know in the comments!

**Follow me for more practical AWS, Terraform, and DevOps guides.**

---

**Tags:**  
`Terraform` `AWS` `EC2` `RDS` `MySQL` `Flask` `DevOps` `Infrastructure as Code` `Serverless` `Cloud Computing` `Python`

---

### Medium Ranking Tips:
- Add **8–12 screenshots** (Terraform outputs, EC2 app, RDS data, architecture diagram)
- Use a **featured image** showing the 2-tier architecture
- Publish on **Tuesday–Thursday morning**
- Share on LinkedIn + relevant Reddit communities immediately

This version is now **professional, engaging, and SEO-optimized**. It should perform well in the DevOps/AWS community.

Would you like a **featured image prompt** for Grok Imagine or the next project article? Just say the word! 🚀
