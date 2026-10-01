# asterisk-splunk-voip-security1
# 🔐 VoIP Security Monitoring with Asterisk & Splunk

## 📌 Overview

This project focuses on the **security monitoring and detection of suspicious activity in a VoIP infrastructure** using **Asterisk** and **Splunk Enterprise**.

The objective was to build a small VoIP environment, centralize Asterisk logs in Splunk, detect repeated SIP authentication failures, investigate suspicious activity, and simulate an incident response workflow.

---

## 🏗️ Architecture

```text
                         Windows Host
                    ┌─────────────────────┐
                    │                     │
              MicroSIP 1001        Zoiper 1002
                    │                     │
                    └──────────┬──────────┘
                               │ SIP
                               ▼
                    ┌─────────────────────┐
                    │     VM1 - Ubuntu    │
                    │                     │
                    │      Asterisk       │
                    │       PJSIP         │
                    │                     │
                    │   192.168.1.7       │
                    └──────────┬──────────┘
                               │
                         Asterisk Logs
                               │
                               ▼
                    ┌─────────────────────┐
                    │     VM2 - Ubuntu    │
                    │                     │
                    │   Splunk Enterprise  │
                    │   SIEM / Monitoring  │
                    │                     │
                    │   192.168.1.17      │
                    └─────────────────────┘
```

---

## 🛠️ Technologies

* **Asterisk 22**
* **PJSIP**
* **Splunk Enterprise**
* **Splunk Universal Forwarder**
* **SIP / VoIP**
* **Ubuntu Linux**
* **MicroSIP**
* **Zoiper**
* **UFW / Firewall**

---

## ⚙️ Project Setup

### VoIP Infrastructure

Asterisk was configured as the VoIP server with two SIP extensions:

* `1001`
* `1002`

The extensions were tested using:

* MicroSIP
* Zoiper

Both extensions were able to register with Asterisk and communicate with each other.

---

## 📊 Log Collection

Asterisk logs were collected from:

```text
/var/log/asterisk/messages.log
```

The **Splunk Universal Forwarder** was installed on the Asterisk server and configured to forward the logs to the Splunk server.

```text
Asterisk
   │
   │ messages.log
   ▼
Splunk Universal Forwarder
   │
   │ TCP 9997
   ▼
Splunk Enterprise
```

The logs were indexed in Splunk using the index:

```text
asterisk
```

---

## 🚨 Security Detection

The project focuses mainly on detecting **failed SIP authentication attempts**.

Example event:

```text
Request 'REGISTER' ... failed to authenticate
```

Splunk was used to identify:

* Source IP address
* Targeted SIP extension
* Number of failed authentication attempts
* Repeated authentication failures

### Example SPL Query

```spl
index=asterisk "Failed to authenticate"
| rex "failed for '(?<src_ip>[0-9.]+):"
| stats count by src_ip
| sort - count
```

This query helps identify IP addresses generating authentication failures.

---

## 🔎 Investigation

After detecting repeated authentication failures, the source IP and targeted extension were extracted from the logs.

Example:

```spl
index=asterisk "Failed to authenticate"
| rex "Request 'REGISTER' from '<sip:(?<extension>[0-9]+)@"
| rex "failed for '(?<src_ip>[0-9.]+):"
| stats count as failed_attempts by src_ip, extension
| sort - failed_attempts
```

This provides an overview of:

| Source IP | Extension | Failed Attempts   |
| --------- | --------- | ----------------- |
| Source IP | 1001      | Multiple attempts |

This type of activity can indicate a possible **SIP brute-force authentication pattern**.

---

## ⏱️ Threshold Detection

A threshold-based detection was created to identify repeated failed authentication attempts within a short period.

```spl
index=asterisk "Failed to authenticate"
| rex "failed for '(?<src_ip>[0-9.]+):"
| bin _time span=5m
| stats count by _time, src_ip
| where count >= 2
```

The detection looks for multiple failed authentication attempts from the same source within a 5-minute window.

---

## 🔔 Splunk Alert

A Splunk alert was configured based on the threshold detection.

The alert is designed to identify repeated SIP authentication failures and generate a detection when the defined threshold is reached.

---

## 🛡️ Incident Response Workflow

The project also includes a basic incident response workflow:

```text
Detection
    ↓
Investigation
    ↓
Containment
    ↓
Verification
```

### 1. Detection

Identify repeated failed SIP authentication attempts through Splunk.

### 2. Investigation

Analyze:

* Source IP
* Targeted extension
* Number of attempts
* Authentication failure events

### 3. Containment

A firewall rule was simulated against a **test IP address** to demonstrate the containment phase without affecting the real VoIP environment.

### 4. Verification

Verify that the detection and containment workflow can be performed without disrupting legitimate VoIP communications.

---



### VoIP Environment

MicroSIP and Zoiper connected to the Asterisk server.


### Splunk Detection

Splunk dashboard/search showing failed SIP authentication activity.


## 🎯 Learning Objectives

Through this project, I practiced:

* VoIP infrastructure configuration
* Asterisk and PJSIP
* SIP authentication
* Linux administration
* Log collection and centralization
* Splunk SIEM
* SPL queries
* Security event investigation
* Threshold-based detection
* Basic incident response
* Firewall-based containment concepts

---



<img width="632" height="297" alt="Screenshot 2026-10-01 141554" src="https://github.com/user-attachments/assets/dd7f14f4-0f5d-4a4c-bccd-11a910a7e940" />
<img width="118" height="235" alt="Screenshot 2026-10-01 141717" src="https://github.com/user-attachments/assets/94bbc9b8-550e-49cf-b73a-f02427c999ec" />
<img width="951" height="477" alt="Screenshot 2026-10-01 171845" src="https://github.com/user-attachments/assets/b3c5851b-e6aa-41ed-8a38-d8940b6bf4cf" />
<img width="413" height="347" alt="Screenshot 2026-10-01 141850" src="https://github.com/user-attachments/assets/13933ef4-45b5-4aa2-8cc7-c1f5051167b3" />
