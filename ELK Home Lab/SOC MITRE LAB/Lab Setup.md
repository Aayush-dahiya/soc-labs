# Lab Setup

Let us start with the lab setup. 

First we need to download the required iso and vm boxes for the specific vms.

windows 10(for victim machine) - [https://www.microsoft.com/en-in/software-download/windows10ISO](https://www.microsoft.com/en-in/software-download/windows10ISO)

ubuntu desktop(for victim machine) - [https://ubuntu.com/download/desktop](https://ubuntu.com/download/desktop)

Kali linux vm box (attacker machine)- [https://www.kali.org/get-kali/#kali-virtual-machines](https://www.kali.org/get-kali/#kali-virtual-machines)

Ubuntu server(for hosting siem solutions) - [https://ubuntu.com/download/server](https://ubuntu.com/download/server)

Since i have gotten all the iso downloaded, lets start with the setup.(do standard installations for all of them, just install openssh in ubuntu server.)

- Windows 10 - done
- ubuntu server - done
- ubuntu victim - done
- kali vm - done

## Setting up the network

# 🛡 SOC Lab Network Documentation

## 1. Lab Topology

- **Host Machine (your PC)**
    - Runs VirtualBox.
    - Accesses ELK via browser + SSH.
- **ELK Server (Ubuntu Server)**
    - IP: `192.168.56.10`
    - Services: Elasticsearch (`9200`), Kibana (`5601`)
    - Role: Central log collection & dashboards.
- **Windows MITRE Attack VM**
    - IP: `192.168.56.20`
    - Role: Simulated adversary activity.
    - Agent: Winlogbeat → sends Windows event logs to ELK.
- **Ubuntu Workstation VM**
    - IP: `192.168.56.30`
    - Role: Simulated endpoint for user activity.
    - Agent: Filebeat → sends syslog/auth logs to ELK.
- **Kali VM (Attacker)**
    - IP: `192.168.56.40`
    - Role: Red Team / offensive simulation.
    - Can be used to generate malicious traffic against Ubuntu/Windows.

---

## 2. VirtualBox Network Configuration

Each VM has **two adapters**:

1. **Adapter 1 (NAT)**
    - Provides internet connectivity (updates, downloads, tools).
2. **Adapter 2 (Host-Only)**
    - Network: `192.168.56.0/24`
    - Used for SOC lab internal communication.
    - Static IPs manually assigned:
        - ELK: `192.168.56.10`
        - Windows MITRE: `192.168.56.20`
        - Ubuntu Workstation: `192.168.56.30`
        - Kali: `192.168.56.40`

### Network Diagram:

```sql
                ┌─────────────────────────┐
                │         Host PC          │
                │  Browser / SSH client    │
                └──────────┬──────────────┘
                           │ Host-Only
                     192.168.56.0/24
                           │
 ┌──────────────┬──────────┼───────────┬─────────────┐
 │              │          │           │             │
 │              │          │           │             │
▼              ▼          ▼           ▼             ▼
ELK Server   Windows     Ubuntu      Kali        (Future VM…)
192.168.56.10 192.168.56.20 192.168.56.30 192.168.56.40

```

---

## 3. Example Netplan Config (Ubuntu Machines)

File: `/etc/netplan/50-cloud-init.yaml`

```yaml
network:
  version: 2
  ethernets:
    enp0s8:     # Host-Only Adapter
      dhcp4: no
      addresses:
        - 192.168.56.10/24   # change per VM
     

```

Apply changes:

```bash
sudo netplan apply
```

## Setting Up all of the ELK Server and make Agents connect it with it

Now time to install various tools required for the simulation.

let us start with installation of elk stack on the ubuntu server.

## 2. Elasticsearch Setup

Install Java environment packages by using the below command

```bash
sudo su
apt install default-jdk default-jre -y
```

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*NiE738O2F1O91IDw_njhlA.png)

Add the elasticsearch APT repository key by using the below command

```bash
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
```

Note: the command apt-key add - adds the key in 

```json
/etc/apt/trusted.gpg
```

which has been deprecated since 2021, instead use the following:

```bash
sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/elastic.gpg
```

this will take the output recieved form the url and then convert it into binary(—dearmor) and then output the binary into the given filename. The name of the key can be anything.

 

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*ayHBi9UJQ5Emw4dFXtyDtQ.png)

Add the elastic to the APT source list by using the below command

```bash
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" > /etc/apt/sources.list.d/elastic-7.x.list
```

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*uVVPnTNiONam4WgmjujF4A.png)

Update the APT source list by using the below command

```bash
apt update
```

Install the Elastic Search by using the below command

```bash
apt install elasticsearch -y
```

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*wFFQ1s_QhVa2T8MCuIKF_w.png)

