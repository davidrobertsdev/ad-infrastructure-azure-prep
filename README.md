<p align="center">
<img src="https://i.imgur.com/Clzj7Xs.png" alt="osTicket logo"/>
</p>

<h1>Active Directory - Preparing the Infrastructure in Azure</h1>
This tutorial walks through preparing the Azure infrastructure for an Active Directory lab. We build a resource group and virtual network, deploy a Windows Server virtual machine to act as the Domain Controller (DC-1), give it a static private IP, deploy a Windows 10 client (Client-1) on the same network, point the client's DNS at the Domain Controller, and verify connectivity. Nothing is promoted to a domain controller yet; this lab lays the groundwork for the Active Directory labs that follow.<br />


<h2>Video Demonstration</h2>

- ### [YouTube: How To Prepare Active Directory Infrastructure in Azure](https://www.youtube.com)

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Resource Groups, Virtual Networks, Virtual Machines)
- Remote Desktop
- Windows PowerShell
- Windows Defender Firewall

<h2>Operating Systems Used </h2>

- Windows Server 2025 Datacenter: Azure Edition (DC-1)
- Windows 10 (Client-1)

<h2>Prerequisites</h2>

- A Microsoft Azure account with an active subscription
- A Remote Desktop client to connect to both virtual machines
- Permission to create resource groups, virtual networks, and virtual machines in the subscription

<h2>Lab Objectives</h2>

- Create a resource group and a virtual network with a subnet
- Deploy DC-1 (Domain Controller) and assign it a static private IP address
- Disable the Windows Firewall on DC-1 for connectivity testing
- Deploy Client-1 in the same region and virtual network as DC-1
- Set Client-1's DNS server to DC-1's private IP address
- Verify connectivity with ping and confirm the DNS setting with <code>ipconfig /all</code>

<h2>Configuration Steps</h2>

<h3>Part 1: Azure Foundation</h3>

<p>
<img src="YOUR-IMGUR-LINK-1" height="80%" width="80%" alt="Creating the resource group"/>
</p>
<p>
<b>Step 1: Create the Resource Group.</b>
<br />1. Sign in to the Azure portal and search for <b>Resource groups</b>.
<br />2. Click <b>Create</b>.
<br />3. Select your <b>Subscription</b> (Azure subscription 1).
<br />4. Enter the <b>Resource group name</b>: <code>active-directory-1</code>.
<br />5. Choose a <b>Region</b> (I used South Central US).
<br />6. Click <b>Review + create</b>, then <b>Create</b>.
<br /><br />
A resource group is a container for everything related to one solution (VMs, network interfaces, disks, virtual networks), so the whole lab can be managed, and later cleaned up, as one unit. The group's region only controls where its metadata is stored. The resources inside it can live in a different region, which is why the VMs below are deployed to East US 2.
</p>
<br />

<p>
<b>Step 2: Create the Virtual Network and Subnet.</b>
<br />1. Search for <b>Virtual networks</b> and click <b>Create</b>.
<br />2. Select the <code>active-directory-1</code> resource group and name the network <code>active-directory-vnet</code>.
<br />3. Set the region to <b>East US 2</b> so it matches the VMs.
<br />4. On the <b>IP addresses</b> tab, create a subnet. My subnet used the <code>10.0.1.0/24</code> range.
<br />5. Click <b>Review + create</b>, then <b>Create</b>.
<br /><br />
The virtual network is the private network both machines will live on, so they can talk to each other by private IP address. The VNet and subnet can also be created from the Networking tab while creating the first VM, which is how Azure auto-named my subnet.
</p>
<br />

<h3>Part 2: Domain Controller (DC-1)</h3>

