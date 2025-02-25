# Active Directory

## Project Overview

This project documents the setup of an **Active Directory (AD) environment** on **Windows Server 2022** in a local lab. It includes the installation, configuration, and management of a single-domain AD infrastructure, along with a **Splunk Server** on Ubuntu, a **Windows Machine**, and an **Attacker Machine running Kali Linux**.

## Logical Diagram

![image](https://github.com/user-attachments/assets/22e954b1-9901-4f5c-b6ce-c873ea83cb3f)

## Lab Setup

### Hardware/Software Requirements

- **Virtualization Platform**: VMware Workstation / VirtualBox / Hyper-V
- **OS**:
  - Windows Server 2022 (Domain Controller)
  - Ubuntu (Splunk Server)
  - Windows 10/11 (Client Machine)
  - Kali Linux (Attacker Machine)
- **RAM**: Minimum 4GB (Server), 2GB (Client), 4GB (Kali & Splunk)
- **Storage**: Minimum 40GB for each VM

### Creating Virtual Machines (VMs)

Create the following VMs in **VMWare Workstation Pro**:

- **Windows Server 2022**
- **Ubuntu Server**
- **Windows Client Machine**
- **Kali Linux Attacker Machine**

### Setting Up NAT Network in VMWare Workstation Pro

1. Go to **Edit > Virtual Network Editor**.
2. Choose **VMnet8**, then click **Change Settings** in the bottom right corner.
3. Set the **Subnet IP** to `192.168.10.0`.

![NAT Network Configuration](https://github.com/user-attachments/assets/10676b08-93f4-405c-a0f8-0738babdbfb3)

4. Set the **default gateway** as `192.168.10.1`.

![image](https://github.com/user-attachments/assets/df3a6bf6-9a86-4812-bf8b-7d52f3eca6bc)

### Setting Network Configuration for VMs

Set the **Network Setting** of all the VMs to **NAT network**.

![image](https://github.com/user-attachments/assets/66d462ae-3435-4874-a6a9-8ff939180efa)

## Installation Steps

## 1. Setting Up Splunk Server

### Install Splunk Enterprise on Ubuntu Server

- Navigate to [Splunk Products > Splunk Enterprise](https://www.splunk.com/en_us/download/splunk-enterprise.html)
- Copy the `.deb` wget link.
- Paste the wget link into the Ubuntu Server VM:
  ```sh
  wget -O splunk-9.4.0-6b4ebe426ca6-linux-amd64.deb "https://download.splunk.com/products/splunk/releases/9.4.0/linux/splunk-9.4.0-6b4ebe426ca6-linux-amd64.deb"
  ```
- Install the package:
  ```sh
  sudo dpkg -i splunk-9.4.0-6b4ebe426ca6-linux-amd64.deb
  ```
- Navigate to the Splunk installation directory:
  ```sh
  cd /opt/splunk
  ```
- List all files to verify ownership by the Splunk user and group:
  ```sh
  ls -la
  ```
- Switch to the Splunk user:
  ```sh
  sudo -u splunk bash
  ```
- Run the Splunk installer:
  ```sh
  cd /opt/splunk/bin  
  ./splunk start  
  ```
- Agree to the license agreement when prompted.
- Exit the Splunk user:
  ```sh
  exit
  ```
- Enable Splunk to start automatically on VM boot:
  ```sh
  sudo /opt/splunk/bin/splunk enable boot-start -user splunk
  ```


### Configure Static IP for Splunk Server

- Check the IP address of the Splunk server:
  ```sh
  ip a
  ```
- To set the static IP address to `192.168.10.10`, navigate to `/etc/netplan` and edit the `.yaml` file:
  ```sh
  sudo nano <filename>.yaml
  ```
- Make the following changes as shown in the image below.

![image](https://github.com/user-attachments/assets/d7a9a732-4c0e-47fc-aee9-12fbbf947f64)

- After making the changes, apply the configuration:
  ```sh
  sudo netplan apply
  ```
- Do a **ping request** to check if the network is working:
  ```sh
  ping 192.168.10.1
  ```
## 2. Setting Up Windows Server 2022

### Renaming the Windows Server
The following steps are similar to those performed for the Windows client machine.
1. Open **Settings** > **System** > **About**.
2. Click **Rename this PC** and enter the new name. We'll be naming it as `ADDC01`.
3. Click **Next** and **Restart** the machine to apply changes.

### Configuring Network Settings (static IP recommended)
The steps here are the same as those done for the Windows client machine, except for choosing the IP address.
1. Open **Control Panel** > **Network and Internet** > **Network and Sharing Center**.
2. Click **Change adapter settings**.
3. Right-click on your active network adapter and select **Properties**.
4. Select **Internet Protocol Version 4 (TCP/IPv4)** and click **Properties**.
5. Choose **Use the following IP address** and enter the details:
    - **IP Address**: `192.168.10.7`
    - **Subnet Mask**: `255.255.255.0`
    - **Default Gateway**: `192.168.10.1`
7. Choose **Use the following DNS server addresses** and enter:
    - **Preferred DNS server**: `8.8.8.8`
8. Click **OK** and close all windows.
  
  ![Screenshot 2025-02-22 044454](https://github.com/user-attachments/assets/44af577c-c04d-42ce-a47f-3f3f2379c765)

9. Open Command Prompt and verify with:
     ```sh
     ipconfig /all
     ```
### Installing and Configuring Sysmon

- Download Sysmon from Microsoft's official Sysinternals website: [Download Sysmon](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon).
- Download the Sysmon configuration file: [Sysmon Config](https://raw.githubusercontent.com/olafhartong/sysmon-modular/master/sysmonconfig.xml)
- Extract the Sysmon zip file to the `C:\` drive.
- Move the downloaded `sysmonconfig.xml` file to the `C:\Sysmon` folder.
- Open PowerShell as Administrator and navigate to the Sysmon directory:
  ```powershell
  cd C:\Sysmon
  ```
- To install Sysmon with a custom configuration, use:
  ```powershell
  .\sysmon64.exe -i sysmonconfig.xml
  ```
- Verify that Sysmon is running by checking its status:
  ```powershell
  Get-Service Sysmon
  ```
- Check logs in Event Viewer under `Applications and Services Logs > Microsoft > Windows > Sysmon`.
  
### Installing Splunk Universal Forwarder

- Download **Splunk Universal Forwarder** from the official [Splunk website](https://www.splunk.com/en_us/download/universal-forwarder.html).

- Run the installer and follow the setup wizard.


![image](https://github.com/user-attachments/assets/afdf4327-fad5-4f4a-b597-04b08cc8eb4b)


![image](https://github.com/user-attachments/assets/0a22f63d-6332-401d-af1d-99b5ddb70cce)


![image](https://github.com/user-attachments/assets/34983e67-4560-4d35-b8b5-6945c9ea5a1e)


![image](https://github.com/user-attachments/assets/d1cae5bf-9297-4110-84d7-0530b547fe9b)


- Click Install

- Complete the installation and start the **Splunk Universal Forwarder** service.
- Open **Notepad as Administrator**.
- Copy and paste the following content into Notepad:
  ```ini
  [WinEventLog://Application]
  index = endpoint
  disabled = false

  [WinEventLog://Security]
  index = endpoint
  disabled = false

  [WinEventLog://System]
  index = endpoint
  disabled = false

  [WinEventLog://Microsoft-Windows-Sysmon/Operational]
  index = endpoint
  disabled = false
  renderXml = true
  source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
  ```
- Save the file as **inputs.conf** in `C:\Program Files\SplunkUniversalForwarder\etc\system\local`.
- **Steps to Modify Splunk Forwarder Service Log On Settings**
  - Search **Services** and run it as Administrator.
  - Locate **SplunkForwarder** in the list.
  - **Double-click** on the service to open its properties window.
  - Navigate to the **Log On** tab.
  - Select **Local System account** instead of the default **NT Service account**.
    - This ensures the forwarder has the necessary permissions to collect logs from all system locations.

  ![image](https://github.com/user-attachments/assets/6d1a026a-4472-4e6c-a0b7-c6a3b53bc18a)

  
  - Click **Apply**, then **OK** to save the changes
  - Right Click and Restart the service
   

### Configuring Active Directory Domain Services (AD DS)

- Open **Server Manager**.
- Go to **Manage** and click **Add Roles and Features**.
- Select **Role-based or feature-based installation** and click **Next**.
- Choose **Select a server from the server pool**, then select your server and click **Next**.
- Select **Active Directory Domain Services** and click **Add Features** when prompted.
- Click **Next** through the remaining prompts and then click **Install**.
- After installation, click **Promote this server to a domain controller**.

  ![image](https://github.com/user-attachments/assets/cbd4bd1c-14f9-480c-b586-63615f66d8ef)

- Select **Add a new forest** and provide a **Root domain name**.
- Click **Next** and configure the domain and forest functional levels.
- Set a **Directory Services Restore Mode (DSRM) password**.
- Complete the wizard and restart the server.
- Verify AD installation using:
  ```sh
  dcdiag
  ```
  
### Create Users, and OUs in AD

- Go to **Tools > Active Directory Users and Computers (ADUC)**.
- Create new **Organizational Units (OUs)** to organize users and groups:
  1. Right-click on the domain name and select **New > Organizational Unit**.
  2. Name the OU (e.g., "Users", "Groups", "Admins"). We'll name it as IT.
  3. Click **OK**.
  4. Repeat the process to create another OU named **HR**.
- Create **Users**:
  1. Navigate to the **IT** OU.
  2. Right-click, select **New > User**.
  3. Fill in the user details and set a password.
  4. Click **Next**, then **Finish**.
  5. Navigate to the **HR** OU and repeat the steps to create a user in the HR department.

## 2. Setting Up Windows Client Machine

These steps mirror those taken for setting up the Windows Server.

### Renaming Windows Client Machine

1. Open **Settings** > **System** > **About**.
2. Click **Rename this PC** and enter the new name (e.g., `WIN10-CLIENT`).
3. Click **Next** and **Restart** the machine to apply changes.

### Configuring Network Settings

1. Open **Control Panel** > **Network and Internet** > **Network and Sharing Center**.
2. Click **Change adapter settings**.
3. Right-click on the active network adapter and select **Properties**.
4. Select **Internet Protocol Version 4 (TCP/IPv4)** and click **Properties**.
5. Choose **Use the following IP address** and enter the details:
    - **IP Address**: `192.168.10.7`
    - **Subnet Mask**: `255.255.255.0`
    - **Default Gateway**: `192.168.10.1`
6. Choose **Use the following DNS server addresses** and enter:
    - **Preferred DNS server**: `8.8.8.8`
7. Click **OK**, then **Close**.

### Installing and Configuring Sysmon

Follow the same steps used for Windows Server to install and configure Sysmon. 

### Installing Splunk Universal Forwarder  

Follow the same steps used for Windows Server to install and configure the Splunk Universal Forwarder.  

### Installing Atomic Red Team on Windows Machine

- Open PowerShell as Administrator.
- Set the execution policy to bypass for the current user:
  ```powershell
  Set-ExecutionPolicy Bypass CurrentUser
  ```
- Add an exclusion for the C:\ drive in Microsoft Defender to prevent it from removing Atomic Red Team files.
  - Open Windows Security.
  - Go to Virus & Threat Protection.
  - Click Manage Settings under Virus & Threat Protection Settings.
  - Scroll down to Exclusions and click Add or remove exclusions.
  - Click Add an exclusion, select Folder, and choose `C:\`.

- Run the following command to install Atomic Red Team:
  ```powershell
  IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing);
  Install-AtomicRedTeam
  ```
  
- Verify the installation by running:
  ```powershell
  Invoke-AtomicTest T1003 -CheckPrereqs
  ```

### Joining Windows Client Machine to Active Directory

#### Configure DNS Settings
- Open **Control Panel > Network and Internet > Network and Sharing Center**.
- Click **Change adapter settings**.
- Right-click on the active network adapter, select **Properties**.
- Select **Internet Protocol Version 4 (TCP/IPv4)** and click **Properties**.
- Set **Preferred DNS Server** to `192.168.10.7`.
- Click **OK**, then **Close**.

#### Join the Windows Client to the Domain 
1. Open **Settings** > **System** > **About**.
2. Click **Advanced system settings**.
3. In the **System Properties** window, go to the **Computer Name** tab and click **Change**.
4. Under **Member of**, select **Domain**, and enter the domain name **V101DEV.LOCAL**.
5. Click **OK** and enter Domain Administrator Credentials.
6. Restart the client machine.

#### Verify Authentication with Domain Credentials
- Click **Other User** on the login screen.
- Enter **domain credentials** (`yourdomain\username`).
- Click **Sign in**.
- Run `whoami` in Command Prompt to confirm domain authentication.

### Enable Remote Desktop Protocol (RDP) on the Windows Machine

To allow remote access and perform brute-force testing from the attacker machine, enable RDP on the Windows client.

- Open **Settings > System > About**.
- Click **Advanced system settings**.
- In the **System Properties** window, go to the **Remote** tab and click **Allow remote connections to this computer**.
- Click **Select Users...** to add users who can access the machine remotely.
- In the **Remote Desktop Users** window, click **Add**.
- Enter the username or click **Advanced > Find Now** to search for a user in Active Directory.
- Select the user and click **OK**.
- Click **OK** again to close all windows and apply the changes.


## 3. Creating an Index in Splunk Dashboard  

- Go to `192.168.10.10:8000` and sign in with the credentials of Splunk Enterprise. The dashboard will open.

- In the dashboard, go to **Settings** and select **Indexes** under **DATA**.

- Create an index **"endpoint"**.

![image](https://github.com/user-attachments/assets/e0fcbdc6-d1c4-4075-918f-89f8846dc6c8)

- Next, we need to enable our Splunk server to receive data. Go to **Settings > Forwarding and Receiving**, then under **Receiving**, click **Configure Receiving**. Click on **New Receiving Port** and add port `9997`.

![image](https://github.com/user-attachments/assets/34af7bc2-7d28-44fd-8aaa-eba122f4c1d1)


- After this step, we'll start receiving data. This can be visible if we go to **Apps** on the top left and navigate to **Search & Reporting**.

- Run the query `index="endpoint"` and hit search.

![image](https://github.com/user-attachments/assets/07619684-ad40-42e1-aba0-5ee46ae911fb)

- The following image shows how the Splunk search dashboard will look after running the query

![image](https://github.com/user-attachments/assets/3e312506-0996-40ee-9ed5-8c08f0b2a2a8)

## 4. Setting Up Kali Linux

- Start the **Kali Linux** VM and log in.
- Right-click on the **Ethernet icon** and select **Edit Connections**.
  
  ![image](https://github.com/user-attachments/assets/2be4eebc-2b3c-4f87-99e1-90bf2691d083)
  
  ![image](https://github.com/user-attachments/assets/04c1fc2a-c805-417e-85e3-7fc549c5ddcb)


- Double Click the Wired connection1 to edit it.
- Set the **IP Address** to `192.168.10.250`.
- Set the **Netmask** to `255.255.255.0` (or `/24`).
- Set the **Default Gateway** to `192.168.10.1`.
- Click **Save** and close the window.&#x20;

  ![image](https://github.com/user-attachments/assets/9ab659db-8838-42f8-8b25-b084a097424f)

- Click on the **Ethernet icon** and disconnect it. Then, reconnect it again. This step ensures that the changes take effect properly.
- Open a terminal and type `ip a` to verify the new network configuration.

  ![image](https://github.com/user-attachments/assets/ebd8244c-0a53-4d5f-887d-8150ef64b1ef)


- Inside Kali, create a directory for the project:

  ```sh
  mkdir ad-project
  ```

- Install the tool **Crowbar** for the attack:

  ```sh
  sudo apt install crowbar
  ```

- We will use the **rockyou.txt** wordlist, which comes pre-installed with Kali Linux.

- Navigate to its folder and unzip it:

  ```sh
  cd /usr/share/wordlists
  sudo gunzip rockyou.txt.gz
  ```

- Copy the `rockyou.txt` file into the `ad-project` directory:

  ```sh
  cp rockyou.txt ~/ad-project/
  ```
- We won't be using the entire wordlist but only the first 20 lines for demonstration purposes.
- Extract the first 20 lines and save them as `passwords.txt`:

  ```sh
  head -n 20 rockyou.txt > passwords.txt
  ```
- Open the `passwords.txt` file using the nano editor:

  ```sh
  nano passwords.txt
  ```

- Add the password of our Windows machine at the end of the file.

- Save and exit the editor by pressing `CTRL+X`, then `Y`, and `Enter`.

#### Running the Brute Force Attack

- Execute the brute force attack using Crowbar with the following command:

  ```sh
  crowbar -b rdp -u jenny -C passwords.txt -s 192.168.10.100/32
  ```

- This command attempts to brute-force the **Remote Desktop Protocol (RDP)** login on the target Windows machine using the credentials from `passwords.txt`.

## 5. Viewing Splunk Telemetry

- Log in to Splunk Web at `192.168.10.10:8000`.
- Navigate to Search & Reporting.
- Run the query:
  ```
  index="endpoint" jenny EventCode=4625
  ```
- Here is the image of failed attempts

  ![Screenshot 2025-02-25 013124](https://github.com/user-attachments/assets/40c479f1-224c-4d68-8297-49fcab2a90eb)

- Now, When we run the query:
  ```
  index="endpoint" jenny EventCode=4624
  ```

  ![image](https://github.com/user-attachments/assets/677d1543-14b0-4445-9c17-b2092d292531)

- It will show the successful login.

## Security Best Practices

- Enable **Account Lockout Policies**.
- Restrict **Administrative Privileges**.
- Implement **Multi-Factor Authentication (MFA)**.
- Regularly update and monitor logs with Splunk.

## Troubleshooting

| Issue                     | Solution                                          |
| ------------------------- | ------------------------------------------------- |
| Client cannot join domain | Verify network settings & DNS configuration       |
| AD DS installation failed | Check event logs & ensure prerequisites are met   |
| Splunk not receiving logs | Verify firewall rules and log forwarding settings |

## Conclusion

This project demonstrates the setup of **Active Directory on Windows Server 2022**, with a **Splunk Server on Ubuntu**, a **Windows Client**, and a **Kali Linux Attacker Machine** for security testing. Future enhancements may include **multi-domain setups, AD Federation Services, and advanced security policies**.

---

**Author**: sv-00\
**GitHub Repository**: [Active Directory](https://github.com/sv-00/Active-Directory)