Configure the elasticsearch by using the below command

```bash
nano /etc/elasticsearch/elasticsearch.yml
```

- **Service Control:**
    
    ```bash
    sudo systemctl enable elasticsearch
    sudo systemctl start elasticsearch
    sudo systemctl status elasticsearch
    ```
    
- **Default API endpoint:**
    - `https://localhost:9200` (secured with self-signed TLS)
    - Accessible remotely at: `https://192.168.1.9:9200`
- **Security:**
    - Authentication enabled by default.
    - Superuser account:
        - **Username:** `elastic`
        - **Password:** (generated/reset with)
            
            ```bash
            sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
            
            ```
            
- **Certificates:**
    - Default CA cert generated at: `/etc/elasticsearch/certs/http_ca.crt`

[Elasticsearch Config](Lab%20Setup%20255f484cbe9d802d970cfefe8d07f93d/Elasticsearch%20Config%20258f484cbe9d80879e75f5dbb6cc64c0.md)

---

## 3. Kibana Setup

- **Service Control:**
    
    ```bash
    sudo systemctl enable kibana
    sudo systemctl start kibana
    sudo systemctl status kibana
    
    ```
    
- **Default UI endpoint:**
    - `http://localhost:5601`
    - Accessible remotely at: `http://192.168.1.9:5601`
- **Kibana Configuration File:** `/etc/kibana/kibana.yml`
    
    Current essential settings:
    
    ```yaml
    server.host: "0.0.0.0"
    
    elasticsearch.hosts: ["https://localhost:9200"]
    elasticsearch.username: "kibana_system"
    elasticsearch.password: "<kibana_system_password>"
    
    # Using insecure mode for now (lab use)
    elasticsearch.ssl.verificationMode: none
    # (Optional secure method instead of above)
    # elasticsearch.ssl.certificateAuthorities: [ "/etc/elasticsearch/certs/http_ca.crt" ]
    
    ```
    

[Kibana.yml](Lab%20Setup%20255f484cbe9d802d970cfefe8d07f93d/Kibana%20yml%20258f484cbe9d80688c61d4abd35dd082.md)

- **Credentials for Kibana Web UI Login:**
    - Username: `elastic`
    - Password: (same as Elasticsearch `elastic` account above)

---

## 4. Firewall / Network

- Kibana bound to all interfaces (`0.0.0.0:5601`).
- Elasticsearch bound to `0.0.0.0:9200`.
- Ensure firewall allows external access:
    
    ```bash
    sudo ufw allow 9200/tcp
    sudo ufw allow 5601/tcp
    sudo ufw reload
    ```
    

## ✅ Next Steps After ELK Installation

### 1. Add Data Sources (Log Ingestion)

