# GitHub Actions CI/CD Deployment to AWS EC2

This project demonstrates a complete CI/CD pipeline using GitHub Actions. It provisions an AWS EC2 instance using Terraform and deploys a multi-container application (two simple Flask apps) using Docker Compose.

## Project Structure

- **`.github/workflows/`**: Contains the GitHub Actions workflows.
  - `1-provision-ec2.yml`: Provisions or destroys the AWS infrastructure.
  - `2-deploy-apps.yml`: Deploys the application code to the EC2 instance using SSH/SCP and Docker Compose.
- **`terraform/`**: Contains the Terraform configuration to provision the EC2 instance, security groups, and key pairs in AWS.
- **`app1/` & `app2/`**: These are simple Python Flask applications, each having their own `Dockerfile`.
- **`docker-compose.yml`**: Docker Compose configuration to build and run both Flask applications on the EC2 instance.

## Prerequisites

To run these workflows, you need to configure the following **Repository Secrets** in your GitHub repository (`Settings` -> `Secrets and variables` -> `Actions`):

| Secret Name | Description |
| :--- | :--- |
| `AWS_ACCESS_KEY_ID` | Your AWS Access Key ID. |
| `AWS_SECRET_ACCESS_KEY` | Your AWS Secret Access Key. |
| `AWS_REGION` | The AWS region to deploy to (e.g., `us-east-1`). |
| `EC2_SSH_PRIVATE_KEY` | The private SSH key used to connect to the EC2 instance. |
| `EC2_SSH_PUBLIC_KEY` | The corresponding public SSH key to be injected into the EC2 instance by Terraform. |

## How to Deploy

The deployment process is split into two distinct workflows.

### Step 1: Provision Infrastructure (Terraform)

1. Go to the **Actions** tab in your GitHub repository.
2. Select the **1 - Provision EC2 Instance (Terraform)** workflow.
3. Click **Run workflow**. 
   - Leave the action as `apply` (default) to provision the resources.
   - This will execute Terraform and create an EC2 instance tagged as `DockerAppServer` in your default VPC, along with a Security Group allowing traffic on ports 22, 8081, and 8082.

### Step 2: Deploy Applications (Docker Compose)

Once the infrastructure is successfully provisioned, you can deploy the applications. 

1. Go to the **Actions** tab in your GitHub repository.
2. Select the **2 - Deploy Applications (Docker Compose)** workflow.
3. Click **Run workflow** (or simply push a change to the `main` branch to trigger it automatically).
   - This workflow will discover the public IP of the EC2 instance provisioned in Step 1.
   - It will copy the application files and `docker-compose.yml` over to the instance using SCP.
   - Finally, it will SSH into the instance to build and start the Docker containers.

## Accessing the Applications

Once deployed, you can access the applications in your web browser using the public IP of the EC2 instance:

- **App 1:** `http://<EC2_PUBLIC_IP>:8081`
- **App 2:** `http://<EC2_PUBLIC_IP>:8082`

*(You can find the `EC2_PUBLIC_IP` by looking at the output of the "Get EC2 Public IP" step in the deploy workflow logs, or via the AWS Console).*

## Cleanup

To avoid ongoing AWS charges, you can destroy the provisioned infrastructure when you are done:

1. Go to the **Actions** tab.
2. Select the **1 - Provision EC2 Instance (Terraform)** workflow.
3. Click **Run workflow**.
4. Change the action dropdown from `apply` to `destroy` and run the workflow.
