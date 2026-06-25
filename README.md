
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