<p>
<img src="YOUR-IMGUR-LINK-2" height="80%" width="80%" alt="DC-1 running in the VM list"/>
</p>
<p>
<b>Step 3: Create the DC-1 virtual machine.</b>
<br />1. Go to <b>Virtual machines</b> and click <b>Create > Azure virtual machine</b>.
<br />2. Select the <code>active-directory-1</code> resource group and name the VM <code>DC-1</code> (the portal displays it as <code>dc-1</code>).
<br />3. Set the region to <b>East US 2</b> and choose a Windows Server image (Windows Server 2025 Datacenter: Azure Edition in my deployment).
<br />4. Choose a size (I used Standard D2lds v7: 2 vCPUs, 4 GiB memory).
<br />5. Set the administrator username to <code>labuser</code> and enter a strong password.
<br />6. On the <b>Networking</b> tab, select <code>active-directory-vnet</code> and its subnet.
<br />7. Click <b>Review + create</b>, then <b>Create</b>.
<br /><br />
Once deployment finishes, the VM list shows <b>dc-1</b> with a status of <b>Running</b> in East US 2. This machine will become the Domain Controller in the next labs.
</p>
<br />

<p>
<img src="YOUR-IMGUR-LINK-3" height="80%" width="80%" alt="DC-1 network settings with the NIC circled"/>
</p>
<p>
<b>Step 4: Open DC-1's network interface.</b>
<br />1. Click <b>dc-1</b>, then expand <b>Networking</b> and click <b>Network settings</b>.
<br />2. Click the network interface link at the top, <b>dc-1266 (primary) / ipconfig1 (primary)</b>.
<br />3. In the NIC's menu, open <b>Settings > IP configurations</b>.
<br />4. Click <b>ipconfig1</b>.
<br /><br />
The Essentials panel confirms DC-1's current private IP address (<code>10.0.1.4</code>) and shows it is in <code>active-directory-vnet</code>.
</p>
<br />

<p>
<img src="YOUR-IMGUR-LINK-4" height="80%" width="80%" alt="IP allocation set to Dynamic"/>
</p>
<p>
<img src="YOUR-IMGUR-LINK-5" height="80%" width="80%" alt="IP allocation set to Static"/>
</p>
<p>
<b>Step 5: Set DC-1's private IP address to Static.</b>
<br />1. In the <b>Edit IP configuration</b> pane, notice that <b>Allocation</b> is set to <b>Dynamic</b> by default (first image).
<br />2. Select <b>Static</b>.
<br />3. Keep the private IP address as <code>10.0.1.4</code> (second image).
<br />4. Click <b>Save</b>.
<br /><br />
This matters because DC-1 will act as the DNS server for the network. With a dynamic address, Azure could hand DC-1 a different IP after a stop and start, and every client pointing at the old address would lose DNS. A static IP guarantees that <code>10.0.1.4</code> always belongs to DC-1.
</p>
<br />

<p>
<b>Step 6: Disable the Windows Firewall on DC-1 (for testing).</b>
<br />1. Connect to DC-1 with Remote Desktop using its public IP address and the <code>labuser</code> account.
<br />2. Press <b>Win + R</b>, type <code>wf.msc</code>, and press Enter.
<br />3. Click <b>Windows Defender Firewall Properties</b>.
<br />4. On the <b>Domain Profile</b>, <b>Private Profile</b>, and <b>Public Profile</b> tabs, set <b>Firewall state</b> to <b>Off</b>.
<br />5. Click <b>Apply</b>, then <b>OK</b>.
<br /><br />
By default the Windows Firewall blocks ICMP echo requests, so Client-1's ping would fail even if the network were correct. Turning it off removes that variable while testing connectivity. This is for lab use only; in a real environment the firewall stays on and specific rules are opened instead.
</p>
<br />

<h3>Part 3: Client (Client-1)</h3>

<p>
<img src="YOUR-IMGUR-LINK-6" height="80%" width="80%" alt="Client-1 being created next to DC-1"/>
</p>
<p>
<b>Step 7: Create the Client-1 virtual machine.</b>
<br />1. Go to <b>Virtual machines</b> and click <b>Create > Azure virtual machine</b>.
<br />2. Select the <code>active-directory-1</code> resource group and name the VM <code>Client-1</code>.
<br />3. Set the region to <b>East US 2</b>, the same region as DC-1.
<br />4. Choose a <b>Windows 10</b> image.
<br />5. Set the administrator username to <code>labuser</code> and enter a strong password.
<br />6. On the <b>Networking</b> tab, select the same virtual network as DC-1, <code>active-directory-vnet</code>.
<br />7. Click <b>Review + create</b>, then <b>Create</b>.
<br /><br />
The VM list now shows <b>client-1</b> in the Creating state next to <b>dc-1</b> (Running), both in East US 2. Placing both machines in the same region and virtual network is what allows them to communicate over private IP addresses.
</p>
<br />

