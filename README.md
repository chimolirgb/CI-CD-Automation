# **Terraform CI/CD Pipeline with GitHub Actions and GCP**

## **Overview**
This project implements a **CI/CD pipeline** using **GitHub Actions** to automate the deployment and management of Terraform infrastructure on **Google Cloud Platform (GCP)**. The workflow ensures:

 **Linting and validation** of Terraform code before deployment.  
 **Automatic infrastructure deployment** upon a commit to `main`.  
 **Automatic resource teardown** when a branch (except `main`) is deleted.  

---

## **Pipeline Triggers**
The GitHub Actions workflow runs automatically when:

1. **Push to `main` branch** → Deploys infrastructure.
2. **Branch deletion (except `main`)** → Destroys infrastructure.

```yaml
on:
  push:
    branches:
      - main
  delete:
    branches-ignore:
      - main
```


## **Pipeline Jobs**

### **1️⃣ Terraform Deployment Job (`terraform`)**
This job is triggered when a commit is pushed to the `main` branch.

#### **Steps:**
1. **Checkout Repository**
   ```yaml
   - name: Checkout code
     uses: actions/checkout@v4
   ```
2. **Set Up Terraform**
   ```yaml
   - name: Setup Terraform
     uses: hashicorp/setup-terraform@v3
     with:
       terraform_version: 1.6.0
   ```
3. **Authenticate to GCP**
   ```yaml
   - name: Authenticate to GCP
     uses: google-github-actions/auth@v2
     with:
       credentials_json: ${{ secrets.GCP_SA_KEY }}
   ```
4. **Initialize Terraform**
   ```yaml
   - name: Initialize Terraform
     run: terraform init
   ```
5. **Validate Terraform Code**
   ```yaml
   - name: Validate Terraform
     run: terraform validate
   ```
6. **Plan Terraform Changes**
   ```yaml
   - name: Plan Terraform changes
     run: terraform plan -var="project_id=$PROJECT_ID" -var="region=$REGION"
   ```
7. **Apply Terraform Changes (Only on `main`)**
   ```yaml
   - name: Apply Terraform changes (only on push to main)
     if: github.ref == 'refs/heads/main'
     run: terraform apply -auto-approve -var="project_id=$PROJECT_ID" -var="region=$REGION"
   ```



### **2️⃣ Terraform Destroy Job (`destroy`)**
This job is triggered when **a branch is deleted** (except `main`).

#### **Steps:**
1. **Checkout Repository**
   ```yaml
   - name: Checkout code
     uses: actions/checkout@v4
   ```
2. **Set Up Terraform**
   ```yaml
   - name: Setup Terraform
     uses: hashicorp/setup-terraform@v3
     with:
       terraform_version: 1.6.0
   ```
3. **Authenticate to GCP**
   ```yaml
   - name: Authenticate to GCP
     uses: google-github-actions/auth@v2
     with:
       credentials_json: ${{ secrets.GCP_SA_KEY }}
   ```
4. **Initialize Terraform**
   ```yaml
   - name: Initialize Terraform
     run: terraform init
   ```
5. **Destroy Terraform Resources**
   ```yaml
   - name: Destroy Terraform resources
     run: terraform destroy -auto-approve -var="project_id=$PROJECT_ID" -var="region=$REGION"
   ```



## **Why This Approach?**
 **Automates Infrastructure Deployment** – Eliminates manual Terraform execution.  
 **Ensures Code Quality** – Uses Terraform validation before deployment.  
 **Cost Optimization** – Deletes unused resources when branches are removed.  
 **Security Best Practices** – Uses GitHub Secrets to store GCP credentials safely.  


