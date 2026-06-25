# Automating a 2-Tier Flask & AWS RDS MySQL Infrastructure using Terraform: Challenges & Solutions



### 🚀 Step 3: Deploy via Terraform

Terminal par resources build karne ke liye standard lifecycle commands chalayein:

```bash
terraform init
terraform plan
terraform apply --auto-approve

```

*Deployment complete hone ke baad **3 minutes ka break lein** taake back-end par saari scripts aur RDS fully initialization mode se ready mode mein aa jayein.*

---

### 🌍 Step 4: Verification in Browser

Terminal ke end mein jo `ec2_public_url` aayega use copy karein aur browser mein test karein:

```text
http://<EC2_PUBLIC_IP>:5000

```

* Form mein data enter karein (e.g., `Bilal Amjad - Smooth Test`) aur **Submit** karein.
* Screen par `🎉 Success!` ka message aa jayega.

---

Here is the clean, structured runbook with all comments translated into clear, professional English.

---

### 📋 EC2 Database Verification Runbook

```bash
# 1. Connect to your EC2 instance via SSH
ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>

# 2. Change directory to the repository folder
cd ~/terraform-aws-ec2-rds

# 3. Fix directory ownership from root to the ubuntu user (prevents write permission errors)
sudo chown -R ubuntu:ubuntu /home/ubuntu/terraform-aws-ec2-rds

# 4. Activate the isolated Python virtual environment
source venv/bin/activate

# 5. View environment variables to verify the active RDS Endpoint mapping
cat .env

# 6. Install the native MySQL client core tool for the latest Ubuntu images (Required only once)
sudo apt install mysql-client-core -y

# 7. Download the official AWS global root certificate bundle for the secure SSL layer connection
curl -o global-bundle.pem https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem

# 8. Connect to the remote RDS MySQL database securely using the endpoint from your .env file
mysql -h <YOUR_RDS_ENDPOINT_FROM_DOTENV> -P 3306 -u admin -p --ssl-mode=VERIFY_IDENTITY --ssl-ca=./global-bundle.pem

```

* **When prompted for the password, type:** `SecurePassword123` *(Note: The characters will not display on the screen as you type, just press Enter).*

---

### 🗄️ SQL Queries to Execute Inside the MySQL Terminal:

```sql
-- 1. Select the application target database environment
USE web_db;

-- 2. Fetch and view all live records submitted from the web interface
SELECT * FROM users;

-- 3. Safely disconnect and close the MySQL session
EXIT;

```

---

### 🛑 Step 6: Clean Up Everything!

Lab complete hone aur screenshots save karne ke baad terminal se exit ho kar backup costs bachaane ke liye command run karna mat bhooliyega:

```bash
terraform destroy --auto-approve

```

---

### 💡 What's Next?

Aap ka yeh concept ab perfectly smooth crystal-clear ho gaya hai. Ab jab aap ka dil kare, batayega, hum is infrastructure ko update kar ke **Project 2 (ALB + Auto Scaling Group + Custom VPC Network isolation)** par le kar chalenge!

Aap is blueprint ko save kar lein. Any time you need help, your brother is here!