<p>
<img src="YOUR-IMGUR-LINK-7" height="80%" width="80%" alt="Client-1 DNS server set to DC-1's private IP"/>
</p>
<p>
<b>Step 8: Point Client-1's DNS at DC-1.</b>
<br />1. Once Client-1 finishes deploying, click <b>client-1</b> and open <b>Networking > Network settings</b>.
<br />2. Click the network interface link (<b>client-1789</b>).
<br />3. Under <b>Settings</b>, click <b>DNS servers</b>.
<br />4. Change the setting from <b>Virtual network inherited</b> to <b>Custom</b>.
<br />5. Enter DC-1's private IP address, <code>10.0.1.4</code>, as the custom DNS server.
<br />6. Click <b>Save</b>.
<br /><br />
Active Directory depends on DNS. Clients find domain controllers by looking up DNS records, so Client-1 must use DC-1 as its DNS server before it can join a domain later.
</p>
<br />

<p>
<b>Step 9: Restart Client-1.</b>
<br />1. In the Azure portal, open the <b>client-1</b> virtual machine.
<br />2. Click <b>Restart</b> on the Overview page and confirm.
<br />3. Wait for the status to return to <b>Running</b>.
<br /><br />
The VM only picks up the new DNS server when its network settings are refreshed, and a restart from the portal is the simplest way to force that.
</p>
<br />

<h3>Part 4: Verify Connectivity</h3>

<p>
<b>Step 10: Ping DC-1 from Client-1.</b>
<br />1. Connect to Client-1 with Remote Desktop using the <code>labuser</code> account.
<br />2. Open <b>PowerShell</b>.
<br />3. Run <code>ping 10.0.1.4</code>.
<br />4. Confirm you receive replies from <code>10.0.1.4</code> with 0% packet loss.
<br /><br />
Successful replies prove the two machines can reach each other across the virtual network and that the firewall on DC-1 is no longer blocking ICMP. If the ping times out, recheck that the Windows Firewall is off on DC-1 and that both VMs are in the same virtual network.
</p>
<br />

<p>
<img src="YOUR-IMGUR-LINK-8" height="80%" width="80%" alt="ipconfig /all output showing DNS server 10.0.1.4"/>
</p>
<p>
<b>Step 11: Confirm the DNS setting with ipconfig.</b>
<br />1. In PowerShell on Client-1, run <code>ipconfig /all</code>.
<br />2. Find the <b>DNS Servers</b> line under the Ethernet adapter.
<br />3. Confirm it shows <code>10.0.1.4</code> (circled), DC-1's private IP address.
<br /><br />
This confirms the Azure DNS change reached the operating system after the restart, so Client-1 will send its DNS queries to DC-1.
</p>
<br />

<h2>Cost Management</h2>

Do not delete the virtual machines. They are reused in the upcoming Active Directory labs. When you are finished for the day, open each VM in the Azure portal and click <b>Stop</b> so it shows as <b>Stopped (deallocated)</b>. Deallocated VMs stop incurring compute charges, and because DC-1's private IP is static, it will still be <code>10.0.1.4</code> when you start it again.

<h2>Summary</h2>

By the end of this lab, Azure hosts a resource group and virtual network containing DC-1, a Windows Server VM with a static private IP and its firewall disabled for testing, and Client-1, a Windows 10 VM on the same network using DC-1 as its DNS server. A successful ping and the <code>ipconfig /all</code> output verify that the foundation is ready for installing Active Directory Domain Services and joining the client to a domain.