- **Install Beats (agents) on your Ubuntu server:**
    - **Filebeat** → ships system logs (`/var/log/syslog`, `/var/log/auth.log`).
    - **Metricbeat** → ships system metrics (CPU, RAM, processes).
    - **Packetbeat** → captures network traffic (DNS, HTTP, TLS, etc.).
    
    Example (Filebeat install):
    
    ```bash
    sudo apt install filebeat
    sudo systemctl enable filebeat
    sudo systemctl start filebeat
    ```
    
    Then enable system module:
    
    ```bash
    sudo filebeat modules enable system
    sudo filebeat setup
    ```
    
    🔎 After this, you’ll see **System Logs dashboards** inside Kibana.
    
    but i receieved this error 
    
    ```
    Exiting: couldn't connect to any of the configured Elasticsearch hosts. Errors: [error connecting to Elasticsearch at http://localhost:9200: Get "http://localhost:9200": EOF]
    
    ```
    
    I will try to resolve this problem by hardcoding the machine ip in the filebeat.yml loacted at 
    
    ```bash
    /etc/filebeat/filebeat.yml
    ```
    
    this did not work even though the port 9200 is accessible on the host.
    
    Now i made changes to filebeat.yml. I added the username elastic and password for the user elastic and now the error has changed
    
    ```
    Exiting: couldn't connect to any of the configured Elasticsearch hosts. Errors: [error connecting to Elasticsearch at https://localhost:9200: Get "https://localhost:9200": x509: certificate signed by unknown authority]
    ```
    
    Meaning there are issues with the certificate.
    
    ```yaml
    
      ssl.certificate_authorities: ["/etc/elasticsearch/certs/http_ca.crt"]
    
    ```
    
    Once i made this change and entered the correct password, the below command started:
    
    ```bash
    elk@elk:~$ sudo filebeat setup
    Overwriting lifecycle policy is disabled. Set `setup.ilm.overwrite: true` to overwrite.
    Index setup finished.
    Loading dashboards (Kibana must be running and reachable)
    Loaded dashboards
    Loaded Ingest pipelines
    ```
    
    Since we have now configured the filebeat to connect to the kibana, now we need to enable the options that will then tell filebeat to send auth and syslog logs to kibana.
    
    Run the command:
    
    ```bash
    sudo nano /etc/filebeat/modules.d/system.yml 
    ```
    
    now make sure it has auth and syslog set to true, like:
    
    ```yaml
    # Module: system
    # Docs: https://www.elastic.co/guide/en/beats/filebeat/9.1/filebeat-module-system.html
    
    - module: system
      # Syslog
      syslog:
        enabled: true
    
        # Set custom paths for the log files. If left empty,
        # Filebeat will choose the paths depending on your OS.
        #var.paths:
    
        # Use journald to collect system logs
        #var.use_journald: false
    
      # Authorization logs
      auth:
        enabled: true
    
        # Set custom paths for the log files. If left empty,
        # Filebeat will choose the paths depending on your OS.
        #var.paths:
    
        # Use journald to collect auth logs
        #var.use_journald: false
    
    ```
    
    and now we are getting logs
    
    ![image.png](Lab%20Setup%20255f484cbe9d802d970cfefe8d07f93d/image.png)
    

---

# Setting up windows endpoint to send logs to elk.

# 🔹 Step 1: Download Winlogbeat

