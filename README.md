# Private-ACR-with-self-hosted-Agent
**Architecture**-
<img width="517" alt="image" src="https://github.com/shubhamagrawal17/Private-ACR-with-self-hosted-Agent/assets/24695227/e17f44f4-9a5f-40a1-8172-ae7131d4b162">


**Steps**-
-  First create the Service connection from azure devops to Azure cloud using managed identity or service principle.
-  Create the Agent pool from the project setting.
-  Now create the 2 virtual network (one for self-hosted agent and another for private ACR).
-  Create the virtual network peering between them.
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

