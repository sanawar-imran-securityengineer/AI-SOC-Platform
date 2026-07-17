# 🚀 SOAR-Flow

## 📌 Project Description

This project demonstrates how to integrate **Shuffle SOAR** with **Wazuh SIEM** and **TheHive** to automate incident response.

- ✅ Receiving security alerts from Wazuh
- ✅ Enriching alerts using external threat intelligence (VirusTotal, AbuseIPDB)
- ✅ Creating an incident in TheHive for case management
- ✅ Sending notifications to a Discord channel
- ✅ (Bonus) Auto-mitigating threats (e.g., blocking malicious IPs)

By implementing this SOAR workflow, you can automate security operations, reduce response time, and improve efficiency in a SOC environment.

---

## 🔧 Tools Used

| Tool | Description |
|------|-------------|
| **Wazuh SIEM** | Security Information & Event Management (SIEM) solution for threat detection |
| **TheHive** | Open-source Security Incident Response Platform (SIRP) |
| **Shuffle** | Open-source Security Orchestration, Automation, and Response (SOAR) platform |
| **VirusTotal API** | Used for malware and URL reputation checks |
| **AbuseIPDB API** | Used for checking if an IP address is malicious |
| **Discord Webhook** | Sends alerts to a Discord channel for real-time monitoring |

---

## 🛠️ Installation & Setup

### VM-1 — Wazuh and TheHive

**Specifications**
- RAM: 12GB+
- HDD: 60GB+
- OS: Ubuntu 24.04 LTS

#### 1️⃣ Install Wazuh SIEM

