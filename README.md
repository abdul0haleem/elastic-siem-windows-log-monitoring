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

### Step 18: Enroll Kibana with Elasticsearch

The next step was to securely connect **Kibana** with the previously configured Elasticsearch instance using the **Kibana enrollment token** generated in Step 14.

The Kibana setup utility was started using:

```bash
sudo /usr/share/kibana/bin/kibana-setup
```

The setup process prompted for an enrollment token:

```text
Enter your enrollment token:
```

The enrollment token generated earlier using the following command was entered:

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

The enrollment process successfully configured Kibana and established its secure connection with Elasticsearch.

![Screenshot 18: Showing Kibana Enrollment with Elasticsearch](images/18-kibana-enrollment-with-elasticsearch.png)

*Figure 18: Kibana setup process showing the enrollment token being entered and Kibana successfully enrolled with Elasticsearch.*

**Result:** Kibana was successfully configured and enrolled with Elasticsearch.

### Step 19: Enable Kibana to Start Automatically

After successfully enrolling Kibana with Elasticsearch, the Kibana system service was configured to start automatically whenever the Kali Linux system boots.

First, the systemd service configuration was reloaded:

```bash
sudo systemctl daemon-reload
```

Next, Kibana was enabled as a system service:

```bash
sudo systemctl enable kibana.service
```

This ensures that Kibana will automatically start when the Kali Linux virtual machine is restarted.

![Screenshot 19: Showing Kibana Configured to Start Automatically](images/19-kibana-auto-start.png)

*Figure 19: Kibana successfully enabled as a system service to start automatically with Kali Linux.*

### Step 20: Start Kibana

After enabling Kibana as a system service, the Kibana service was started manually using the following command:

```bash
sudo systemctl start kibana.service
```

The status of the service was then checked to verify that Kibana was running successfully:

```bash
sudo systemctl status kibana.service
```

Kibana may take approximately **30–60 seconds** to fully initialize. The service status was checked for:

```text
Active: active (running)
```

The **active (running)** status confirms that Kibana started successfully and is running on the Kali Linux system.

![Screenshot 20: Showing Kibana Service Status](images/19-kibana-auto-start.png)

*Figure 20: Kibana service successfully started and ready for browser-based access.*

**Result:** Kibana was successfully started and is ready for browser-based access and further configuration.

### Step 21: Open Kibana in the Browser

After confirming that the Kibana service was running successfully, the Kibana web interface was accessed through a web browser on the Kali Linux virtual machine.

The following address was entered in the browser:

```text
http://localhost:5601
```

The **Kibana login page** was displayed, requesting authentication with the built-in **`elastic`** user.

**Username:**

```text
elastic
```

**Password:**

The password generated for the **`elastic`** user during the password reset performed in **Step 11** was entered.

![Screenshot 21: Showing Kibana Login Page](images/20-kibana-login-page.png)

*Figure 21: Kibana login page displayed in the browser, requesting authentication with the built-in **`elastic`** user.*

After entering the correct credentials, the authentication was successful and the **Kibana web interface** was displayed.

![Screenshot 22: Showing Kibana After Successful Login](images/23-kibana-successful-login.png)

*Figure 22: Kibana web interface successfully opened after authentication, confirming that Kibana is accessible and ready for further configuration.*

**Result:** Kibana was successfully accessed through the browser and the **`elastic`** user authentication was completed successfully.

### Step 23: Open Fleet

After successfully logging in to Kibana, the main Kibana navigation menu was opened to access **Fleet**. Fleet is used to manage **Elastic Agents**, configure **Agent Policies**, and manage endpoint integrations.

The following navigation path was used:

**☰ Menu → Management → Fleet**

The **Fleet** page provides the management interface required to configure the Elastic Agent and create the Agent Policy that will later be used for the Windows 11 endpoint.

![Screenshot 23: Showing Kibana Fleet Navigation](images/22-kibana-fleet-navigation.png)

*Figure 23: Kibana navigation panel displaying the **Fleet** option under **Management**.*

