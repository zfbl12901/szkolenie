# 1. AWS Getting Started

## 🎯 Objectives

- Create a free AWS account
- Understand AWS Free Tier
- Navigate AWS Console
- Configure basic security (IAM)
- Monitor costs

## 📋 Table of Contents

1. [Create a Free AWS Account](#create-a-free-aws-account)
2. [Understand Free Tier](#understand-free-tier)
3. [Navigate AWS Console](#navigate-aws-console)
4. [IAM Configuration (Security)](#iam-configuration-security)
5. [Cost Monitoring](#cost-monitoring)

---

## Create a Free AWS Account

### Step 1: Registration

1. **Go to AWS website**
   - URL: https://aws.amazon.com/free/
   - Click "Create a Free Account"

2. **Fill out the form**
   - Email
   - Strong password
   - AWS account name

3. **Contact information**
   - Full name
   - Phone number
   - Country

4. **Verification**
   - Code received by SMS
   - Enter verification code

5. **Payment method**
   - **Important**: Credit card required but **not charged**
   - AWS won't charge you as long as you stay within Free Tier
   - You can remove the card later (not recommended)

6. **Identity verification**
   - Automatic call
   - Enter 4-digit code

7. **Support plan**
   - Choose "Basic Plan" (free)
   - Other plans are paid

### Step 2: Confirmation

- Confirmation email received
- AWS account active immediately
- Access to AWS Console

**⚠️ Important**: Don't create multiple accounts with the same credit card (risk of suspension).

---

## Understand Free Tier

### Types of Free Tier

AWS offers **3 types** of free services:

#### 1. Free services for 12 months

**Services useful for Data Analyst:**

- **Amazon EC2**: 750 hours/month (t2.micro)
- **Amazon RDS**: 750 hours/month
- **Amazon Redshift**: 750 hours/month (2 months only)
- **Amazon Elasticsearch**: 750 hours/month

**Conditions:**
- Free for 12 months after registration
- Monthly limits
- Beyond: normal billing

#### 2. Always free services (with limits)

**Services useful for Data Analyst:**

- **Amazon S3**: 5 GB storage (always free)
- **AWS Lambda**: 1 million requests/month (always free)
- **AWS Glue**: 10,000 objects/month (always free)
- **Amazon Athena**: 10 GB data scanned/month (always free)
- **Amazon CloudWatch**: 10 custom metrics (always free)

**Conditions:**
- Free indefinitely
- Monthly limits
- Beyond: billing beyond the limit

#### 3. Short-term free trials

- **Amazon Redshift**: 2 months free
- **Amazon QuickSight**: 1 free user

### Check Your Free Tier

1. Go to AWS Console
2. Menu "Services" → "Billing"
3. Click "Free Tier"
4. View usage by service

---

## Navigate AWS Console

### Main Interface

**Key elements:**

1. **Search bar** (top)
   - Quickly search for services
   - Example: type "S3" to access Amazon S3

2. **Services menu** (top left)
   - All AWS services
   - Organized by category

3. **Region** (top right)
   - Choose AWS region
   - **Recommendation**: Choose closest region
   - Example: `eu-west-3` (Paris) for France

4. **Account name** (top right)
   - Account settings
   - Billing
   - Support

### Essential Services for Data Analyst

**In Services menu, search for:**

- **S3**: Data storage
- **Glue**: Serverless ETL
- **Redshift**: Data warehouse
- **Athena**: SQL queries on S3
- **Lambda**: Serverless processing
- **IAM**: Access management

### First Connection

1. Sign in: https://console.aws.amazon.com/
2. Explore the dashboard
3. Click "Services" to see all services
4. Use search bar to find a service

---

## IAM Configuration (Security)

### What is IAM?

**IAM** (Identity and Access Management) = Access and identity management

- Create users
- Manage permissions
- Secure access to services

### Security Best Practices

#### 1. Enable Two-Factor Authentication (MFA)

**For root account:**

1. Go to IAM
2. Click "Activate MFA"
3. Choose a device (phone)
4. Scan QR code with MFA app
5. Enter verification codes

**⚠️ Important**: Always enable MFA for root account.

#### 2. Create IAM User (Recommended)

**Don't use root account for daily work.**

1. Go to IAM
2. Click "Users" → "Add users"
3. Username: `data-analyst`
4. Access type: "Programmatic access" + "AWS Management Console access"
5. Permissions: "Attach existing policies directly"
   - Select: `PowerUserAccess` (to start)
   - Or create custom permissions
6. Create user
7. **Save credentials** (access key + secret)

#### 3. IAM Groups (Optional)

Create groups to organize users:

1. IAM → "Groups" → "Create group"
2. Name: `DataAnalystGroup`
3. Attach policies
4. Add users to group

### Recommended IAM Policies for Data Analyst

**Essential policies:**

- `AmazonS3FullAccess`: Full access to S3
- `AWSGlueServiceRole`: Access to Glue
- `AmazonRedshiftFullAccess`: Access to Redshift
- `AmazonAthenaFullAccess`: Access to Athena
- `AWSLambdaFullAccess`: Access to Lambda

**⚠️ Principle of least privilege**: Give only necessary permissions.

---

## Cost Monitoring

### Enable Billing Alerts

**Step 1: Enable alerts**

1. Go to "Billing" → "Preferences"
2. Enable "Receive Billing Alerts"
3. Enable "Receive Free Tier Usage Alerts"

**Step 2: Create CloudWatch Alert**

1. Go to CloudWatch
2. "Alarms" → "Create alarm"
3. Metric: "EstimatedCharges"
4. Threshold: $5 (recommended)
5. Notification: Email

**Result**: Email received if costs exceed $5.

### Check Free Tier Usage

1. "Billing" → "Free Tier"
2. View usage by service
3. Check remaining limits
4. Monitor expiration dates (12 months)

### AWS Cost Explorer

1. "Billing" → "Cost Explorer"
2. View costs by service
3. Filter by period
4. Export reports

**⚠️ Important**: Check regularly (weekly recommended).

### Tips to Stay Free

1. **Delete unused resources**
   - Stop unused EC2 instances
   - Delete empty S3 buckets
   - Clean up snapshots

2. **Respect Free Tier limits**
   - Read conditions carefully
   - Monitor usage
   - Set alerts

3. **Use free regions**
   - Some regions offer more free services
   - Check availability

4. **Stop unused services**
   - Redshift: stop cluster when not used
   - EC2: stop instances
   - RDS: stop databases

---

## 📊 Key Points to Remember

1. **Free AWS account**: $200 credit + Free Tier
2. **Free Tier**: 3 types (12 months, always free, trials)
3. **IAM Security**: Enable MFA, create users
4. **Monitoring**: Billing alerts essential
5. **Stay free**: Delete unused resources

## 🔗 Next Module

Go to module [2. Amazon S3 - Data Storage](../02-s3/README.md) to learn how to store data on AWS.

