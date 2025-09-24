# AWS Application Load Balancer Demo (Blue/Red Deployment)

This project demonstrates how to set up an **Application Load Balancer (ALB)** on AWS to distribute traffic between two EC2 instances (Blue & Red servers).  
It also includes optional **Route 53 hosted routing**, where you can configure a fake domain for demo purposes or use a real domain if you own one.  

---

## Prerequisites
Before you begin, make sure you have:
- An **AWS Account** (Free Tier is enough).
- **IAM user** with administrator or EC2/ALB privileges.
- Basic knowledge of launching an **EC2 instance**.

---

## Architecture
The setup includes:
- **2 EC2 Instances**: Blue server & Red server.
- **Target Groups**: One for each server.
- **Application Load Balancer**: Distributes traffic.
- **Route 53 Hosted Zone** (optional): For domain-based routing.

![Project Architecture](images/aws_alb_demo_architecture.png)

---

### 1. Create the S3 Bucket
1. Go to **AWS Console → S3 → Create bucket**.  
2. Enter a **globally unique name**: e.g. `arr-bucket-123456`.  
3. Region: **us-east-1 (N. Virginia)** to match the lab setup.  
4. Keep **Block Public Access = ON** (recommended).  
   - EC2 will use an IAM role to fetch files (not public objects).  
5. Click **Create bucket**.  

![S3 Bucket](images/mybucket.jpg)

---

### 2. Upload the Website Code
1. Prepare your local project so that it contains two folders: `red/` and `blue/`, each with its own `index.html`.  
2. In the **S3 Console**, open your bucket → click **Upload**.  
3. Make sure you only upload correct files & folders

![Bucket Files](images/s3files.jpg)

---

## 3. Get Your Bucket ARN
1. In **S3 console** → open your bucket → go to **Properties**.  
2. Copy the ARNs: 
**For example:**
arn:aws:s3:::arr-bucket-123456 (bucket)  
arn:aws:s3:::arr-bucket-123456/* (all objects inside)  

![Bucket ARN](images/bucketarn.jpg)

---

## 4. Create IAM Policy for S3 Access
1. Create a JSON file `bucket-permissions.json` with:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "YOUR-BUCKET-ARN/*"
        }
    ]
}
```
➡ Replace `YOUR-BUCKET-ARN` with your actual bucket name.

## Steps to create the IAM Policy

1. Go to **IAM → Policies → Create Policy → JSON tab**.  
2. Paste the JSON above.  
3. Name it: **S3-ARR-Policy**.  

![IAM Policy Created](images/iampolicy.jpg)

## CLI Method

```bash
aws iam create-policy --policy-name S3-ARR-Policy --policy-document file://bucket-permissions.json
```

---

## 5. Create IAM Role for EC2

1. Go to **IAM → Roles → Create role**.  
2. Select **AWS service → EC2**.  
3. Attach the policy: **S3-ARR-Policy**.  
4. Name it: **S3-ARR-Role**.  

![EC2 IAM Role](images/ec2role.jpg)


## CLI Method 

```bash
aws iam create-role --role-name S3-ARR-Role --assume-role-policy-document file://ec2-trust-policy.json
aws iam attach-role-policy --role-name S3-ARR-Role --policy-arn arn:aws:iam::123456789012:policy/S3-ARR-Policy
```

---

## 6. Prepare User Data Scripts

Name them: **user-data-red** and **user-data-blue**.  

Replace `YOUR-BUCKET-NAME` with your actual bucket name.  

Example commands:  

```bash
aws s3 cp --recursive s3://arr-bucket-123456/red /var/www/html/red
aws s3 cp --recursive s3://arr-bucket-123456/blue /var/www/html/blue
```

---

# 7. Launch EC2 Instances (Red & Blue)

1. Go to **EC2 → Instances → Launch Instance**.

2. **Choose AMI:**
   - Select **Amazon Linux 2 AMI** (Free tier eligible).

3. **Instance Type:**
   - Choose **t2.micro**.

4. **Key Pair:**
   - **Proceed without a key pair** or create a **key pair**.

5. **Network Settings:**
   - Allow **HTTP (80)** and **SSH (22)**.

6. **Advanced Details:**
   - IAM Instance Profile: Select **S3-ARR-Role**.
   - User Data: Paste the contents of:
     - `user-data-red` (for the Red instance), or
     - `user-data-blue` (for the Blue instance).

7. **Launch Instances:**
   - Launch **2 instances**: one Red and one Blue.

![Launched EC2](images/launcedec2.jpg)

---

# 8. Create Target Groups

1. Go to **EC2 → Target Groups → Create target group**.  
2. **Choose Instances** as target type.  

### Target Group 1 (Red)
- **Name:** `tg-red`  
- **Protocol:** HTTP  
- **Port:** 80  
- **Register:** Red instance  

### Target Group 2 (Blue)
- **Name:** `tg-blue`  
- **Protocol:** HTTP  
- **Port:** 80  
- **Register:** Blue instance  

![Target Groups](images/createdtargetgrps.jpg)
---

