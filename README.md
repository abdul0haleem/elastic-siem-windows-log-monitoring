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

### Step 8: Enable Elasticsearch as a System Service

After installing Elasticsearch, the systemd service configuration was reloaded to ensure that the newly installed Elasticsearch service was recognized by the system.

The following command was executed:

```bash
sudo systemctl daemon-reload
```

Next, Elasticsearch was enabled as a system service so that it would **start automatically whenever the Kali Linux system boots**.

```bash
sudo systemctl enable elasticsearch.service
```

The output confirmed that the Elasticsearch service was successfully enabled. This ensures that Elasticsearch does not need to be manually started after every system restart.

![Screenshot 8: Showing Elasticsearch Configured to Start Automatically](images/08-elasticsearch-auto-start.png)

*Figure 8: Elasticsearch successfully enabled as a system service to start automatically with Kali Linux.*

### Step 9: Start Elasticsearch

After enabling Elasticsearch as a system service, the Elasticsearch service was started manually using the following command:

```bash
sudo systemctl start elasticsearch.service
```

The status of the service was then checked to verify that Elasticsearch was running successfully:

```bash
sudo systemctl status elasticsearch.service
```

The output was checked for the following status:

```text
Active: active (running)
```

The **active (running)** status confirms that Elasticsearch started successfully and is currently running on the Kali Linux system.

Any warnings displayed in the output were Java/Elasticsearch-related warnings and did not prevent the Elasticsearch service from running successfully.

![Screenshot 9: Showing Elasticsearch Service Status](images/09-elasticsearch-service-status.png)

*Figure 9: Elasticsearch service status showing Active: active (running), confirming that Elasticsearch is running successfully.*

### Step 10: Find the CA Certificate

The next step was to locate the **Certificate Authority (CA) certificate** generated by Elasticsearch. This certificate will be required for secure communication with Elasticsearch in the subsequent configuration steps.

The following command was executed:

```bash
sudo find /etc/elasticsearch -name "http_ca.crt"
```

The command searches the Elasticsearch configuration directory for the `http_ca.crt` file and displays its exact location.

The command returned the following path:

```text
/etc/elasticsearch/certs/http_ca.crt
```

The exact path returned by the system was noted for use in the upcoming steps.

![Screenshot 10: Showing the Elasticsearch CA Certificate Path](images/10-elasticsearch-ca-certificate.png)

*Figure 10: Terminal output displaying the location of the Elasticsearch `http_ca.crt` CA certificate.*

### Step 11: Test Elasticsearch

After locating the CA certificate, Elasticsearch was tested to verify that the service was accessible over its secure HTTPS interface.

The following command was executed with root privileges:

```bash
sudo curl --cacert /etc/elasticsearch/certs/http_ca.crt -u elastic https://localhost:9200
```

The command uses the previously identified CA certificate to establish a trusted HTTPS connection with Elasticsearch running on **localhost at port 9200**. The `-u elastic` option specifies the built-in `elastic` superuser for authentication.

When prompted for the password, the **current password for the `elastic` user** was entered.

![Screenshot 11: Showing Prompt for Entering the Password](images/11-elasticsearch-password-prompt.png)

*Figure 11: Terminal prompt requesting the password for the `elastic` superuser during Elasticsearch authentication.*

### Step 12: Reset the `elastic` Password

The password for the built-in **`elastic` superuser** was reset to generate a new password for authentication with Elasticsearch.

The following command was executed:

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

The command displayed a confirmation prompt asking whether to continue. After confirming, Elasticsearch generated a **new password for the `elastic` user**.

The newly generated password was copied and stored temporarily because it would be required in the next step for authenticating with Elasticsearch.

![Screenshot 12: Showing the Reset elastic Password](images/12-reset-elastic-password.png)

*Figure 12: Terminal output showing the password reset process and the newly generated password for the `elastic` superuser.*

### Step 13: Test Elasticsearch Again

After resetting the `elastic` user password, Elasticsearch was tested again to verify that the service, HTTPS connection, and authentication were working correctly.

The following command was executed:

