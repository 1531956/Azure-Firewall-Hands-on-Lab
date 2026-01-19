
---

# [Azure Firewall Hands-On Lab](https://www.youtube.com/watch?v=AImIkxryErQ)

---

**Credits**

- This project was inspired by:
	- [A Guide To Cloud](https://www.youtube.com/@AGuideToCloud)

---

Azure Firewall is a managed, cloud-based network security service that protects your Azure Virtual Network resources.

![image](https://github.com/user-attachments/assets/9f0f92d1-2eb5-4238-9e5b-c5ced9e5ae72)
![image](https://github.com/user-attachments/assets/aeed154c-0ede-482e-829d-d9aa2e45f142)

---

This project is a hands-on lab tutorial demonstrating how to create and configure an Azure firewall. Here's a summary:

- **Setting up the Environment**: We begin by creating a resource group and a virtual network with two subnets: "firewall subnet" and "workload subnet".
- **Virtual Machine Creation**: We then create a Windows Server virtual machine to log in to at the end of the project.
- **Firewall Deployment**: Then we deploy an Azure firewall using a firewall policy.
- **Route Table Configuration**: Afterwards, a route table is created, associating the "workload subnet" with the firewall's IP address as the default gateway.
- **Application Rule Configuration**: An application rule is added to the firewall policy, allowing outbound access to `google.com`.
- **Network Rule Configuration**: A network rule is added, allowing outbound DNS traffic (UDP port 53) to specific public DNS servers.
- **Destination NAT Rule Configuration**: A destination NAT rule is configured for RDP (port 3389), translating traffic to the VM's private IP.
- **DNS Server Configuration (Optional)**: We'll show how to change the primary and secondary DNS servers for the virtual machine.
- **Firewall Testing**: Finally, the firewall is tested by attempting to access `google.com` (successful) and `microsoft.com` (unsuccessful due to the lack of an explicit rule). This demonstrates that the configured rules are working correctly.

- **Key Skills/Tools:** Azure Firewall, route tables, network rules, NAT, DNS settings

<center>Let's get started!</center>

---

## Task 1: Create a resource group

- Log in to Azure and click "Resource groups".
- ![image](https://github.com/user-attachments/assets/64974b02-9f44-4c6c-9aba-bad9f695a01e)
- Click "Create".
- ![image](https://github.com/user-attachments/assets/4b386337-b946-4075-b402-553ba8694440)
- Now on the "Create a resource group" page, name the resource group `Test-FW-RG`. Then click "Review + create".
- ![image](https://github.com/user-attachments/assets/3f3680e1-25ae-4686-bfbf-0fb4389f1cdc)
- Click "Create".
- ![image](https://github.com/user-attachments/assets/f7ad7742-704d-4582-8ae6-b720b0f407b1)
- Success!
- ![image](https://github.com/user-attachments/assets/3c5dad1e-5e08-42cf-9321-d2dca11150cd)

---

## Task 2: Create a virtual network and subnets

- Now we need to create a virtual network within the resource group. Click on your resource group `Test-FW-RG`. Then click "Create".
- ![image](https://github.com/user-attachments/assets/931d2fc2-9b9d-4f99-a47b-584a95f588db)
- You'll be brought to the "Marketplace" page. Search `Virtual network` in the search bar and click on the service.
- ![image](https://github.com/user-attachments/assets/cd3a056d-65d0-4324-ade1-b202506593b2)
- Then click "Create".
- ![image](https://github.com/user-attachments/assets/23bcd761-1d2d-4d3e-9b7e-f5260bcd33a2)
- Once you're on the "Create virtual network" page, enter `Test-FW-VN` for the virtual network name. Then click "Next".
- ![image](https://github.com/user-attachments/assets/dfa48736-61d8-4099-a94b-2b1b473cf113)
- Click "Next" to skip the "Security" tab since we will add the Azure Firewall later.
- ![image](https://github.com/user-attachments/assets/d760c252-8933-43aa-84f9-e51ce2749584)
- On the "IP addresses" section, edit the "default" subnet by clicking the pencil icon. A pop up will form on the right hand side that says "Edit subnet". Here is what we will choose for the options:
	- Subnet purpose: `Azure Firewall`
	- Name: `AzureFirewallSubnet`
	- Everything else: Leave default
- Click "Save".
- ![image](https://github.com/user-attachments/assets/4414a688-3f3d-41c5-9881-f7e7ba623965)
- Now click "Add a subnet". This time name the subnet `Workload-SN`. Then click "Add".
- ![image](https://github.com/user-attachments/assets/affbcb25-201e-40ee-8307-3b9f89feea0c)
- Click "Review + create".
- ![image](https://github.com/user-attachments/assets/746adca1-fd4d-4eae-b4dc-1b5e71543017)
- Click "Create".
- ![image](https://github.com/user-attachments/assets/481b835d-fb18-410d-9ecf-00b4a1dd7698)
- Success!
- ![image](https://github.com/user-attachments/assets/8628c69c-9408-4d6d-8d3a-edb6827b0cba)

---

## Task 3: Create a virtual machine

- Search `Virtual machines` in the search bar and select the first option.
- ![image](https://github.com/user-attachments/assets/5a20297a-912e-4882-b177-7239e6d95748)
- Click "Create". Then select "Azure virtual machine".
- ![image](https://github.com/user-attachments/assets/a38ec95d-5a18-42b0-a76c-db1c5044881a)
- You will be brought to this page:
- ![image](https://github.com/user-attachments/assets/258aae53-9795-4d42-ad04-7ef94283fef2)
- Let's fill in the fields section by section:
	- **Project details**
		- Subscription: `Azure subscription 1`
		- Resource group: `Test-FW-RG`
	- ![image](https://github.com/user-attachments/assets/ae9a3fd2-f355-4adc-ab0f-9f4066c354f1)
	- **Instance details**
		- Virtual machine name: `Srv-Work`
		- Region: `(US) East US`
		- Availability options: `Availability zone`
		- Zone options: `Self-selected zone`
		- Availability zone: `Zone 1`
		- Security type: `Trusted launch virtual machines`
		- Image: `Windows Server 2019 Datacenter - x64 Gen2 (free services eligible)`
		- VM architecture: `x64`
		- Run with Azure Spot discount: Leave unchecked
		- ⚠️ Size:
			- If everything is greyed out, click "See all sizes". Then find the one closest to these specs:
				- **VM Size** - `Standard_D2s_v3`
				- **Type** - `General Purpose`
				- **vCPUs** - `2`
				- **RAM (GiB)** - `8`
				- **Data Disks** - `4`
				- **Max IOPS** - `3750`
				- **Local storage (GiB)** - `N/A`
				- **Premium disk** - `Supported`
				- **Cost/month** - `$70.08`
			- I found `D2as_v6`. Click "Select" when finished.
			- ![image](https://github.com/user-attachments/assets/4a3b7149-20e6-4b98-9997-c5d229177a63)
		- Enable Hibernation: Leave unchecked
	- ![image](https://github.com/user-attachments/assets/baa80980-bb9e-4879-9190-301037e3ed87)
	- **Administrator account**
		- Username: `labuser`
		- Password: `******`
		- Confirm password: `******`
	- ![image](https://github.com/user-attachments/assets/168573ac-0583-476b-a9d7-77f1dae64bae)
	- **Inbound port rules**
		- Public inbound ports: `Allow selected ports`
		- Select inbound ports: `RDP (3389)`
	- ![image](https://github.com/user-attachments/assets/608d0f44-2775-450d-947f-ff52929fe386)
- Click "Next : Disks >".
- ![image](https://github.com/user-attachments/assets/a420463e-52ea-4025-8069-c0817278f5b7)
- Leave everything as default in the "Disks" section. Click "Next..".
- ![image](https://github.com/user-attachments/assets/296e059b-096c-4c1b-aadf-b63f07449368)
- Leave everything as default in the "Networking" section except for one option. Check the checkbox for "Delete public IP and NIC when VM is deleted". Click "Next..".
- ![image](https://github.com/user-attachments/assets/3031d484-8aac-4114-989b-4b3ce084ea94)
- Leave everything as default in the "Management" section. Click "Next..".
- ![image](https://github.com/user-attachments/assets/2d82f033-8d49-44ce-9588-77b1b520f59c)
- Leave everything as default in the "Monitoring" section except for "Boot diagnostics" where we will choose "Disable". Click "Next..".
- ![image](https://github.com/user-attachments/assets/42ad18dd-1504-4c21-b18a-d4b918f7c139)
- Click "Review + create" to skip the "Advanced" and "Tags" section.
- ![image](https://github.com/user-attachments/assets/e1e6da28-97b0-4aaf-a0d2-26b9910469cd)
- Validation passed! Click "Create".
- ![image](https://github.com/user-attachments/assets/06fe88f7-5549-4696-8d4c-4efb676c07a1)
- Success! The deployment is complete!
- ![image](https://github.com/user-attachments/assets/1ed5de27-72d1-4d58-b36b-df4da791f726)
- Click on `Srv-Work` to go to your machine and you can see that it is running:
- ![image](https://github.com/user-attachments/assets/d312058c-9a23-4b9f-a729-122693db50f6)

---

## Task 4: Deploy the firewall and firewall policy

- Search `Firewalls` in the search bar and select the first option.
- ![image](https://github.com/user-attachments/assets/d99ed411-7041-4ec3-a220-a91a1f72031a)
- Click "Create".
- ![image](https://github.com/user-attachments/assets/8225538e-9011-4ddf-8710-7d2ad4d4711d)
- Now we're on the "Create a firewall" page.
- ![image](https://github.com/user-attachments/assets/776b7ed4-e56a-4508-961c-80339bd71970)
- Select these options:
	- **Project details**
		- Subscription: `Azure subscription 1`
		- Resource group: `Test-FW-RG`
	- ![image](https://github.com/user-attachments/assets/86449494-729c-4e3d-9769-e5dd3bc9c9d1)
	- **Instance details**
		- Name: `Test-FW01`
		- Region: `East US`
		- Availability zone: `None`
		- Firewall SKU: `Standard`
		- Firewall management: `Use a Firewall Policy to manage this firewall`
		- Firewall policy:
			- Click "Add new". Then give it the name `fw-test-pol` and click "OK".
			- ![image](https://github.com/user-attachments/assets/9f626b72-f333-44ec-8278-dcd360484579)
		- Choose a virtual network: `Use existing`
		- Virtual network: `Test-FW-VN (Test-FW-RG)`
		- Public IP address:
			- Click "Add new". Then give it the name `fw-pip` and click "OK".
			- ![image](https://github.com/user-attachments/assets/3d7e0271-5e26-479e-8af6-bee5ca174601)
	- ![image](https://github.com/user-attachments/assets/0b408380-b490-459a-9384-6a55ed531f5e)
	- **Firewall Management NIC**
		- Enable Firewall Management NIC: Uncheck the checkbox
- Click "Next : Tags".
- ![image](https://github.com/user-attachments/assets/1101dd08-397a-44f0-9559-ddd4a6ddd403)
- Leave blank. Click "Review + create >".
- ![image](https://github.com/user-attachments/assets/9dfcc033-4836-4a64-ae57-a7b2f9f6a429)
- Click "Create".
- ![image](https://github.com/user-attachments/assets/9f2ff53b-b173-41af-b276-89b7fb08de43)
- Success! Now click "Go to resource".
- ![image](https://github.com/user-attachments/assets/b70cac6e-1d89-4b65-ab7c-f0504e63c8e4)
- Here in the "Overview" page, we can see that the Firewall private IP address is `10.0.0.4`
- ![image](https://github.com/user-attachments/assets/dd63b2e8-7de1-499a-8776-1f133461c88c)
- On the left hand side, underneath the "Settings" drop down menu, click on "Public IP configuration" to see the Public IP, which is `172.190.212.34`
- ![image](https://github.com/user-attachments/assets/0d53edfb-104d-43dd-a241-d672273b8227)

---

## Task 5: Create a default route

- Search `Route tables` in the search bar and select the first option.
- ![image](https://github.com/user-attachments/assets/8d04875c-4e0d-488c-97df-cf34decc9ebd)
- Click "Create".
- ![image](https://github.com/user-attachments/assets/1e4a1091-4b80-45ba-9cfd-b80e10ff72f6)
- On the "Create Route table" page, select your resource group and name the route `Firewall-route`. Then click "Review + create".
- ![image](https://github.com/user-attachments/assets/e8ad2982-8360-4568-95a0-dce74c43a2f8)
- Click "Create".
- ![image](https://github.com/user-attachments/assets/025ae714-5d1d-4dbf-b98b-9e1f0fa101ce)
- Success! Now click on `Firewall-route`.
- ![image](https://github.com/user-attachments/assets/a4255fc9-bfdb-4f4e-882b-65d3071b5dad)
- Here is the firewall route we just created. Under the "Settings" drop down menu, click "Subnets".
- ![image](https://github.com/user-attachments/assets/a627b2a0-eca7-46d4-a0b8-c148f1741c7d)
- As you can see it is empty. Click "Associate" to add one.
- ![image](https://github.com/user-attachments/assets/58281895-963b-4226-bd27-7e5bbbd35a94)
- Click "OK".
- ![image](https://github.com/user-attachments/assets/6c76a9b4-47c6-44b6-a961-60878d24c390)
- Success!
- ![image](https://github.com/user-attachments/assets/3481ad27-8306-472c-92d0-e99021a8fc2f)
- Now under "Settings" click on "Routes". Then click "Add". We'll enter these inputs:
	- Route name: `fw-dg`
	- Destination type: `IP Addresses`
	- Destination IP addresses/CIDR ranges: `0.0.0.0/0`
	- Next hop type: `Virtual appliance`
	- Next hop address: Our Test Firewall IP - `10.0.0.4`
		- ![image](https://github.com/user-attachments/assets/1e0a27ba-3a3f-4a9b-a552-8c8d8a3965e0)
- Click "Add".
- ![image](https://github.com/user-attachments/assets/34a88d22-333f-4b15-abe0-4810567870fb)
- Success!
- ![image](https://github.com/user-attachments/assets/97208d4f-92e6-49d2-98a6-d578e5d64607)

---

## Task 6: Configure an application rule

- Search `fw-test-pol` in the search bar and select the first option. This is the firewall policy we created earlier.
- ![image](https://github.com/user-attachments/assets/4cc5d536-2fb3-4fd3-ab3e-bdbb2ba2bc1e)
- Now on the "fw-test-pol" page, we are going to add an application rule that allows outbound access to `google.com`. Under the "Rules" drop down menu, click "Application rules".
- ![image](https://github.com/user-attachments/assets/0abb9d03-6700-4e43-bfaf-12bcff2a0c95)
- Click "Add a rule collection".
- ![image](https://github.com/user-attachments/assets/bfee1c35-4417-46e5-925f-38e734cb2a04)
- Enter these options in their respective fields.
	- **Add a rule collection**
		- Name: `App-Coll01`
		- Rule collection type: `Application`
		- Priority: `200`
		- Rule collection action: `Allow`
		- Rule collection group: `DefaultApplicationRuleCollectionGroup`
		- Rules:
			- Name: `Allow-Google`
			- Source type: `IP Address`
			- Source: `10.0.1.0/24` → your workload subnet
			- Protocol: `http, https`
			- TLS inspection: unchecked
			- Destination Type: `FQDN`
			- Destination: `www.google.com`
	- Click "Add".
	- ![image](https://github.com/user-attachments/assets/674f04d0-e1a4-410a-b7b7-ca87fad338f9)
- Success!
- ![image](https://github.com/user-attachments/assets/6c5707c5-cb3a-492f-b7de-520ad8e7d2eb)

---

## Task 7: Configure a network rule

- Still within our `fw-test-pol` "Firewall Policy" page, under the "Rules" drop down menu, select "Network rules". Then click "Add a rule collection".
- ![image](https://github.com/user-attachments/assets/4b9c3741-83cb-4670-8285-ec22d7faab58)
- We'll be creating an **Allow-DNS** rule to say: “Allow all VMs in 10.0.1.0/24 to send UDP requests to port 53 **only to these DNS servers**.” Enter these options in their respective fields.
	- **Add a rule collection**
		- Name: `Net-Coll01`
		- Rule collection type: `Network`
		- Priority: `200`
		- Rule collection action: `Allow`
		- Rule collection group: `DefaultNetworkRuleCollectionGroup`
		- Rules:
			- Name: `Allow-DNS`
			- Source type: `IP Address`
			- Source: `10.0.1.0/24` → your workload subnet
			- Protocol: `UDP`
			- Destination Ports: `53` → DNS
			- Destination Type: `IP Address`
			- Destination: `8.8.8.8, 8.8.4.4` Google's Public DNS IPs → where DNS queries are allowed to go
	- Click "Add".
	- ![image](https://github.com/user-attachments/assets/81bc19e3-fd2c-4ce9-a685-3d0064bf918c)
- Success!
- ![image](https://github.com/user-attachments/assets/701e569f-701f-4d22-aa94-98a4c8ca5808)

---

## Task 8: Configure a Destination NAT (DNAT) rule

- Still within "Rules", click "DNAT rules". Then click "Add a rule collection".
- ![image](https://github.com/user-attachments/assets/2285e18b-b85d-461e-81e8-1aecc56bdbb4)
- Enter these options in their respective fields.
	- **Add a rule collection**
		- Name: `rdp`
		- Rule collection type: `DNAT`
		- Priority: `200`
		- Rule collection action: `Destination Network Address Translation (DNAT)`
		- Rule collection group: `DefaultDnatRuleCollectionGroup`
		- Rules:
			- Name: `rdp-nat`
			- Source type: `IP Address`
			- Source: `*`
			- Protocol: `TCP`
			- Destination Ports: `3389`
			- Destination (Firewall IP): `172.190.212.34` -> Firewall Public IP
				- ![image](https://github.com/user-attachments/assets/9fa71bf0-79df-47d1-b6c5-eb9503a4146f)
			- Translated type: `IP Address`
			- Translated address or: `10.0.1.4` -> VM Private IP address
				- ![image](https://github.com/user-attachments/assets/3b8b34da-3eef-4813-a8c1-bc6a95762c3d)
			- Translated port: `3389`
	- Click "Add."
	- ![image](https://github.com/user-attachments/assets/428140bb-0196-47d2-b4ea-37dfde624dc2)
- Success!
- ![image](https://github.com/user-attachments/assets/bd5e3470-87a0-4e5e-9128-a5dbc9315b35)

---

## Task 9: Change the primary and secondary DNS address for the server's network interface - (OPTIONAL)

- Type `Srv-Work` in the search bar and select the first option.
- ![image](https://github.com/user-attachments/assets/295533e3-51e3-4471-b266-c539548ffb4a)
- Under the "Networking" drop down menu, click "Network settings". Then click on your network interface.
- ![image](https://github.com/user-attachments/assets/9db7b274-8ea5-43c6-bdf7-b6ac60180450)
- On the left hand side under "Settings", click "DNS servers".
- ![image](https://github.com/user-attachments/assets/908f0ef7-618b-47a0-8992-d8505ed2549f)
- Click "Custom". Enter `8.8.8.8` and `8.8.4.4`. Then click "Save".
- ![image](https://github.com/user-attachments/assets/1fa63498-0e12-4aa3-a7f8-9cfdf8191419)
- Success! Now go back to your VM Overview page.
- ![image](https://github.com/user-attachments/assets/f8cfeebd-b7e1-4101-99fc-6c3a4e4bcc3a)
- Click "Restart".
- ![image](https://github.com/user-attachments/assets/a2164260-3ac1-46f9-8a2c-6a1b612bb712)
- Click "Yes".
- ![image](https://github.com/user-attachments/assets/f559b2d8-ef78-4779-8a2f-d96a8617a7e7)
- Success!
- ![image](https://github.com/user-attachments/assets/88622b26-bc9e-458d-9eea-62349caf0c7c)

---

## Task 10: Test the firewall

- Now we will verify if the rules were configured correctly. This config will enable us to connect a remote desktop connection to the server through the firewall via the firewall's public IP address. Let's test it out! On your local Windows machine or Windows VM, search for `Remote Desktop Connection` and open the application.
- ![image](https://github.com/user-attachments/assets/905fcb21-4e8e-45fb-b36f-40a423fbc57c)
- When it opens, click "Show Options", and where it says "Computer:", type in the firewall's public IP address, followed by the port number `3389`. For me that is `172.190.212.34:3389`. In the "User name:" field, enter `labuser`. Then click "Connect".
- ![image](https://github.com/user-attachments/assets/bbcaa28e-9eb3-4084-b948-81e01d88b9fd)
- It will now ask for the password you created. Enter it and click "OK".
- ![image](https://github.com/user-attachments/assets/3adf74b8-77c7-4b8d-94f9-b80bab8b894e)
- Click "Yes".
- ![image](https://github.com/user-attachments/assets/189815f6-ab9d-40d3-bd5c-23fdd54b5e3c)
- Now we are using the public IP of the firewall to connect into the server. Now that we are in the server, let's test if the firewall rules are working. Click "No" on the right hand side. Also, exit out of "Server Manager" when it pops up.
- ![image](https://github.com/user-attachments/assets/8d98fc1b-fb0a-4b8c-bb5c-a82b6b801106)
- Click "Internet Explorer" in the task bar. Then click "OK".
- ![image](https://github.com/user-attachments/assets/d05633b5-fc3a-4cee-b68d-c085e141b260)
- In the search bar, type `www.google.com`. We **should** be able to access this website.
- ![image](https://github.com/user-attachments/assets/45923869-9e87-4fdd-b66a-b744bb7b8346)
- Click "OK".
- ![image](https://github.com/user-attachments/assets/e7cc4b9a-cf5e-4b2d-9288-74ffc6bdb843)
- Click "Close".
- ![image](https://github.com/user-attachments/assets/1de45faa-2f84-41d3-81ae-e8ae06226a7a)
- ✅ Success! We are able to access `google.com`!
- ![image](https://github.com/user-attachments/assets/5d9f2267-74ed-49e2-8de2-64abd84fcb10)
- Now let's try `www.microsoft.com`. We **should not** be able to access this website.
- ![image](https://github.com/user-attachments/assets/5e4e680f-f461-475a-996b-818a28fa6bce)
- Click "Yes".
- ![image](https://github.com/user-attachments/assets/2052410f-b39b-45a6-86c0-ad2547018f2b)
- 🚫 As you can see we successfully blocked anything but `google.com`. When we try and access `microsoft.com` we get an error of "Action: Deny. Reason: No rule matched. Proceeding with default action."
- ![image](https://github.com/user-attachments/assets/e5c17042-009b-490c-9a72-be42900ed5f7)

---

## Delete your Resources 🗑️

- Now that the lab is complete, don't forget to delete your resources! Head to your portal and click "All resources".
- ![image](https://github.com/user-attachments/assets/89ae0a30-e781-4307-b5f9-0274fb5b5329)
- Select all the resources and click "Delete".
- ![image](https://github.com/user-attachments/assets/aaf630d7-7ce9-4e8e-adb0-88f99c38c5cd)
- Enter "delete" to confirm deletion. Then click "Delete".
- ![image](https://github.com/user-attachments/assets/09585912-e831-47a4-a757-dbe698b5c352)
- Click "Delete". (You may have to complete this process a few times to completely get rid of all resources, or delete manually.)
- ![image](https://github.com/user-attachments/assets/51cdf893-9f6d-4a3b-a002-f6e55e4d7c52)
- Now delete the Resource groups and we're done!
	- `NetworkWatcherRG`
	- ![image](https://github.com/user-attachments/assets/d19e1061-8c94-4c76-8166-b2b13b4139b2)
	- `Test-FW-RG`
	- ![image](https://github.com/user-attachments/assets/f99aa7e6-975e-4ed9-8101-0eea879a7d8b)
- Done! Thank you for participating in this lab!

---
