


Steps:



## Step 1 — Deploy the Infrastructure

terraform apply

## Step 2 — Verify the Infrastructure

 - sg
 - rds
 - ec2

## Step 3 — Test the Complete Workflow

## Step 4 - Clean Up Resources

Conclusion


---

## Step 1 — Deploy the Infrastructure

terraform apply

<img width="1600" height="900" alt="ec2-rds 3" src="https://github.com/user-attachments/assets/317c6fb2-11a2-4883-ad0e-ff368430541b" />


## Step 2 — Verify the Infrastructure

 - sg
 - rds
 - ec2

<img width="1600" height="900" alt="ec2-rds 5" src="https://github.com/user-attachments/assets/d98fb50f-c1f3-4213-bff4-6ae6efb32f3b" />

<img width="1600" height="900" alt="ec2-rds 6" src="https://github.com/user-attachments/assets/cb2f50b5-bbd0-4b2a-a0da-c99e2055d0ca" />

<img width="1600" height="900" alt="ec2-rds 7" src="https://github.com/user-attachments/assets/74b367fd-f106-466a-b171-2b4a311f2849" />


## Step 3 — Test the Complete Workflow

<img width="1600" height="900" alt="ec2-rds 8" src="https://github.com/user-attachments/assets/e2542e24-2e89-4232-b71b-eb49d0df7b71" />


<img width="1600" height="900" alt="ec2-rds 10" src="https://github.com/user-attachments/assets/3380a94c-f686-4829-ac7e-d9f5707e2a7d" />

<img width="1600" height="900" alt="ec2-rds 12" src="https://github.com/user-attachments/assets/f4e2e406-e447-487c-ae3b-98c609f59123" />


<img width="1600" height="900" alt="ec2-rds 13" src="https://github.com/user-attachments/assets/9caecf0d-e82f-43e8-954d-7fab0956a723" />



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

<img width="1600" height="900" alt="ec2-rds 15" src="https://github.com/user-attachments/assets/59fa9f49-0a13-4914-a2c4-b94e2972a7d5" />

<img width="1600" height="900" alt="ec2-rds 19" src="https://github.com/user-attachments/assets/e23bf2aa-3315-4e78-850c-da60f7c41e8c" />



## Step 4 - Clean Up Resources

<img width="1600" height="900" alt="ec2-rds 21" src="https://github.com/user-attachments/assets/629e4bf9-55eb-48f4-8257-3aa8542bddb6" />


Conclusion