**Result:** The Fleet management interface was successfully accessed and is ready for Fleet Server and Elastic Agent configuration.

### Step 24: Add the Fleet Encryption Key

Although Elasticsearch and Kibana were configured and running successfully, Fleet requires an **encrypted saved objects encryption key** to securely store sensitive Fleet-related information.

Open the Kibana configuration file using:

```bash
sudo nano /etc/kibana/kibana.yml
```

Add the following line at the bottom of the file:

```yaml
xpack.encryptedSavedObjects.encryptionKey: "kali-elastic-soc-lab-encryption-key-2026"
```

The encryption key must be **at least 32 characters long**.

![Screenshot 24: Showing Kibana Encryption Key Configuration](images/25-kibana-encryption-key.png)

*Figure 24: Kibana configuration file showing the `xpack.encryptedSavedObjects.encryptionKey` configured in `kibana.yml`.*

Save the configuration file using:

**Ctrl + O → Enter → Ctrl + X**

This configures the encryption key required by Fleet to securely manage encrypted saved objects.

**Result:** The Fleet encryption key was successfully added to `kibana.yml`, completing the required encryption configuration for Fleet.

### Step 25: Restart Kibana and Access Fleet

After adding the Fleet encryption key to the Kibana configuration, the Kibana service was restarted to apply the new configuration.

Run the following command:

```bash
sudo systemctl restart kibana
```

Wait approximately **30–60 seconds** for Kibana to fully restart. The service status can be verified using:

```bash
sudo systemctl status kibana
```

Once Kibana is running, refresh the browser and access the Kibana web interface:

```text
http://localhost:5601
```

Log in using the **`elastic`** user credentials:

- **Username:** `elastic`
- **Password:** The configured `elastic` user password

After successful authentication, navigate to:

**☰ Menu → Management → Fleet**

The Fleet management page should now display the **Add Fleet Server** option, indicating that Fleet is ready for the next stage of configuration.

![Screenshot 25: Showing Kibana Fleet Add Fleet Server](images/24-fleet-add-server.png)

*Figure 25: Fleet management interface displaying the **Add Fleet Server** option, confirming that Fleet is successfully enabled and ready for Fleet Server configuration.*

**Result:** Kibana was successfully restarted with the new encryption configuration, and the Fleet management interface was accessed successfully.

### Step 26: Add a Fleet Server

The **Add a Fleet Server** option was selected to begin configuring Fleet Server for the Elastic SIEM lab.

On the **Add a Fleet Server** page, **Quick Start** was selected. A new Fleet Server policy was created with the following configuration:

- **Policy Name:** `SOC-Lab-Fleet-Server`
- **Other Settings:** Kept at their default values
- **Fleet Server URL:** `https://192.168.29.106:8220`

The following ports were used in the Elastic Stack deployment:

- **9200** → Elasticsearch
- **5601** → Kibana
- **8220** → Fleet Server

The communication flow for the lab environment is:

```text
Windows 11 Elastic Agent
        ↓
https://192.168.29.106:8220
        ↓
Fleet Server (Kali Linux)
        ↓
Elasticsearch :9200
        ↓
Kibana :5601
```

After completing the Fleet Server configuration, Kibana confirmed that the **Fleet Server policy** was created successfully. The required service token was generated, and the Fleet Server host was configured as:

```text
https://192.168.29.106:8220
```

![Screenshot 26: Showing Fleet Server Configuration](images/26-fleet-server-policy.png)

*Figure 26: Fleet Server policy successfully created and configured with the Kali Linux Fleet Server endpoint.*

**Result:** The `SOC-Lab-Fleet-Server` policy was successfully created, and the Fleet Server host and service token were generated. The next stage is to install and start Fleet Server on the Kali Linux machine.

### Step 27: Download the Correct Elastic Agent

Before downloading the Elastic Agent, the Kali Linux system architecture was checked to ensure that the correct package was selected.

Run:

```bash
uname -m
```

