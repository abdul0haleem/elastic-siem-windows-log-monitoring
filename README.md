# elastic-siem-windows-log-monitoring

Hands-on Elastic SIEM lab demonstrating the deployment of Elasticsearch, Kibana, and Fleet Server on Kali Linux, along with Elastic Agent deployment on a Windows 11 virtual machine for centralized Windows log collection, monitoring, and analysis.

## ⚙️ Project Implementation

### In This Project, I Used

The project was implemented using the following virtual machines and Elastic Stack components:

```text
Kali Linux (VirtualBox VM)
        │
        ├── Elasticsearch
        ├── Kibana
        └── Fleet Server
                │
                ▼
Windows 11 (VMware Workstation VM)
        │
        └── Elastic Agent
                │
                ▼
           Fleet Server
```

**Kali Linux on VirtualBox VM → Elasticsearch + Kibana + Fleet Server**

**Windows 11 on VMware Workstation VM → Elastic Agent → Fleet Server**

This lab environment was designed to demonstrate the deployment of the Elastic Stack SIEM, Fleet Server configuration, Elastic Agent enrollment, and collection of Windows security, system, and application logs.

### Step 1: Configure Kali Linux on VirtualBox

Kali Linux was configured as a virtual machine using **Oracle VirtualBox**. This virtual machine was used as the primary system for deploying the Elastic Stack components, including **Elasticsearch, Kibana, and Fleet Server**.

The Kali Linux virtual machine was successfully started and prepared for the subsequent stages of the project implementation.

![Screenshot 1: Kali Linux running on a VirtualBox Virtual Machine](images/01-kali-virtualbox.png)

*Figure 1: Kali Linux virtual machine configured in VirtualBox.*

### Step 2: Check Kali Linux Version

After configuring the Kali Linux virtual machine, the operating system version was verified using the Terminal. The following command was executed:

```bash
cat /etc/os-release
```

The command displays information about the installed operating system, including its name and version. The output confirmed that the system was running **Kali Linux 2026.3 (kali-rolling)**.

![Screenshot 2: Showing the Kali Linux Version](images/02-kali-version.png)

*Figure 2: Kali Linux version information displayed in the Terminal, confirming Kali Linux 2026.3 (kali-rolling).*

### Step 3: Update Kali Linux

Before installing **Elasticsearch and Kibana**, the Kali Linux system was updated to ensure that the package repositories and existing software packages were up to date.

The following command was executed in the Terminal:

```bash
sudo apt update && sudo apt upgrade -y
```

The `apt update` command refreshes the local package lists with the latest information from the configured repositories. The `apt upgrade -y` command then upgrades the installed packages to their latest available versions. The update and upgrade process was allowed to complete before proceeding with the Elastic Stack installation.

![Screenshot 3: Showing Kali Linux Package Update Process](images/03-kali-package-update.png)

*Figure 3: Kali Linux packages being updated and upgraded through the Terminal.*

### Step 4: Install Required Dependencies

Before installing the Elastic Stack components, the required packages for repository configuration and package installation were installed on the Kali Linux system.

The following command was executed in the Terminal:

```bash
sudo apt install apt-transport-https wget gnupg -y
```

These packages provide the basic utilities required for the installation process. `apt-transport-https` enables package retrieval over HTTPS, `wget` is used to download files from the internet, and `gnupg` is used for handling and verifying repository signing keys.

The installation completed successfully, and the required dependencies were ready for the next stage of the Elastic Stack setup.

![Screenshot 4: Showing the Required Dependencies Installed](images/04-required-dependencies.png)

*Figure 4: Required packages successfully installed on Kali Linux using the APT package manager.*

### Step 5: Add the Elastic Package Repository

The official Elastic package repository was configured on the Kali Linux system. This process involved two parts: first, downloading and storing Elastic's official GPG signing key, and then adding the Elastic 9.x APT repository. This allows the system to securely download and install Elastic Stack packages through the APT package manager.

#### 5.1 Add the Elastic GPG Signing Key

The following command was executed to download Elastic's official GPG signing key and store it as a keyring:

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
```

The GPG key is used by APT to verify the authenticity of packages downloaded from the Elastic repository.

![Screenshot 5: Showing the Elastic GPG Key Command](images/05-elastic-gpg-key.png)

*Figure 5: Elastic's official GPG signing key being downloaded and stored as a keyring.*

#### 5.2 Add the Elastic 9.x Repository

After adding the signing key, the official Elastic 9.x APT repository was added using the following command:

```bash
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-9.x.list
```

This creates the repository configuration file:

```text
/etc/apt/sources.list.d/elastic-9.x.list
```

The `signed-by` option specifies the Elastic GPG keyring that APT uses to verify packages obtained from the repository.

![Screenshot 6: Showing the Elastic Repository Added](images/06-elastic-repository.png)

*Figure 6: Elastic 9.x APT repository successfully added to the Kali Linux package sources.*

### Step 6: Refresh the Package List

After adding the Elastic package repository, the APT package list was refreshed to ensure that Kali Linux could access the newly configured Elastic repository.

The following command was executed:

```bash
sudo apt update
```

During the update process, the output was checked for an entry similar to:

```text
https://artifacts.elastic.co/packages/9.x/apt stable
```

The update completed without any **GPG, signature, or repository errors**, confirming that the Elastic repository was configured correctly and was accessible through the APT package manager.

### Step 7: Install Elasticsearch

The next step was to install **Elasticsearch** on the Kali Linux virtual machine. Elasticsearch serves as the central data storage and indexing component of the Elastic Stack. It will store and index the security data collected from the Windows 11 endpoint through the Elastic Agent.

The following command was executed:

```bash
sudo apt install elasticsearch -y
```

The installation process may take some time to complete. During installation, Elasticsearch automatically generated a password for the built-in **`elastic` superuser account**.

The generated password was copied and stored temporarily because it is required for authentication during the subsequent configuration and setup of the Elastic Stack.

![Screenshot 7: Showing Elasticsearch Installation and Generated Password](images/07-elasticsearch-installation-password.png)

*Figure 7: Elasticsearch being installed on Kali Linux, with the automatically generated password for the built-in `elastic` superuser displayed in the Terminal.*