1. Go to Elastic’s official downloads:
    
    [https://www.elastic.co/downloads/beats/winlogbeat](https://www.elastic.co/downloads/beats/winlogbeat)
    
2. Download the **ZIP package** (not MSI).
3. Extract it to:
    
    ```
    C:\Program Files\Winlogbeat
    ```
    

# 🔹 Step 2: Configure `winlogbeat.yml`

Open:

```
C:\Program Files\Winlogbeat\winlogbeat.yml

```

Modify these sections:

### ✅ 1. Elasticsearch output

sending logs **directly to Elasticsearch**:

```yaml
output.elasticsearch:
  hosts: ["https://<your-elk-server-ip>:9200"]
  username: "elastic"
  password: "your_password"
  ssl.verification_mode: "none"   # (disable SSL check for now, later replace with proper certs)
```

---

### ✅ 2. Kibana setup (dashboards)

```yaml
setup.kibana:
  host: "http://<your-elk-server-ip>:5601"
  username: "elastic"
  password: "your_password"
```

For this setup i want to use api key. For this we will run the following command:

```bash
curl -u elastic:4Xd1w-l7-vcpXe4G_erS -k -X POST "https://localhost:9200/_security/api_key" -H 'Content-Type: application/json' -d '{
  "name": "winlogbeat-key",
  "role_descriptors": {
    "winlogbeat_writer": {
      "cluster": ["monitor", "manage_index_templates", "manage_ilm"],
      "index": [
        {
          "names": ["winlogbeat-*"],
          "privileges": ["write", "create_index"]
        }
      ]
    }
  }
}'

```

you will get the following output:

```json
{"id":"-93b0JgBenGWUr7P4I-V","name":"winlogbeat-key","api_key":"7rTc8diCuJAT7JgVCWjw9g","encoded":"LTkzYjBKZ0JlbkdXVXI3UDRJLVY6N3JUYzhkaUN1SkFUN0pnVkNXanc5Zw=="}
```

# 🔹 Step 3: Install Winlogbeat as a Service

Open **PowerShell as Administrator** and run:

```powershell
cd "C:\Program Files\Winlogbeat"
.\install-service-winlogbeat.ps1

```

---

# 🔹 Step 4: Test the Config

Still in PowerShell:

```powershell
.\winlogbeat.exe test config -c .\winlogbeat.yml -e

```

If all good → start the service:

```powershell
Start-Service winlogbeat
```

---

---

### ✅ 3. Event logs to collect

By default, Winlogbeat collects Application, Security, and System logs:

```yaml
winlogbeat.event_logs:
  - name: Security
  - name: System
  - name: Application

```

Later, you can add:

```yaml
  - name: Microsoft-Windows-Sysmon/Operational
```

(if you install Sysmon — highly recommended for SOC labs).

---

# 🔹 Step 3: Install Winlogbeat as a Service

Open **PowerShell as Administrator** and run:

```powershell
cd "C:\Program Files\Winlogbeat"
.\install-service-winlogbeat.ps1

```

---

## Setting up filebeat in ubuntu desktop:

# 🔹 1. Install Filebeat on the *other* Ubuntu machine

On the Ubuntu client you want to monitor:

```bash
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg

echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list

sudo apt update
sudo apt install filebeat -y

```

---

# 🔹 2. Configure Filebeat to send to ELK server

Edit `/etc/filebeat/filebeat.yml` on the **client machine**:

```yaml
# ============================== Filebeat inputs ===============================
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/*.log
    - /var/log/syslog
    - /var/log/auth.log

# ============================== Filebeat modules ==============================
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: true

# ============================== Elasticsearch output ==========================
output.elasticsearch:
  hosts: ["http://<ELK-SERVER-IP>:9200"]
  username: "elastic"        # only if security is enabled
  password: "yourpassword"

```

👉 Replace `<ELK-SERVER-IP>` with the **Host-only IP** of your ELK VM.

👉 If you’re running Kibana/ES with security disabled, you can remove `username` and `password`.

---

# 🔹 3. Enable the System Module (for system logs)

```bash
sudo filebeat modules enable system

```

---

# 🔹 4. Load Index Templates & Dashboards

Run once on the **client machine**:

```bash
sudo filebeat setup \
  -E output.elasticsearch.hosts=["http://<ELK-SERVER-IP>:9200"] \
  -E setup.kibana.host="http://<ELK-SERVER-IP>:5601"

```

This loads dashboards, index templates, and ingest pipelines into Elasticsearch + Kibana.

---

# 🔹 5. Start Filebeat

```bash
sudo systemctl enable filebeat
sudo systemctl start filebeat

```

---

# 🔹 6. Verify

On ELK server:

```bash
curl -X GET "http://localhost:9200/filebeat-*/_count?pretty"

```

You should see documents flowing in 🎉

In Kibana:

- Go to **Discover**
- Create a Data View: `filebeat-*`
- Pick `@timestamp`
- You’ll now see your Ubuntu machine’s logs (auth.log, syslog, etc).

---

⚡ Question: do you want these Ubuntu machines just for **system logs** (syslog/auth.log), or do you also want things like **process, network activity, security events**?

👉 If it’s for SOC-style detection, I’d recommend using **Auditbeat** alongside Filebeat — it gives you much deeper visibility.

Would you like me to show you how to set up **Auditbeat** too for that extra visibility?

### You said:

it is throwing x509 error

### ChatGPT said:

Got it — the **`x509: certificate signed by unknown authority`** error happens because your **Elasticsearch is running with TLS enabled**, and your Filebeat client doesn’t trust the certificate.

This is normal if:

- You enabled **xpack security** (Elastic defaults to HTTPS with self-signed certs), or
- You’re connecting with `https://` instead of `http://`.

---

# 🔹 Fix Options

### **Option 1: If you don’t need TLS yet (lab setup)**

If your Elasticsearch is running on plain HTTP (`http://<ELK-IP>:9200`) but Filebeat is still trying `https://`, edit `/etc/filebeat/filebeat.yml` on the Ubuntu client:

```yaml
output.elasticsearch:
  hosts: ["http://<ELK-SERVER-IP>:9200"]

```

👉 Notice it’s **http://** not https://.

# 🔹 1. Install Filebeat on the *other* Ubuntu machine

On the Ubuntu client you want to monitor:

```bash
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg

echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list

sudo apt update
sudo apt install filebeat -y

```

---

# 🔹 2. Configure Filebeat to send to ELK server

Edit `/etc/filebeat/filebeat.yml` on the **client machine**:

```yaml
# ============================== Filebeat inputs ===============================
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/*.log
    - /var/log/syslog
    - /var/log/auth.log

# ============================== Filebeat modules ==============================
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: true

# ============================== Elasticsearch output ==========================
output.elasticsearch:
  hosts: ["http://<ELK-SERVER-IP>:9200"]
  username: "elastic"        # only if security is enabled
  password: "yourpassword"

```

👉 Replace `<ELK-SERVER-IP>` with the **Host-only IP** of your ELK VM.

👉 If you’re running Kibana/ES with security disabled, you can remove `username` and `password`.

---

# 🔹 3. Enable the System Module (for system logs)

```bash
sudo filebeat modules enable system

```

---

# 🔹 4. Load Index Templates & Dashboards

Run once on the **client machine**:

```bash
sudo filebeat setup \
  -E output.elasticsearch.hosts=["http://<ELK-SERVER-IP>:9200"] \
  -E setup.kibana.host="http://<ELK-SERVER-IP>:5601"

```

This loads dashboards, index templates, and ingest pipelines into Elasticsearch + Kibana.

---

# 🔹 5. Start Filebeat

```bash
sudo systemctl enable filebeat
sudo systemctl start filebeat

```

---

# 🔹 6. Verify

On ELK server:

```bash
curl -X GET "http://localhost:9200/filebeat-*/_count?pretty"

```

You should see documents flowing in 🎉

In Kibana:

- Go to **Discover**
- Create a Data View: `filebeat-*`
- Pick `@timestamp`
- You’ll now see your Ubuntu machine’s logs (auth.log, syslog, etc).

---

⚡ Question: do you want these Ubuntu machines just for **system logs** (syslog/auth.log), or do you also want things like **process, network activity, security events**?

👉 If it’s for SOC-style detection, I’d recommend using **Auditbeat** alongside Filebeat — it gives you much deeper visibility.

Would you like me to show you how to set up **Auditbeat** too for that extra visibility?

### You said:

it is throwing x509 error

### ChatGPT said:

Got it — the **`x509: certificate signed by unknown authority`** error happens because your **Elasticsearch is running with TLS enabled**, and your Filebeat client doesn’t trust the certificate.

This is normal if:

- You enabled **xpack security** (Elastic defaults to HTTPS with self-signed certs), or
- You’re connecting with `https://` instead of `http://`.

---

# 🔹 Fix Options

### **Option 1: If you don’t need TLS yet (lab setup)**

If your Elasticsearch is running on plain HTTP (`http://<ELK-IP>:9200`) but Filebeat is still trying `https://`, edit `/etc/filebeat/filebeat.yml` on the Ubuntu client:

```yaml
output.elasticsearch:
  hosts: ["http://<ELK-SERVER-IP>:9200"]

```

👉 Notice it’s **http://** not https://.

## 🔹 Next Steps

### 1. Enable Filebeat Modules

Choose what you want to collect (syslog, auth, Apache, MySQL, etc.). For system logs:

```bash
sudo filebeat modules enable system

```

Verify what's enabled:

```bash
sudo filebeat modules list

```

---

### 2. Configure the Module (Optional)

Edit the configuration if needed:

```bash
sudo nano /etc/filebeat/modules.d/system.yml

```

By default, it collects:

- `/var/log/auth.log`
- `/var/log/syslog`

---

### 3. Test Configuration

```bash
sudo filebeat test config
sudo filebeat test output

```

✅ You should see "Config OK" and a successful connection to Elasticsearch.

---

### 4. Start Filebeat Service

```bash
sudo systemctl enable filebeat
sudo systemctl start filebeat

```

---

### 5. Verify Data in Elasticsearch

Check if the index is being created:

```bash
curl -u elastic:yourpassword -k "https://192.168.56.10:9200/_cat/indices?v"

```

Look for something like:

```
green open filebeat-7.17.13-2025.08.23-000001 ...

```

---

### 6. Verify in Kibana

1. Open Kibana → `http://192.168.56.10:5601`
2. Go to **Discover** → Choose the `filebeat-*` index pattern
3. You should now see logs from your Ubuntu machine 🎉

---

⚡ To send logs from multiple Ubuntu hosts (like your other VM), simply install Filebeat on each, copy over the `http_ca.crt`, and repeat the setup. All hosts will feed into the same ELK stack.

---

### 2. Configure Ingest Pipelines & Index Patterns

- Go to **Kibana → Stack Management → Index Patterns**.
- Add patterns:
    - `filebeat-*`
    - `metricbeat-*`
    - `packetbeat-*`
    - `winlogbeat-*`

This lets you query logs via **Discover**.

---

### 3. Enable Elastic Security App

- In Kibana, navigate to **Security → Overview**.
- This activates the **SIEM app**.
- You’ll see:
    - Alerts & Detections
    - Host & Network views
    - Timeline for investigations

---

### 4. Simulate Security Events (for SOC practice)

- Try brute-force SSH attempts (they will show up in auth logs).
- Run simple `nmap` scans against your lab box (network logs will show in Packetbeat).
- You’ll see these events inside Kibana → Security.

---

### 5. Create Detection Rules

- In **Kibana Security → Rules**, enable some **prebuilt rules** (Elastic provides hundreds).
- Example:
    - Detect multiple failed SSH logins
    - Detect privilege escalation attempts
    - Detect port scanning

```markdown
						 ┌─────────────────────────┐
            │         Host PC          │
            │  Browser / SSH client    │
            └──────────┬──────────────┘
                       │ Host-Only
                 192.168.60.0/24
                       │
┌──────────────┬──────────┼───────────┬─────────────┐
│              │          │           │             │
│              │          │           │             │
▼              ▼          ▼           ▼             ▼
Splunk Server Windows   Ubuntu       Kali      (Future VM…)
192.168.60.3 192.168.60.6 192.168.60.7 192.168.60.30
```