The command should return:

```text
x86_64
```

![Screenshot 27: Kali Linux Terminal displaying the output of uname -m](images/27-kali-architecture.png)

*Figure 27: Kali Linux Terminal displaying the system architecture as `x86_64`.*

Next, return to the **Fleet → Add a Fleet Server** page and select the appropriate options:

* **Operating system:** Linux
* **Architecture:** x86_64
* **Version:** 9.5.3

The download package should be similar to:

```text
elastic-agent-9.5.3-linux-x86_64.tar.gz
```

![Screenshot 28: Fleet Add a Fleet Server page](images/28-fleet-server-platform-selection.png)

*Figure 28: Fleet **Add a Fleet Server** page showing Linux, x86_64, and Elastic Agent version 9.5.3.*

After selecting the correct package, click **Save and continue** / **Continue** to proceed with the Fleet Server installation.

### Step 28: Run the Fleet Server Installation Command

Return to the **Fleet → Add a Fleet Server** page and copy the **Linux x86_64 Fleet Server installation command** provided by Kibana.

Run the copied command in the Kali Linux Terminal to install and enroll the Elastic Agent as a **Fleet Server**.

![Screenshot 29: Fleet page displaying the Linux x86\_64 Fleet Server installation command](images/29-fleet-server-installation-command.png)

*Figure 29: Fleet page displaying the Linux x86_64 Fleet Server installation command used to install and enroll the Elastic Agent on Kali Linux.*

### Step 29: Verify the Elastic Agent Service

The Elastic Agent service was further verified using the following command:

```bash
sudo systemctl status elastic-agent --no-pager
```

The expected output should show:

```text
Active: active (running)
```

![Screenshot 30: Terminal displaying the Elastic Agent service status](images/30-elastic-agent-service-status.png)

*Figure 30: Terminal displaying the Elastic Agent service status as `Active: active (running)`.*

Next, Fleet Server was verified to be listening on port **8220** using:

```bash
sudo ss -tulpn | grep 8220
```

Port **8220** is the communication port used by Fleet Server for Elastic Agent management and communication.

![Screenshot 31: Terminal output showing Fleet Server listening on port 8220](images/31-fleet-server-port-8220.png)

*Figure 31: Terminal output showing Fleet Server actively listening on port 8220.*

These checks confirm that the **Elastic Agent service is running and Fleet Server is actively listening on port 8220**.

### Step 30: Check Fleet in Kibana

After installing and starting Fleet Server, its status was verified from the Kibana Fleet interface.

Navigate to:

**Kibana → Fleet → Agents**

The Kali Linux machine should be listed as a **Fleet Server/Agent**. Its status should ideally appear as:

**Healthy 🟢**

A **Healthy** status confirms that the Fleet Server is successfully enrolled, running, and communicating with Kibana and Elasticsearch.

![Screenshot 32: Kibana Fleet Agents page showing the Kali Linux Fleet Server](images/32-kali-fleet-server-healthy.png)

*Figure 32: Fleet Agents page confirming that the Kali Linux Fleet Server is healthy and successfully enrolled.*

**Result:** Fleet Server setup was completed successfully.

---

### Step 31: Add the Windows 11 VM

The Windows 11 virtual machine was added to Fleet so that Elastic can collect and monitor **Windows Security, System, and Application logs**.

In Kibana, navigate to:

**☰ Menu → Fleet → Agents → Add agent**

Select:

**Enroll in Fleet**

Under **Agent policy**, select an existing endpoint policy. If a suitable policy does not exist, create a new policy with the following details:

* **Agent policy name:** Windows-SOC-Lab
* **Description:** Windows 11 endpoint monitoring for SOC lab

Then click **Create agent policy**.

![Screenshot 33: Kibana Fleet Add agent page showing the Windows-SOC-Lab agent policy](images/33-windows-soc-lab-agent-policy.png)

*Figure 33: Kibana Fleet Add agent page showing the Windows-SOC-Lab agent policy being created.*
