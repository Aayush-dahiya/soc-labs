# Splunk Home Lab

This repository documents my personal Splunk home lab for SOC analyst practice.  
The lab simulates a small enterprise network where multiple hosts send logs into a central Splunk server for monitoring, analysis, and security use cases.

---

## 🏗️ Lab Architecture

![](https://github.com/Aayush-dahiya/soc-labs/blob/main/Splunk%20Homelab/images/Splunk%20data%20Flow.gif)


*Figure 1 — SOC architecture showing Splunk server, Windows, Ubuntu, and Kali VMs in host-only subnet.*


```

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



- **Splunk Server**: Central log collector and search head.  
- **Windows VM**: For Windows Event Log forwarding.  
- **Ubuntu VM**: For Linux syslog and application logs.  
- **Kali VM**: For generating attack traffic and monitoring detections.  
- **Future VM(s)**: Additional simulated hosts or servers.

---

## ✅ Current Status

- Lab infrastructure created (servers + clients + networking).  
- Splunk Enterprise installed on **192.168.60.3**.  
- Connectivity verified across all VMs on the `192.168.60.0/24` subnet.  

---

## 🚧 Next Steps (Data Ingestion)

1. **Install Splunk Universal Forwarder** on:
   - Windows (forward Windows Event Logs).
   - Ubuntu (forward `/var/log/syslog`, `/var/log/auth.log`).
   - Kali (forward system + security tool logs).

2. **Configure forwarder outputs**  
   Update `outputs.conf` on each forwarder:
   ```conf
   [tcpout]
   defaultGroup = default-autolb-group

   [tcpout:default-autolb-group]
   server = 192.168.60.3:9997
