<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>Preparing Active Directory Infrastructure in (Azure)</h1>
This repository demonstrates the deployment of an on-premises-style Active Directory infrastructure using Microsoft Azure Virtual Machines (VMs). The project walks through provisioning a Domain Controller (DC) and a client machine, configuring network connectivity, and implementing the required DNS settings to support Active Directory Domain Services.<br />

<h2>Environments and Technologies Used</h2>

<img src="https://skillicons.dev/icons?i=azure,windows,powershell" />

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)
<h2>Architecture Overview </h2>

- DC-1 = Domain Controller (DNS + AD DS)
- Client-1 = Domain-joined machine
- Both connected via VNet (same subnet)

<h2>Deployment and Configuration Steps</h2>

**1. Setup Domain Controller VM in Azure**
---
***CREATE RESOURCE GROUP.***

Created a resource group to organize and manage all Azure resources for the Active Directory lab. Begin by signing in to the [Azure](https://portal.azure.com/) portal, searching for “Resource Groups,” and selecting the service. From there, initiated the creation process, assigned the name “rg-active-directory-lab,” and completed the setup by selecting “Review + Create,” followed by “Create.”
 
 <img width="1119" height="709" alt="577009206-5785d2fe-6cd6-4c2a-bdec-b9283fe8d78b" src="https://github.com/user-attachments/assets/186b419b-02a6-4b5f-b4c1-2c7f64a73fa1" />
<img width="813" height="703" alt="Screenshot 2026-04-10 211121" src="https://github.com/user-attachments/assets/1cbbe209-dfdd-499a-bb5a-bcf041ad6306" />

<br>
<br>
<br>

***CREATE VIRTUAL NETWORK.***

In the Azure Portal, I searched for “Virtual Networks,” selected the service, and clicked “+ Create.” During the setup process, I chose the previously created resource group “rg-active-directory-lab” and assigned the name “vnet-active-directory-lab” to the virtual network. This virtual network provides a secure environment that allows Azure virtual machines to communicate with one another.

<img width="816" height="703" alt="Screenshot 2026-04-10 211247" src="https://github.com/user-attachments/assets/f7842141-c51d-4625-a391-655d854b45b6" />

<br>
<br>
<br>

***CREATE DOMAIN CONTROLLER VM***

In the Azure portal, I initiated virtual machine creation by searching for “Virtual Machine” in the search bar and selecting "+ Create". I chose the **rg-active-directory-lab** resource group, named the VM "DC-1", and configured the username and password under the Admin Account section. I set the region to **(US) East US** with **Availability Zone 3**, selected the **Windows Server 2022 Datacenter: Azure Edition (x64 Gen2)** image, and used the **Standard D2s_v3 (2 vCPUs, 8 GiB RAM)** size.

<img width="814" height="1289" alt="Screenshot 2026-04-10 211602" src="https://github.com/user-attachments/assets/ec288a74-6741-4888-99ab-7d4f5473604e" />

<br>
<br>
<br>

During Virtual machine creation, I navigated to the **Networking** tab for the virtual network. I selected the previously created VNet "vnet-active-directory-lab", assigned a public IP address, and selected 'default' for Subnet. Once the networking settings were configured, select "Review + Create," then "Create". 

<img width="848" height="1302" alt="Screenshot 2026-04-10 211624" src="https://github.com/user-attachments/assets/b2943ae0-13d7-4d14-ab5f-7b23da956bd0" />


<br>
<br>
<br>

***DOMAIN CONTROLLER VM CONFIGURATION***

After creating the DC-1 virtual machine, I accessed it by searching “Virtual Machines” in the Azure Portal and navigating to **Virtual Machines > DC-1**. From the left-hand menu, I selected **Networking > Network Settings**, then clicked on the VM’s **Network Interface** listed under **Essentials**.

<img width="1319" height="388" alt="577014590-ff30e45b-65d3-4e82-9f36-616d9e7a63d6" src="https://github.com/user-attachments/assets/19dae7fc-c687-4256-8b13-d6156589380f" />

<br>
<br>
<br>

Within the network interface settings, I navigated to **Settings > IP Configurations > ipconfig1**, where I changed the Private IP allocation from dynamic to **Static**, and saved the configuration. Afterward, I returned to the DC-1 VM overview page to locate and copy the Domain Controller’s public IP address.

Configuring the Domain Controller with a static private IP ensures consistent and reliable network communication. This is critical because client machines depend on the Domain Controller for DNS resolution and authentication services, which require a fixed IP address to function properly.

<img width="1048" height="819" alt="Screenshot 2026-04-10 212605" src="https://github.com/user-attachments/assets/2c2f1736-b96e-4c63-9e35-58c914329ab5" />

<br>
<br>
<br>

***DISABLE FIREWALL ON DC-1 VM (Temporary for Testing)***

On my local computer, I used Remote Desktop to connect to DC-1 VM using its public IP address (20.83.153.138). To connect, I entered the correct credentials created when the VM was created.   

<img width="1381" height="613" alt="Screenshot 2026-04-10 213001" src="https://github.com/user-attachments/assets/25ad1938-1ef6-4450-ab8c-85e5e596b9bc" />

<br>
<br>
<br>

Next, I selected **Yes** to proceed, which initiated the Domain Controller’s boot process. Once the system was fully up and running, I opened **Windows Defender Firewall with Advanced Security** and accessed **Windows Defender Firewall Properties**. From there, I disabled the firewall by setting the Firewall State to **Off** for the Domain, Public, and Private profiles, then clicked **Apply** to save the changes.

**Reason:**

The firewall is temporarily disabled to allow connectivity testing between virtual machines, permitting `ICMP` ping traffic. DC-1 has been successfully deployed, connected to the correct Virtual Network and subnet, and is now ready for further Active Directory configuration and role installation.

> [!NOTE]
> ***In a production environment, firewall rules would be properly configured rather than disabled.***

<img width="911" height="736" alt="Screenshot 2026-04-10 213841" src="https://github.com/user-attachments/assets/22168902-e12c-4377-a31f-c8973e2f4082" />

**2. Setup Client VM in Azure**
---
***CREATE CLIENT VM***

In the Azure portal, I initiated virtual machine creation by searching for “Virtual Machine” in the search bar and selecting **+ Create**. I chose the **rg-active-directory-lab** resource group, named the VM **Client-1**, and configured the administrator username and password under the Admin Account section. I set the region to **(US) East US** with **Availability Zone 3**, selected the **Windows 10 Enterprise 22H2 (x64 Gen2)** image with x64 architecture, and used the **Standard D2s_v3 (2 vCPUs, 8 GiB RAM)** size. 

In the **Networking** tab, I selected the previously created virtual network (**rg-active-directory-lab**) and kept the subnet as default. After completing the configuration, I clicked **Review + Create**, then selected **Create** to deploy the virtual machine.

<img width="883" height="1299" alt="Screenshot 2026-04-10 214235" src="https://github.com/user-attachments/assets/51f34805-3fdc-40fc-b8ad-197999d63147" />

<br>
<br>
<br>

***CLIENT VM CONFIGURATION***

After deployment, I navigated to the **Virtual Machines** list, selected the **DC-1** VM, and opened **Network Settings** from the left-hand pane. From there, I located and recorded the private IP address (10.0.0.4) for later use.


<img width="1489" height="590" alt="577113397-eb908055-194b-47b4-9faf-01a52125f2fe" src="https://github.com/user-attachments/assets/8da620c0-6a85-4236-99cd-163a39e9e1ee" />

<br>
<br>
<br>

Next, I returned to the Virtual Machines list and selected **Client-1**. From the left-hand pane, I navigated to **Network > Network Settings**, then selected the **Network Interface** under **Essentials**.

<img width="1483" height="664" alt="577113444-9b1298b3-b45c-4f6f-b932-6e83bee43f2b" src="https://github.com/user-attachments/assets/69c7fafe-a23e-48a8-8a66-89a384ba5ade" />

<br>
<br>

In the network interface settings, I opened **DNS servers**, switched the option to **Custom**, and entered the **DC-1 private IP address**. Finally, I clicked **Save** to apply the configuration.

<img width="837" height="633" alt="Screenshot 2026-04-10 215627" src="https://github.com/user-attachments/assets/e7b641fb-9423-4065-b646-05f315fe52b3" />

<br>
<br>

To apply the updated DNS settings, I restarted Client-1 directly from the Azure Portal. After the reboot, the machine was successfully configured to use DC-1 for DNS resolution and was ready for domain connectivity.

<img width="1378" height="563" alt="577113588-d43307e5-ff2a-46b6-b26e-a91094ff8c1f" src="https://github.com/user-attachments/assets/18727a7a-eb54-4347-8d60-1151e5053886" />

<br>
<br>
<br>

***Validate Connectivity and Configuration***

To validate connectivity to **DC-1**, I used **Remote Desktop** on my local computer to connect to the **Client-1** VM using its public IP address (52.186.171.6) using the credentials created during deployment. After successfully logging in, I opened **PowerShell** and ran `ping` to the Domain Controller’s private IP address (**10.0.0.4**) to confirm network connectivity. Finally, I executed `ipconfig /all` to verify that the **DNS server** is set to the private IP address of **DC-1**.

<img width="695" height="853" alt="Screenshot 2026-04-10 221728" src="https://github.com/user-attachments/assets/80448694-4f6f-4c00-b9de-baaecd15edce" />


**Successful replies confirm:**
  * Network connectivity is functioning properly
  * The firewall is not blocking ICMP traffic
  * Both **DC-1** and **Client-1** VMs are on the same Virtual Network and subnet

**3. Install Active Directory**
---

To install Active Directory on the domain controller, I returned to the **DC-1** VM and navigated to the **Server Manager** dashboard. Proceed by selecting option 2, **Add roles and features**, then select **Next** until you reach **Server Roles**. In Server roles, select **Active Directory Domain Services (AD DS)** under **Server Roles**. Upon selection, I was prompted to add the required features, including Group Policy Management and Remote Server Administration Tools. I clicked **Add Features** and ensured that management tools were included to enable full administrative functionality. 
