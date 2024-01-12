# Private-ACR-with-self-hosted-Agent
**Architecture**-
<img width="517" alt="image" src="https://github.com/shubhamagrawal17/Private-ACR-with-self-hosted-Agent/assets/24695227/e17f44f4-9a5f-40a1-8172-ae7131d4b162">


**Steps**-
-  First create the Service connection from azure devops to Azure cloud using managed identity or service principle.
-  Create the Agent pool from the project setting.
-  Now create the 2 virtual network (one for self-hosted agent and another for private ACR).
  <img width="661" alt="image" src="https://github.com/shubhamagrawal17/Private-ACR-with-self-hosted-Agent/assets/24695227/f2c96d0f-16f8-4661-ba9b-e5978e638332">

-  Create the virtual network peering between them.
  <img width="816" alt="image" src="https://github.com/shubhamagrawal17/Private-ACR-with-self-hosted-Agent/assets/24695227/a113a8b4-5d25-449c-9695-c1ecccc08418">

-  Create the agent VM (windows or linux)as per your requirement and during vm creation create the ssh key so we can login to vm.
-  Once VM is created copy the SSH key in .ssh folder, if you are using windows vm a client then location will be (C:\Users\shubhamagrawal01\.ssh).
-  Connect to VM using this command - ssh -i ~/.ssh/agent-vm_key.pem shubham@20.204.58.216
-  Create the PAT token for authentication.
-  Once you login to vm then run the following commands to registed the vm as a agent.
-  Create the Agent - mkdir myagent && cd myagent
-  Download the agent - wget https://vstsagentpackage.azureedge.net/agent/3.232.1/vsts-agent-linux-x64-3.232.1.tar.gz
-  extract the file - tar zxvf vsts-agent-linux-x64-3.232.1.tar.gz
-  List the files in the directory after extracting - ls -al
-  Run the command: ./config.sh
-  Accept the Team Explorer Everywhere license agreement now? Type Y and enter
-  Enter server URL > https://dev.azure.com/yourorganization
-  Enter authentication type (press enter for PAT) > PAT
- Enter personal access token
- Enter Agent pool - which you have created 
- Enter Agent name --> myBuildAgent
- Enter work folder > enter
- Configure the Agent to run as a Service -sudo ./svc.sh install &
- Execute now to run as a service- ./runsvc.sh &
- Check the status of build Agent
  <img width="731" alt="image" src="https://github.com/shubhamagrawal17/Private-ACR-with-self-hosted-Agent/assets/24695227/ca8790d1-4e47-4897-ba40-fa407b4e2b72">
  - Agent is configured successfully now we need to install the tools so we can build our application.
  - **Install Docker** - **This is required because we are going to build the docker image on this agent.**
    
            <br/>sudo apt install -y apt-transport-https ca-certificates curl software-properties-common<br/>
            <br/>curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -<br/>
            <br/> sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"<br/>
            <br/>sudo apt update<br/>
            <br/>sudo apt install -y docker-ce<br/>
            <br/>sudo usermod -aG docker shubham<br/>
            <br/>sudo chmod 666 /var/run/docker.sock<br/>

  **Install Azure CLI** -
           <br/> curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash<br/>

**-  Create the Private azure container registry**
  - Private azure container registry can be created with only premium plan.
    <img width="473" alt="image" src="https://github.com/shubhamagrawal17/Private-ACR-with-self-hosted-Agent/assets/24695227/1fcb7e01-4d04-4bc6-b8a4-65e062cffcec">
- In network section you have to select private access and you have to create the private endpoint connection.
  <img width="1013" alt="image" src="https://github.com/shubhamagrawal17/Private-ACR-with-self-hosted-Agent/assets/24695227/c72637d5-5e7b-4ed8-ad5d-420c230e5164">
- Once you create the Private acr you can not access the repository from local machine because its not part of your vnet. you can access the ACR only from the Vnet which is peered with acr-vnet.
  <img width="761" alt="image" src="https://github.com/shubhamagrawal17/Private-ACR-with-self-hosted-Agent/assets/24695227/7a69bfb5-67e0-4393-acf4-d470fb76bfe1">
  - Next we want to push the image on acr from azure devops CI pipleine so we need to assign the IAM role acr push to managed identity which we have used in service connection.
    <img width="679" alt="image" src="https://github.com/shubhamagrawal17/Private-ACR-with-self-hosted-Agent/assets/24695227/314caf4e-a9b8-4a54-9767-a26772243ff4">
- Now we will add the agent pool name in our CI pipeline.
  <img width="386" alt="image" src="https://github.com/shubhamagrawal17/Private-ACR-with-self-hosted-Agent/assets/24695227/10741da2-baaa-480a-bae3-6cc165131d4a">
  - Now we will run the pipeline and we can see in below screenshot its failed due to access issue and that is the ip address of our azure VM.
    
    <img width="703" alt="image" src="https://github.com/shubhamagrawal17/Private-ACR-with-self-hosted-Agent/assets/24695227/a429e187-1ff9-4857-86d6-673c8d87eb90">

    <img width="695" alt="image" src="https://github.com/shubhamagrawal17/Private-ACR-with-self-hosted-Agent/assets/24695227/30c30688-e3d4-4df0-9def-da14b665c4f4">

    - This issue we are facing because there is no private link between acr-vnet and agent-vm , the vm should reach to acr from the private endpoint but on agent-vnet does not have access to acr private endpoint. so we can create the new virtual network link.
      
      <img width="974" alt="image" src="https://github.com/shubhamagrawal17/Private-ACR-with-self-hosted-Agent/assets/24695227/fec0a70b-41e2-47e0-8f9a-722c43b263fd">
      
      <img width="773" alt="image" src="https://github.com/shubhamagrawal17/Private-ACR-with-self-hosted-Agent/assets/24695227/d4298fa7-9fb0-4087-80a9-6081986e64ba">

  - Now we can see the build and pushed taks completed with no errors.
        <img width="640" alt="image" src="https://github.com/shubhamagrawal17/Private-ACR-with-self-hosted-Agent/assets/24695227/f8e60f28-56e0-4eb3-abd8-b65412fb707d">
  - we will login the agent vm to see the acr repository
       <br/>az acr repository list -n eticket.azurecr.io<br/>
      <br/>az acr repository show-tags -n eticket.azurecr.io --repository etickets-main<br/>
    <img width="760" alt="image" src="https://github.com/shubhamagrawal17/Private-ACR-with-self-hosted-Agent/assets/24695227/865d9da8-7509-4466-b202-128d51b7071d">




     


      




    




