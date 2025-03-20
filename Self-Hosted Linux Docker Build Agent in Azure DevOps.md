# **Guide to Configuring a Self-Hosted Docker Build Agent on Windows**

## **Overview**
This guide provides a step-by-step process to configure a **Windows-based self-hosted Docker agent** for Azure DevOps. The agent will be containerized using Docker and registered in **Azure DevOps Agent Pools** to run build jobs inside a Windows container.

---

## **Step 1: Create an Agent Pool in Azure DevOps**

1. Go to **Azure DevOps Dashboard** → [Azure DevOps](https://dev.azure.com/).
2. Select your project.
3. Navigate to **Project Settings** (bottom left corner).
4. Click on **Agent Pools**.
5. **Create a new agent pool**:
   - **Name:** `myWindowsAgentPool` (or any name of your choice).
   - **Grant access permission to all pipelines** → ✅ **Enable this option**.
6. Click **Create**.

---

## **Step 2: Install Docker on Windows VM**

### **Why Enable Hyper-V and Containers Feature?**
- **Hyper-V** is required for **virtualization**, which allows running **Windows containers** on a VM.
- **Containers Feature** enables support for **Windows-based Docker containers**, allowing applications to be packaged and run in an isolated environment.
- Without these features, **Docker cannot create and manage containers on Windows.**

### **Steps to Install Docker**
1. **Login to the Windows VM** where the Docker build agents will run.
2. **Open PowerShell as Administrator** and run:

```powershell
# Enable Hyper-V and Containers feature
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All -NoRestart
Enable-WindowsOptionalFeature -Online -FeatureName Containers -All -NoRestart

# Restart the system
Restart-Computer -Force
```

3. **Install Docker CLI**:

```powershell
Invoke-WebRequest -Uri "https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe" -OutFile "DockerInstaller.exe"
Start-Process -FilePath "DockerInstaller.exe" -ArgumentList "/quiet" -Wait
```

4. **Verify Docker installation**:

```powershell
docker --version
```

5. **Switch to Windows Containers**:

```powershell
& 'C:\Program Files\Docker\Docker\DockerCli.exe' -SwitchDaemon
```

---

## **Step 3: Clone the Repository Containing Dockerfile**
```powershell
git clone https://github.com/akannan1087/ado-docker-agent-repo.git
```

Navigate into the cloned directory:
```powershell
cd ado-docker-agent-repo/azp-agent-in-docker/
```

---

## **Step 4: Create a Windows-Based Dockerfile**

### **What Does the Dockerfile Do?**
- **Uses Windows Server Core as the base image.**
- **Sets environment variables** for Azure DevOps agent registration.
- **Installs PowerShell, Azure CLI, and Azure DevOps dependencies.**
- **Creates the necessary agent directories.**
- **Downloads, configures, and runs the Azure DevOps agent.**

### **Dockerfile Explained**
```dockerfile
# Use Windows Server Core as base image
FROM mcr.microsoft.com/windows/servercore:ltsc2022

# Set environment variables for Azure DevOps
ENV AZP_URL=https://dev.azure.com/YOUR_ORG
ENV AZP_TOKEN=YOUR_PERSONAL_ACCESS_TOKEN
ENV AZP_POOL=myWindowsAgentPool
ENV AZP_AGENT_NAME=docker-windows-agent

# Install PowerShell and required dependencies
SHELL ["powershell", "-Command"]
RUN Install-PackageProvider -Name NuGet -Force; `
    Install-Module -Name Az -AllowClobber -Scope AllUsers -Force

# Create working directory
RUN mkdir C:\azp
WORKDIR C:\azp

# Download and extract Azure DevOps Agent
RUN Invoke-WebRequest -Uri https://vstsagentpackage.azureedge.net/agent/3.225.0/vsts-agent-win-x64-3.225.0.zip -OutFile agent.zip; `
    Expand-Archive -Path agent.zip -DestinationPath C:\azp

# Configure and run the agent
CMD ["powershell", "./config.cmd --unattended --url $env:AZP_URL --auth pat --token $env:AZP_TOKEN --pool $env:AZP_POOL --agent $env:AZP_AGENT_NAME --replace; ./run.cmd"]
```

---

## **Step 5: Build the Docker Image**
### **Where Will the Image Be Built?**
- The **Docker image** will be built **locally on the Windows VM** where Docker is installed.
- It is stored in **Docker's local image registry**.
- To check stored images, run:
  ```powershell
  docker images
  ```

Run the following command to **build the Docker image**:

```powershell
docker build --tag "azp-agent:windows" --file Dockerfile .
```

---

## **Step 6: Run the Windows-Based Docker Agent**
### **Why Don’t We Need to Download the Agent Every Time?**
- Since the image **already includes** the Azure DevOps agent, **you don’t need to manually download it** after VM creation.
- Once the VM starts, the **Docker container will automatically register itself as an agent** in Azure DevOps.

Run the following command to **start the container and register the agent**:

```powershell
docker run -e AZP_URL="https://dev.azure.com/YOUR_ORG" `
            -e AZP_TOKEN="XXXX" `
            -e AZP_POOL="myWindowsAgentPool" `
            -e AZP_AGENT_NAME="myWindowsDockerBuildAgent" `
            --name "azp-agent" `
            --restart always `
            -d azp-agent:windows
```

Verify that the **Docker container is running**:
```powershell
docker ps
```

---

## **Step 7: Verify the Agent in Azure DevOps**
1. Go to **Azure DevOps → Project Settings → Agent Pools**.
2. Click on **myWindowsAgentPool**.
3. Click on **Agents**.
4. You should see **myWindowsDockerBuildAgent** in the list.

---

## **Step 8: Use the Windows Docker Agent in a DevOps Pipeline**
### **1. YAML-Based Pipeline**
Use the following YAML code to run a **Maven build on the self-hosted Windows agent**:

```yaml
trigger:
- main

pool:
  name: myWindowsAgentPool

stages:
- stage: Build
  displayName: Build stage
  jobs:
  - job: MavenPackageAndPublishArtifacts
    displayName: Maven Package and Publish Artifacts
    steps:
    - task: Maven@3
      displayName: 'Maven Package'
      inputs:
        mavenPomFile: 'MyWebApp/pom.xml'
```

---

🚀 **Now, your Windows Docker agent is successfully set up and running for Azure DevOps pipelines!** 🚀
