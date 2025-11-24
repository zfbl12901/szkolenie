# 1. Azure Getting Started

## 🎯 Objectives

- Create a free Azure account
- Understand Azure free credits
- Navigate Azure Portal
- Configure basic security (Azure AD)
- Monitor costs

## 📋 Table of Contents

1. [Create a Free Azure Account](#create-a-free-azure-account)
2. [Understand Free Credits](#understand-free-credits)
3. [Navigate Azure Portal](#navigate-azure-portal)
4. [Azure AD Configuration (Security)](#azure-ad-configuration-security)
5. [Cost Monitoring](#cost-monitoring)

---

## Create a Free Azure Account

### Step 1: Registration

1. **Go to Azure website**
   - URL: https://azure.microsoft.com/free/
   - Click "Start free"

2. **Sign in with Microsoft**
   - Use existing Microsoft account
   - Or create new Microsoft account

3. **Identity verification**
   - Code received by SMS or email
   - Enter verification code

4. **Personal information**
   - Name
   - First name
   - Phone number
   - Country

5. **Phone verification**
   - Automatic call or SMS
   - Enter verification code

6. **Payment method**
   - **Important**: Credit card required but **not charged**
   - Azure gives you $200 credit for 30 days
   - After 30 days: permanent free services
   - You can remove the card later (not recommended)

7. **Final identity verification**
   - Verification by SMS or call
   - Account confirmation

### Step 2: Confirmation

- Confirmation email received
- Azure account active immediately
- Access to Azure Portal
- **$200 credit** available for 30 days

**⚠️ Important**: Don't create multiple accounts with the same credit card (risk of suspension).

---

## Understand Free Credits

### Azure Free Offer

Azure offers **3 types** of free services:

#### 1. $200 Credit (30 days)

**What you can do:**
- Test any Azure service
- Create virtual machines
- Use paid services
- Experiment freely

**Conditions:**
- Valid 30 days after registration
- If credit exhausted before 30 days: services stopped
- After 30 days: switch to permanent free services

#### 2. Free services for 12 months

**Services useful for Data Analyst:**

- **Azure SQL Database**: Free up to 32 GB (12 months)
- **Azure Storage**: 5 GB (12 months)
- **Azure App Service**: 60 minutes/day (12 months)
- **Azure Functions**: 1 million executions/month (always free)

**Conditions:**
- Free for 12 months after registration
- Monthly limits
- Beyond: normal billing

#### 3. Always free services

**Services useful for Data Analyst:**

- **Azure Functions**: 1 million executions/month (always free)
- **Azure Cosmos DB**: 400 RU/s (always free)
- **Azure Active Directory**: 50,000 objects (always free)
- **Azure DevOps**: 5 users (always free)

**Conditions:**
- Free indefinitely
- Monthly limits
- Beyond: billing beyond the limit

### Check Your Credits

1. Go to Azure Portal
2. "Cost Management + Billing"
3. View remaining credits
4. View usage by service

---

## Navigate Azure Portal

### Main Interface

**Key elements:**

1. **Search bar** (top)
   - Quickly search for services
   - Example: type "SQL" to find SQL Database

2. **Azure menu** (☰ icon top left)
   - All Azure services
   - Organized by category
   - Customizable favorites

3. **Notifications** (top right)
   - Alerts and notifications
   - Deployment status

4. **Settings** (top right)
   - Account settings
   - Theme (light/dark)
   - Language

5. **Cloud Shell** (>_ icon top)
   - Terminal in browser
   - PowerShell or Bash
   - Very useful for commands

### Essential Services for Data Analyst

**In Azure menu, search for:**

- **Storage accounts**: Data storage
- **Data Factory**: Cloud ETL
- **SQL databases**: SQL databases
- **Synapse Analytics**: Data warehouse
- **Databricks**: Big Data analytics
- **Functions**: Serverless computing

### First Connection

1. Sign in: https://portal.azure.com/
2. Explore the dashboard
3. Click "All services" to see all services
4. Use search bar to find a service
5. Pin frequent services to dashboard

---

## Azure AD Configuration (Security)

### What is Azure AD?

**Azure AD** (Azure Active Directory) = Identity and access management

- Manage users
- Manage permissions
- Secure access to services
- Multi-factor authentication (MFA)

### Security Best Practices

#### 1. Enable Multi-Factor Authentication (MFA)

**For administrator account:**

1. Go to Azure AD
2. "Users" → Select your account
3. "Multi-factor authentication"
4. Click "Enable"
5. Follow instructions

**⚠️ Important**: Always enable MFA for administrator accounts.

#### 2. Create Azure AD Users (Recommended)

**For team work:**

1. Go to Azure AD
2. "Users" → "New user"
3. Username: `data-analyst@yourdomain.onmicrosoft.com`
4. Temporary password
5. Roles: "User" (default)
6. Create user

#### 3. Azure Roles (RBAC)

**Roles useful for Data Analyst:**

- **Contributor**: Can create and manage resources
- **Reader**: Can only read
- **Storage Account Contributor**: Access to storage accounts
- **SQL DB Contributor**: Access to SQL databases

**Assign a role:**

1. Go to resource (ex: Storage Account)
2. "Access control (IAM)"
3. "Add" → "Add role assignment"
4. Select role
5. Select user

### Recommended Security Policies

1. **Strong passwords**
   - Minimum 12 characters
   - Complexity required

2. **Password expiration**
   - 90 days (recommended)

3. **Account lockout**
   - After 5 failed attempts

---

## Cost Monitoring

### Enable Cost Alerts

**Step 1: Configure alerts**

1. Go to "Cost Management + Billing"
2. "Cost alerts"
3. "New cost alert"
4. Threshold: $5 (recommended)
5. Email notification

**Result**: Email received if costs exceed $5.

### Check Credit Usage

1. "Cost Management + Billing"
2. "Azure credits"
3. View remaining credits
4. View usage by service
5. View expiration date (30 days)

### Azure Cost Management

1. "Cost Management + Billing" → "Cost Management"
2. View costs by service
3. Filter by period
4. Export reports
5. Create budgets

**⚠️ Important**: Check regularly (weekly recommended).

### Tips to Stay Free

1. **Delete unused resources**
   - Stop unused virtual machines
   - Delete empty storage accounts
   - Clean up resource groups

2. **Use free services**
   - Prioritize always-free services
   - Use credits wisely
   - Stop unused services

3. **Create budgets**
   - "Cost Management" → "Budgets"
   - Create $5 budget
   - Automatic alerts

4. **Stop unused services**
   - Virtual machines: stop when not used
   - Databases: stop or pause
   - Storage accounts: delete if empty

### Resource Groups

**Organize your resources:**

1. Create resource group: `rg-data-analyst-training`
2. All training resources in this group
3. Facilitates one-time deletion
4. Facilitates cost management

**Create a resource group:**

1. "Resource groups" → "Add"
2. Name: `rg-data-analyst-training`
3. Region: Choose closest region
4. Create

---

## 📊 Key Points to Remember

1. **Free Azure account**: $200 credit (30 days) + free services
2. **Free credits**: 3 types ($200, 12 months, always free)
3. **Azure AD Security**: Enable MFA, create users
4. **Monitoring**: Cost alerts essential
5. **Stay free**: Delete unused resources, use resource groups

## 🔗 Next Module

Go to module [2. Azure Storage - Data Storage](../02-storage/README.md) to learn how to store data on Azure.

