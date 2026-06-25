# Building a Two-Tier Web Application on AWS Using EC2, RDS, and Terraform

*Learn how to deploy a Flask application on Amazon EC2, connect it to Amazon RDS MySQL, and automate the entire infrastructure using Terraform.*

---

## Introduction

Most real applications aren't a single server — they're at least two tiers: something that runs the application code, and something that stores the data. In AWS terms, that usually means an EC2 instance running your app, talking to an RDS database that AWS manages for you.

This project builds exactly that: a Flask web app on EC2 that accepts form submissions and writes them to a MySQL database on RDS — with the EC2 instance's security group explicitly the *only* thing allowed to reach the database. Terraform provisions the whole thing, and the EC2 instance bootstraps itself automatically on first boot — no manual SSH-and-install step required to get the app running.

## Business Problem

A common requirement for any backend service is durable, structured storage — user records, form submissions, application state. Running a database on the same server as your application is tempting for a quick lab, but it couples your data's lifecycle to your compute's lifecycle: restart the server, risk the data. It also makes the database reachable from anywhere the app server is reachable, which is rarely what you want.

The standard fix is a two-tier architecture: a compute tier that's replaceable, and a database tier that's isolated, managed, and only reachable from the application tier itself.

## Solution

This project uses:

- **Amazon EC2** — runs the Flask application
- **Amazon RDS (MySQL)** — a managed database, isolated from public access
- **Security Groups** — the RDS security group only accepts traffic from the EC2 security group, nothing else
- **Terraform** — provisions both tiers and wires the connection between them automatically
- **EC2 user data** — bootstraps the application on first boot: clones the repo, installs dependencies, configures the database connection, and starts the app — without any manual server setup

## Steps We'll Follow

1. **Write the Terraform configuration** — security groups, RDS instance, EC2 instance with a bootstrap script
2. **Deploy with Terraform** — `terraform apply`
3. **Verify the infrastructure** — security groups, RDS instance, EC2 instance
4. **Verify the app in the browser** — submit a test form entry
5. **Verify the database directly** — SSH into EC2, connect to RDS with the MySQL client, and confirm the data landed
6. **Clean up** — `terraform destroy`

## Architecture

```
Browser
   │
   ▼
EC2 Instance (Flask app, port 5000)
   │  security group allows: 22 (SSH), 5000 (app)
   │
   ▼  (only the EC2 security group is allowed in)
RDS MySQL Instance (private, not publicly accessible)
```

## Prerequisites

