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
-  
