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

### 📊 Step 5: Inside the Database Verification

Ab data check karne ke liye apne EC2 server ke andar enter ho kar ye simple flow chalayein:

```bash
# 1. Server mein enter hon
ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>

# 2. Lab folder mein shift hon
cd terraform-aws-ec2-rds

# 3. Environment ko activate kar ke dynamic env parameters check karein
source venv/bin/activate
cat .env

# 4. Latest Ubuntu images ke mutabik client install karein (Sirf ek baar)
sudo apt install mysql-client-core -y

# 5. AWS Certificate download karein secure layer connection ke liye
curl -o global-bundle.pem https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem

# 6. Database connect karein (Endpoint custom .env se fetch karein)
mysql -h <YOUR_RDS_ENDPOINT_FROM_DOTENV> -P 3306 -u admin -p --ssl-mode=VERIFY_IDENTITY --ssl-ca=./global-bundle.pem

```

*Password enter karein:* `SecurePassword123`

**MySQL terminal ke andar ye queries run karein:**

```sql
USE web_db;
SELECT * FROM users;

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