Follow the official guide: [Wazuh Installation Guide](https://documentation.wazuh.com/current/installation-guide/index.html)

```bash
# Update and upgrade
apt-get update && apt-get upgrade

# Install Wazuh 4.10
curl -sO https://packages.wazuh.com/4.10/wazuh-install.sh && sudo bash ./wazuh-install.sh -a

# Extract Wazuh credentials
sudo tar -xvf wazuh-install-files.tar
```

**Wazuh Dashboard Credentials:**
- User: `admin`
- Password: `***************` *(generated during install)*

Access the dashboard at: `https://<YOUR-IP>`
<img width="814" height="363" alt="image" src="https://github.com/user-attachments/assets/34cc6803-9b26-48a6-baa1-928c78700986" />

#### 2️⃣ Install TheHive

Follow the official guide: [TheHive Installation Guide](https://docs.thehive-project.org/thehive/installation-and-configuration/)
<img width="788" height="338" alt="image" src="https://github.com/user-attachments/assets/3c194d91-441d-4fc9-8440-04d020eabf48" />

**Install dependencies:**
```bash
apt install wget gnupg apt-transport-https git ca-certificates ca-certificates-java curl software-properties-common python3-pip lsb-release
```

**Install Java:**
```bash
wget -qO- https://apt.corretto.aws/corretto.key | sudo gpg --dearmor -o /usr/share/keyrings/corretto.gpg
echo "deb [signed-by=/usr/share/keyrings/corretto.gpg] https://apt.corretto.aws stable main" | sudo tee -a /etc/apt/sources.list.d/corretto.sources.list
sudo apt update
sudo apt install java-common java-11-amazon-corretto-jdk
echo JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto" | sudo tee -a /etc/environment
export JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto"
```

**Install Cassandra:**
```bash
wget -qO - https://downloads.apache.org/cassandra/KEYS | sudo gpg --dearmor -o /usr/share/keyrings/cassandra-archive.gpg
echo "deb [signed-by=/usr/share/keyrings/cassandra-archive.gpg] https://debian.cassandra.apache.org 40x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
sudo apt update
sudo apt install cassandra
```

**Install ElasticSearch:**
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update
sudo apt install elasticsearch
```

**Install TheHive:**
```bash
wget -O- https://archives.strangebee.com/keys/strangebee.gpg | sudo gpg --dearmor -o /usr/share/keyrings/strangebee-archive-keyring.gpg
echo 'deb [signed-by=/usr/share/keyrings/strangebee-archive-keyring.gpg] https://deb.strangebee.com thehive-5.2 main' | sudo tee -a /etc/apt/sources.list.d/strangebee.list
sudo apt-get update
sudo apt-get install -y thehive
```

**Default Credentials for TheHive:**
- Port: `9000`
- Login: `admin@thehive.local`
- Password: `secret`

---

### VM-2 — Shuffle

**Specifications**
- RAM: 4GB+
- HDD: 40GB+
- OS: Ubuntu 24.04 LTS

#### 3️⃣ Install Shuffle SOAR

Follow the official guide: [Shuffle Installation Guide](https://shuffler.io/docs/installation)

```bash
# Install Docker if not already installed
sudo apt update && sudo apt install -y docker.io docker-compose

# Enable and start Docker
sudo systemctl enable docker
sudo systemctl start docker

# Clone the Shuffle repository
git clone https://github.com/Shuffle/Shuffle.git
cd Shuffle

# Build and run Shuffle with Docker Compose
sudo docker-compose up -d
```

Access the Shuffle Web UI at: `http://<YOUR-IP>:3001`

#### 4️⃣ Create a Discord Webhook

1. Go to your Discord Server → **Settings → Integrations → Webhooks**
2. Click **New Webhook** → name it `SOC Alerts`
3. Copy the Webhook URL (you'll need it later)

---

## ⚙️ Configuration for TheHive

### Configure Cassandra

Edit the config file:
```bash
nano /etc/cassandra/cassandra.yaml
```

Update the following values:
```yaml
cluster_name: 'SOAR-Flow'
listen_address: <YOUR-IP>
rpc_address: <YOUR-IP>
seeds: "<YOUR-IP>:7000"
```

Restart Cassandra:
```bash
systemctl stop cassandra.service
rm -rf /var/lib/cassandra/*
systemctl start cassandra.service
```

### Configure ElasticSearch

Edit the config file:
```bash
nano /etc/elasticsearch/elasticsearch.yml
```

Update the following values:
```yaml
cluster.name: thehive
node.name: node-1
network.host: <YOUR-IP>
http.port: 9200
discovery.seed_hosts: ["127.0.0.1"]
cluster.initial_master_nodes: ["node-1"]
```

Start ElasticSearch:
```bash
systemctl start elasticsearch
systemctl enable elasticsearch
systemctl status elasticsearch
```

### Configure TheHive

Ensure proper ownership:
```bash
ls -la /opt/thp
chown -R thehive:thehive /opt/thp
```

Edit the config file:
```bash
nano /etc/thehive/application.conf
```

Update database and index configuration:
```hocon
db.janusgraph {
  storage {
    backend = cql
    hostname = ["<YOUR-IP>"]
    cql {
      cluster-name = SOAR-Flow
      keyspace = thehive
    }
  }
}

index.search {
  backend = elasticsearch
  hostname = ["<YOUR-IP>"]
  index-name = thehive
}

application.baseUrl = "http://<YOUR-IP>:9000"
```

Start TheHive:
```bash
systemctl start thehive
systemctl enable thehive
systemctl status thehive
```

---

## 🔄 Workflow — Automating Incident Response

### 📌 Workflow Overview

This workflow automates incident response using Shuffle:

1️⃣ Receive alerts from Wazuh SIEM when suspicious activity is detected
2️⃣ Enrich the alert using VirusTotal & AbuseIPDB API
3️⃣ Create an incident in TheHive for case tracking
4️⃣ Send a notification to Discord with alert details
5️⃣ *(Optional)* Perform auto-mitigation (e.g., blocking malicious IPs)

### 📌 Shuffle Workflow Steps

**🔹 Step 1: Add Wazuh Alert as Trigger**
- In Shuffle, create a new workflow and add a **Webhook** trigger
- Configure Wazuh to send alerts via webhooks

**🔹 Step 2: Enrich Data with VirusTotal & AbuseIPDB**
- Add an **HTTP Request** node to check IPs/hashes using the VirusTotal API
- Add another **HTTP Request** node to query AbuseIPDB for malicious IPs

**🔹 Step 3: Create an Incident in TheHive**
- Use the TheHive API to create a new case with alert details

**🔹 Step 4: Send Alert to Discord**
- Use the Discord Webhook to send a formatted message to a SOC channel

**🔹 Step 5: (Optional) Auto-Mitigation**
- If the IP is high risk, trigger a firewall rule to block the attacker
<img width="801" height="383" alt="image" src="https://github.com/user-attachments/assets/b6de4ed4-1c79-4f1c-83ff-343a2260de27" />

---

## 🚀 Running the Workflow

### Step 1: Configure Wazuh to Send Alerts to Shuffle

Edit the Wazuh `ossec.conf` file to send webhook alerts:

```xml
<integration>
  <name>custom-webhook</name>
  <hook_url>http://<SHUFFLE-IP>:5001/webhook</hook_url>
  <alert_format>json</alert_format>
</integration>
```

Restart Wazuh to apply changes:
```bash
sudo systemctl restart wazuh-manager
```

### Step 2: Configure TheHive API Key

Generate an API key in TheHive and add it to Shuffle's HTTP Request node.

### Step 3: Configure Discord Webhook in Shuffle

Use the Discord Webhook URL in the Shuffle HTTP Request node. Example payload:

```json
{
  "content": "🚨 New Security Alert 🚨\n\nIP: 192.168.1.100\nSeverity: High\nSource: Wazuh SIEM"
}
```

### Step 4: Test the Workflow

1. Trigger an alert in Wazuh (e.g., failed SSH logins)
2. Verify the incident is created in TheHive
3. Check if the alert is sent to Discord

---

## 📌 Example Output

**✅ TheHive Incident Created:**
```
[INFO] New Incident Created in TheHive:
Title: Suspicious SSH Login Attempts
Severity: High
Source: Wazuh SIEM
```

**✅ Discord Alert:**
> 🚨 New Security Alert 🚨
> IP: 192.168.1.100
> Severity: High
> Source: Wazuh SIEM
<img width="409" height="105" alt="image" src="https://github.com/user-attachments/assets/98909b3c-fd76-4fd3-915b-02e20322897a" />

---

## 🎯 Future Enhancements

- 🔹 Add auto-mitigation (e.g., blocking attacker IPs via firewall rules)
- 🔹 Integrate more threat intelligence feeds (e.g., MISP, Shodan API)
- 🔹 Expand automation to handle different types of incidents

---

## 📜 License

This project is licensed under the **MIT License**.