```bash
sudo curl --cacert /etc/elasticsearch/certs/http_ca.crt -u elastic https://localhost:9200
```

When prompted:

```text
Enter host password for user 'elastic':
```

the **newly generated password** from Step 12 was entered.

The command returned a successful Elasticsearch response, confirming that the Elasticsearch API was reachable and authentication was working correctly.

![Screenshot 13: Showing Successful Elasticsearch Verification](images/13-elasticsearch-verification.png)

*Figure 13: Successful Elasticsearch API response showing Elasticsearch version 9.5.3, the `elasticsearch` cluster, and successful authenticated HTTPS communication.*

#### Verification

The output confirmed the following:

* **Elasticsearch version:** 9.5.3
* **Cluster name:** `elasticsearch`
* **Cluster UUID:** Generated successfully
* **Security authentication:** Working
* **HTTPS certificate:** Working
* **Elasticsearch API:** Reachable on port 9200

**Result:** Elasticsearch was successfully installed, configured, secured, and verified. The Elasticsearch component of the lab is ready for the next stage of the Elastic Stack deployment.

### Step 14: Install Kibana

After successfully installing and verifying Elasticsearch, the next step was to install **Kibana 9.x**. Kibana provides the web-based interface for interacting with the Elastic Stack and will be used later to view, search, analyze, and visualize the security data collected from the Windows endpoint.

The following command was executed:

```bash
sudo apt install kibana -y
```

The installation was allowed to complete. The installed version was **Kibana 9.5.3**.

![Screenshot 14: Showing Kibana Installation](images/14-kibana-installation.png)

*Figure 14: Kibana 9.5.3 being installed successfully on the Kali Linux virtual machine.*

**Result:** Kibana 9.5.3 was successfully installed and is ready for configuration in the next step.

### Step 15: Generate a Kibana Enrollment Token

Because Elasticsearch security is enabled, a **Kibana enrollment token** was generated to securely connect Kibana with the Elasticsearch instance.

The following command was executed:

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

The command generated a long **Kibana enrollment token**. The token was copied and stored temporarily because it would be required during the Kibana configuration process in the next step.

![Screenshot 15: Showing the Kibana Enrollment Token](images/15-kibana-enrollment-token.png)

*Figure 15: Terminal output displaying the generated Kibana enrollment token.*

**Result:** The Kibana enrollment token was successfully generated and is ready to be used for the Kibana configuration.

### Step 16: Configure Kibana

The next step was to configure Kibana so that it can connect securely to the Elasticsearch instance.

The Kibana configuration file was opened using the following command:

```bash
sudo nano /etc/kibana/kibana.yml
```

This configuration file contains the settings required to configure Kibana, including its network address and connection details for Elasticsearch.

The file was opened in the **Nano text editor** so that the required configuration changes could be made in the following steps.

![Screenshot 16: Showing the Kibana Configuration File](images/16-kibana-config-file.png)

*Figure 16: Kibana configuration file `kibana.yml` opened in the Nano text editor.*

### Step 17: Configure the Kibana Server Address

The Kibana configuration file was modified to allow Kibana to be accessed through the lab network, including from the Windows 11 virtual machine.

The following line was located in `/etc/kibana/kibana.yml`:

```yaml
#server.host: "localhost"
```

It was changed to:

```yaml
server.host: "0.0.0.0"
```

Setting the server address to `0.0.0.0` allows Kibana to listen for connections on the available network interfaces, making it accessible from a browser on the lab network.

At this stage, the **Elasticsearch connection settings were left unchanged**. No `elasticsearch.hosts` or authentication credentials were manually added because the **Kibana enrollment token** generated in Step 14 will be used to establish the secure connection with Elasticsearch.

#### Save the Configuration

After making the change, the file was saved in Nano using:

1. Press **Ctrl + O**
2. Press **Enter** to confirm the filename
3. Press **Ctrl + X** to exit Nano

The configuration file was then closed, and no further changes were made at this stage.

![Screenshot 17: Showing the Kibana Server Address Configuration](images/17-kibana-server-address.png)

*Figure 17: Kibana configuration file showing `server.host: "0.0.0.0"` configured in `kibana.yml`.*