- An AWS account with programmatic access
- [Terraform](https://developer.hashicorp.com/terraform/downloads) installed and on your PATH
- An EC2 key pair (so you can SSH in for the verification steps)
- A Flask app repo with a `requirements.txt` and `app.py` ready to deploy ([example repo](#))

---

## Step 1: The Terraform Configuration

A few changes from a typical copy-pasted lab config, worth calling out:

- The database password is a Terraform variable, not a hardcoded string — so it's never committed to your repo in plain text.
- The bootstrap script's `chown` path now matches the actual clone path (a mismatch here would silently break permissions on first boot).
- **Before deploying**, verify the AMI ID is valid for *your* region. AMI IDs are region-specific — copying one from a different region's tutorial won't work.

```hcl
# ==============================================================================
# 1. PROVIDER & VARIABLES
# ==============================================================================
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1" # change to your preferred region
}

variable "db_password" {
  description = "Password for the RDS MySQL admin user"
  type        = string
  sensitive   = true
  # Set this via a terraform.tfvars file (gitignored) or TF_VAR_db_password env var —
  # never hardcode it directly in this file.
}

# ==============================================================================
# 2. SECURITY GROUPS (WEB & DATABASE TIER)
# ==============================================================================
resource "aws_security_group" "web_sg" {
  name        = "flask-ec2-security-group"
  description = "Allows SSH and application traffic on port 5000"

  ingress {
    description = "SSH access"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"] # ⚠️ lab-only — restrict to your own IP in anything beyond a lab
  }

  ingress {
    description = "Flask application port"
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

resource "aws_security_group" "db_sg" {
  name        = "mysql-rds-security-group"
  description = "Restricts database traffic to the EC2 web tier only"

  ingress {
    description     = "MySQL traffic, EC2 security group only"
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.web_sg.id] # this is the key isolation: no CIDR block, just the web SG
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# ==============================================================================
# 3. RDS MYSQL DATABASE TIER
# ==============================================================================
resource "aws_db_instance" "mysql_rds" {
  identifier             = "dev-backend-mysql"
  allocated_storage      = 20
  db_name                = "web_db"
  engine                 = "mysql"
  engine_version         = "8.0"
  instance_class         = "db.t3.micro"
  username               = "admin"
  password               = var.db_password
  vpc_security_group_ids = [aws_security_group.db_sg.id]
  skip_final_snapshot    = true
  publicly_accessible    = false # never reachable from outside the VPC

  tags = {
    Name        = "dev-backend-database"
    Environment = "development"
    Project     = "two-tier-foundations"
  }
}

# ==============================================================================
# 4. COMPUTE TIER WITH AUTOMATIC BOOTSTRAPPING
# ==============================================================================
resource "aws_instance" "web_app_server" {
  ami                    = "ami-XXXXXXXXXXXXXXXXX" # 👈 verify a current Ubuntu 22.04 AMI for YOUR region
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.web_sg.id]

  user_data = <<-EOF
              #!/bin/bash
              # 1. Update system packages and install Python tooling
              sudo apt-get update -y
              sudo apt-get install python3-pip python3-dev python3-venv git -y

              # 2. Clone the application repo
              cd /home/ubuntu
              git clone https://github.com/bilalamjad-devops/terraform-aws-ec2-rds.git
              cd terraform-aws-ec2-rds

              # 3. Fix ownership so the ubuntu user can write to this directory
              sudo chown -R ubuntu:ubuntu /home/ubuntu/terraform-aws-ec2-rds

              # 4. Create and activate an isolated virtual environment
              python3 -m venv venv
              source venv/bin/activate

              # 5. Install dependencies inside the venv
              pip3 install -r requirements.txt

              # 6. Write the database connection details for the app to read
              echo "DB_HOST=${aws_db_instance.mysql_rds.address}" > .env
              echo "DB_USER=${aws_db_instance.mysql_rds.username}" >> .env
              echo "DB_PASSWORD=${var.db_password}" >> .env
              echo "DB_NAME=web_db" >> .env

              # 7. Start the Flask app in the background
              nohup python3 app.py > flask.log 2>&1 &
              EOF

  tags = {
    Name        = "dev-app-server"
    Environment = "development"
    Project     = "two-tier-foundations"
  }
}

output "ec2_public_url" {
  value       = "http://${aws_instance.web_app_server.public_ip}:5000"
  description = "Public URL to test the application"
}
```

Set the database password without ever writing it into `main.tf`:

```bash
export TF_VAR_db_password="choose-a-real-password-here"
```

---

## Step 2: Deploy It

```bash
terraform init
terraform plan
terraform apply
```

Confirm with `yes`. Terraform provisions both security groups, the RDS instance, and the EC2 instance — and the EC2 instance bootstraps itself via the `user_data` script the moment it boots, with no manual SSH step needed to get the app running.

<img width="1600" height="900" alt="terraform apply completing successfully" src="https://github.com/user-attachments/assets/317c6fb2-11a2-4883-ad0e-ff368430541b" />

---

## Step 3: Verify the Infrastructure

A few quick checks in the console before testing the app:

- **Security groups** — confirm `flask-ec2-security-group` allows 22 and 5000, and `mysql-rds-security-group` only allows 3306 *from* the web security group — not from any CIDR block.

<img width="1600" height="900" alt="security group rules confirmed in the console" src="https://github.com/user-attachments/assets/d98fb50f-c1f3-4213-bff4-6ae6efb32f3b" />

- **RDS** — confirm the `dev-backend-mysql` instance is `Available`, and that "Publicly accessible" reads **No**.

<img width="1600" height="900" alt="RDS instance available and not publicly accessible" src="https://github.com/user-attachments/assets/cb2f50b5-bbd0-4b2a-a0da-c99e2055d0ca" />

- **EC2** — confirm `dev-app-server` is `Running`, and copy its public IP from the Terraform output (`ec2_public_url`).

<img width="1600" height="900" alt="EC2 instance running with public IP visible" src="https://github.com/user-attachments/assets/74b367fd-f106-466a-b171-2b4a311f2849" />

---

## Step 4: Verify the App in the Browser

Take the `ec2_public_url` value from the Terraform output:

```
http://<EC2_PUBLIC_IP>:5000
```

Open it in a browser, fill in the form (e.g. "Bilal Amjad — Smooth Test"), and submit. A success message should confirm the entry was written to the database.

<img width="1600" height="900" alt="opening the app in the browser via the EC2 public IP" src="https://github.com/user-attachments/assets/e2542e24-2e89-4232-b71b-eb49d0df7b71" />

<img width="1600" height="900" alt="submitting the test form" src="https://github.com/user-attachments/assets/3380a94c-f686-4829-ac7e-d9f5707e2a7d" />

<img width="1600" height="900" alt="success message after form submission" src="https://github.com/user-attachments/assets/f4e2e406-e447-487c-ae3b-98c609f59123" />

<img width="1600" height="900" alt="confirming the app is reachable and responsive" src="https://github.com/user-attachments/assets/9caecf0d-e82f-43e8-954d-7fab0956a723" />

---

## Step 5: Verify the Database Directly

This step proves the data actually landed in RDS, not just that the app *said* it succeeded.

```bash
# 1. SSH into the EC2 instance
ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>

# 2. Move into the application directory
cd ~/terraform-aws-ec2-rds

# 3. Fix ownership if needed (should already be correct from the bootstrap script)
sudo chown -R ubuntu:ubuntu /home/ubuntu/terraform-aws-ec2-rds

# 4. Activate the virtual environment
source venv/bin/activate

# 5. Confirm the RDS endpoint the app is actually using
cat .env

# 6. Install the MySQL client (only needed once)
sudo apt install mysql-client-core -y

# 7. Download AWS's root certificate bundle, needed for the SSL connection
curl -o global-bundle.pem https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem

# 8. Connect to RDS using the endpoint from .env
mysql -h <YOUR_RDS_ENDPOINT_FROM_DOTENV> -P 3306 -u admin -p --ssl-mode=VERIFY_IDENTITY --ssl-ca=./global-bundle.pem
```

Enter the database password you set via `TF_VAR_db_password` when prompted (it won't echo to the screen — that's expected).

Once connected, run:

```sql
-- Select the application's database
USE web_db;

-- View the records submitted through the web form
SELECT * FROM users;

-- Disconnect
EXIT;
```

<img width="1600" height="900" alt="connecting to RDS from the EC2 instance via the MySQL client" src="https://github.com/user-attachments/assets/59fa9f49-0a13-4914-a2c4-b94e2972a7d5" />

<img width="1600" height="900" alt="SQL query result showing the submitted record in the users table" src="https://github.com/user-attachments/assets/e23bf2aa-3315-4e78-850c-da60f7c41e8c" />

Seeing your test submission in the query results is the real proof the two tiers are correctly wired together — the app, the network path, and the database are all working as one system.

---

## Step 6: Clean Up

```bash
terraform destroy --auto-approve
```

<img width="1600" height="900" alt="terraform destroy completing successfully" src="https://github.com/user-attachments/assets/629e4bf9-55eb-48f4-8257-3aa8542bddb6" />

This removes the EC2 instance, the RDS instance, and both security groups. Don't leave an RDS instance running longer than needed — it's billed continuously, unlike Lambda-based projects that only cost money while actually executing.

---

## Production Considerations

This lab prioritizes clarity over hardening. Before pointing anything like this at real traffic:

- Restrict SSH (port 22) to your own IP, not `0.0.0.0/0`
- Move the EC2 instance into a private subnet behind a load balancer or bastion host, rather than exposing it directly
- Use AWS Secrets Manager or SSM Parameter Store for the database password instead of an environment variable on disk
- Enable RDS automated backups and multi-AZ if this were anything beyond a dev environment
- Add HTTPS (a load balancer with a TLS certificate) instead of serving the app directly over HTTP on a custom port

## Conclusion

This project covers a pattern that shows up in nearly every real backend: a compute tier and a database tier, deployed together, with the database deliberately unreachable from anywhere except the application itself. Terraform makes the whole thing reproducible, and the EC2 bootstrap script means the app is running the moment the instance boots — no manual setup step in between.

---

*Code for this project is available on [GitHub](https://github.com/bilalamjad-devops/terraform-aws-ec2-rds) — update the link if your repo name differs.*
