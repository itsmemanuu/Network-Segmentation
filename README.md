# Network-Segmentation

## Overview

This project implements a virtual enterprise network segmentation lab using VMware Workstation, pfSense, OPNsense, Ubuntu Server, nginx, Wazuh SIEM, and Zabbix.

The objective is to simulate a realistic segmented network where public-facing services are isolated in a DMZ, internal users are separated from security infrastructure, and monitoring/logging tools are protected inside a dedicated Security network.

The lab validates firewall segmentation, controlled service exposure, SIEM visibility, monitoring traffic, routing between zones, and blocked traffic evidence.

---

## Project Objectives

* Build a segmented enterprise-style network in VMware Workstation.
* Use pfSense as the DMZ/perimeter firewall.
* Use OPNsense as the enterprise/internal firewall.
* Isolate public-facing services inside a DMZ.
* Deploy an nginx web server and DNS service in the DMZ.
* Deploy Wazuh SIEM and Zabbix in the Security network.
* Restrict access to sensitive monitoring services.
* Allow only required telemetry traffic between zones.
* Validate segmentation using connectivity tests and firewall logs.
* Document evidence with screenshots and test results.

---

## Technologies Used

| Technology         | Purpose                              |
| ------------------ | ------------------------------------ |
| VMware Workstation | Virtualization platform              |
| pfSense            | DMZ/perimeter firewall               |
| OPNsense           | Enterprise/internal firewall         |
| Ubuntu Server      | Linux server operating system        |
| nginx              | DMZ web server                       |
| Wazuh SIEM         | Security monitoring and log analysis |
| Zabbix             | Infrastructure monitoring            |
| Ubuntu Client      | Internal Linux client                |

---

## Network Architecture

The lab is divided into four main security zones:

| Zone             | Purpose                                                          |
| ---------------- | ---------------------------------------------------------------- |
| External Network | Simulates an outside client or internet-side network             |
| DMZ              | Hosts public-facing services such as web and DNS                 |
| Internal Users   | Represents normal internal user workstations                     |
| Security Network | Hosts monitoring and security platforms such as Wazuh and Zabbix |
### Topology Diagram

<img width="2278" height="2404" alt="Diagrama en blanco - Página 1(2)" src="https://github.com/user-attachments/assets/03495224-b459-4693-bd9c-008b673c938c" />

---

## IP Addressing

| Device / Interface  |      IP Address | Role                              |
| ------------------- | --------------: | --------------------------------- |
| pfSense WAN         |  172.16.153.100 | External-facing interface         |
| pfSense LAN         |    192.168.10.2 | DMZ gateway                       |
| pfSense OPT1        | 192.168.239.131 | Transit interface toward OPNsense |
| OPNsense WAN        | 192.168.239.130 | Transit interface toward pfSense  |
| OPNsense LAN        |    192.168.40.2 | Internal Users gateway            |
| OPNsense OPT1       |  192.168.30.129 | Security network gateway          |
| Wazuh SIEM / Zabbix |  192.168.30.128 | Security monitoring server        |
| Web/DNS Server      |   192.168.10.10 | DMZ server running nginx and DNS  |

---

## Security Design

The network follows a defense-in-depth approach by separating systems based on exposure and sensitivity.

The DMZ contains public-facing services, including the nginx web server and DNS service. These services are reachable from the external network only through specific NAT and firewall rules.

The Internal Users network is separated from both the DMZ and the Security network. Normal users should not have unrestricted access to security monitoring platforms.

The Security network hosts Wazuh SIEM and Zabbix. These platforms collect logs, security telemetry, and monitoring data, but they are not exposed directly to the external network or general users.

---

## Firewall Segmentation

The lab uses two firewall layers:

| Firewall | Role                                                                              |
| -------- | --------------------------------------------------------------------------------- |
| pfSense  | Controls access between the external network, DMZ, and transit network            |
| OPNsense | Controls access between the transit network, Internal Users, and Security network |

The firewall policy is based on explicit allow rules followed by default blocking. Only required services are permitted.

---

## pfSense Firewall Rules

pfSense acts as the DMZ/perimeter firewall. It exposes only selected DMZ services and forwards allowed traffic to the Web/DNS server.

### WAN Port Forwarding

| Interface | Protocol | Source | Destination | NAT Destination | Port | Purpose                                       |
| --------- | -------- | ------ | ----------- | --------------- | ---: | --------------------------------------------- |
| WAN       | TCP      | Any    | WAN address | 192.168.10.10   |  443 | Publish HTTPS service from the DMZ web server |
| WAN       | UDP      | Any    | WAN address | 192.168.10.10   |   53 | Publish DNS over TCP from the DMZ DNS server  |

These NAT rules publish selected services from the DMZ server without exposing the entire host. External clients can reach only the ports explicitly forwarded by pfSense.


### DMZ Monitoring Rules

| Protocol | Source        | Destination    |      Port | Action | Purpose                                         |
| -------- | ------------- | -------------- | --------: | ------ | ----------------------------------------------- |
| TCP      | 192.168.10.10 | 192.168.30.128 | 1514-1515 | Allow  | Allows Wazuh agent communication and enrollment |
| TCP      | 192.168.10.10 | 192.168.30.128 |     10051 | Allow  | Allows Zabbix active monitoring traffic         |

These rules allow the DMZ Web/DNS server to send security and monitoring telemetry to the Security network.

The DMZ server is not granted broad access to internal networks. It can only communicate with the Wazuh/Zabbix server on the required monitoring ports.

### pfSense Static Route

| Destination Network | Next Hop              | Purpose                                                                  |
| ------------------- | --------------------- | ------------------------------------------------------------------------ |
| 192.168.30.0/24     | OPNsense transit side | Allows pfSense/DMZ traffic to reach the Security network behind OPNsense |

This route tells pfSense how to reach the Security network. Without this route, pfSense would not know that `192.168.30.0/24` is located behind OPNsense.

---

## OPNsense Firewall Rules

OPNsense acts as the enterprise/internal firewall. It enforces segmentation between the DMZ path, Internal Users network, and Security network.

### DMZ Server to Wazuh

| Rule | Protocol | Source        | Destination    |      Port | Action | Purpose                                         |
| ---- | -------- | ------------- | -------------- | --------: | ------ | ----------------------------------------------- |
| 100  | TCP      | 192.168.10.10 | 192.168.30.128 | 1514-1515 | Allow  | Allows Wazuh agent communication and enrollment |

This rule allows the DMZ server to connect to Wazuh in the Security network. It is required for the Wazuh agent to register and send security events to the Wazuh manager.

### DMZ Server to Zabbix

| Rule | Protocol | Source        | Destination    |  Port | Action | Purpose                          |
| ---- | -------- | ------------- | -------------- | ----: | ------ | -------------------------------- |
| 200  | TCP      | 192.168.10.10 | 192.168.30.128 | 10051 | Allow  | Allows Zabbix monitoring traffic |

This rule allows the DMZ server to send monitoring data to Zabbix. The access is limited to the monitoring server and the required Zabbix port.

### Default Block Rule

| Rule | Protocol | Source | Destination | Port | Action | Purpose                                           |
| ---- | -------- | ------ | ----------- | ---: | ------ | ------------------------------------------------- |
| 300  | Any      | Any    | Any         |  Any | Block  | Blocks all traffic that is not explicitly allowed |

The final rule blocks all remaining traffic. This creates a default-deny security model where only traffic matching the previous allow rules is permitted.

This is important because the Enterprise Firewall should not allow unrestricted communication between the DMZ, Internal Users, and Security networks. Access is granted only for required services such as Wazuh agent communication and Zabbix monitoring.

---

## Routing

The lab uses routing between firewalls to allow controlled communication across segmented networks.

| Route                               | Purpose                                                                 |
| ----------------------------------- | ----------------------------------------------------------------------- |
| pfSense route to `192.168.30.0/24`  | Allows DMZ traffic to reach the Security network through OPNsense       |
| OPNsense route to `192.168.10.0/24` | Allows return traffic from the Security network to reach the DMZ server |

The transit network `192.168.239.0/24` connects pfSense and OPNsense. This network is used only for firewall-to-firewall routing and should not host regular client or server systems.

---

## Services Deployed

### DMZ Web/DNS Server

| Service          | Host           |    IP Address |
| ---------------- | -------------- | ------------: |
| nginx Web Server | Web/DNS Server | 192.168.10.10 |
| DNS Server       | Web/DNS Server | 192.168.10.10 |

The Web/DNS server is placed in the DMZ because it represents a public-facing service. External access is controlled through pfSense NAT and firewall rules.

### Wazuh SIEM

| Service    | Host            |     IP Address |
| ---------- | --------------- | -------------: |
| Wazuh SIEM | Security Server | 192.168.30.128 |

Wazuh centralizes security monitoring and log analysis. The DMZ server can send security events to Wazuh, but Wazuh is not exposed directly to the external network.

### Zabbix

| Service | Host            |     IP Address |
| ------- | --------------- | -------------: |
| Zabbix  | Security Server | 192.168.30.128 |

Zabbix monitors infrastructure availability and service health. The DMZ server can communicate with Zabbix only through the required monitoring port.

---

## Validation Tests

| Test                                                 | Source                  | Destination                     | Expected Result | Status  | Evidence                  |
| ---------------------------------------------------- | ----------------------- | ------------------------------- | --------------- | ------- | ------------------------- |
| External client reaches DMZ web server               | External Client         | 192.168.10.10 TCP/443           | Allowed         | Pending | Screenshot                |
| External client reaches DMZ DNS server               | External Client         | 192.168.10.10 TCP/53 and UDP/53 | Allowed         | Pending | Screenshot                |
| External client cannot reach internal network        | External Client         | Internal Users network          | Blocked         | Pending | Screenshot / firewall log |
| Internal admin accesses Wazuh dashboard              | Internal Admin          | 192.168.30.128 HTTPS            | Allowed         | Pending | Screenshot                |
| Normal internal client cannot access Wazuh dashboard | Ubuntu Client           | 192.168.30.128 HTTPS            | Blocked         | Pending | Screenshot / firewall log |
| DMZ server sends logs to Wazuh                       | 192.168.10.10           | 192.168.30.128 TCP/1514-1515    | Allowed         | Pending | Wazuh agent screenshot    |
| DMZ server sends monitoring data to Zabbix           | 192.168.10.10           | 192.168.30.128 TCP/10051        | Allowed         | Pending | Zabbix screenshot         |
| Unauthorized traffic is blocked                      | Any unauthorized source | Protected networks              | Blocked         | Pending | Firewall log screenshot   |

---

## Evidence

### pfSense Interfaces

<img width="567" height="91" alt="image" src="https://github.com/user-attachments/assets/d8e54140-baa2-4c47-9d5c-652f19d25f9d" />


### pfSense NAT Rules

<img width="1253" height="155" alt="image" src="https://github.com/user-attachments/assets/43650d74-a0a4-412a-85fa-3f4ff22f1a32" />


### pfSense Firewall Rules
<img width="1255" height="199" alt="image" src="https://github.com/user-attachments/assets/314d16de-c51f-4969-95ab-ef2d98a9a2d3" />
<img width="1255" height="217" alt="image" src="https://github.com/user-attachments/assets/4997cc99-a88d-443d-b181-54e6984c32fb" />


### pfSense Static Routes
<img width="1251" height="114" alt="image" src="https://github.com/user-attachments/assets/030d427e-ca14-4d85-ab1e-c2a90e42b468" />


### OPNsense Interfaces
<img width="548" height="93" alt="image" src="https://github.com/user-attachments/assets/f0a329b4-34c8-45b1-8332-268107cddddd" />


### OPNsense Firewall Rules
<img width="1081" height="193" alt="image" src="https://github.com/user-attachments/assets/70a7decb-ae19-47a6-9aee-4f4c0ff895d0" />


### OPNsense Routes
<img width="1082" height="197" alt="image" src="https://github.com/user-attachments/assets/da88cbb2-8efd-4836-aed5-140848762e7b" />


### nginx Web Server Reachability

```md
![nginx Web Server Test](screenshots/nginx-web-test.png)
```
### Wazuh Dashboard

```md
![Wazuh Dashboard](screenshots/wazuh-dashboard.png)
```
### Zabbix Dashboard
<img width="1870" height="971" alt="image" src="https://github.com/user-attachments/assets/2cca121a-bd71-4e64-806d-6142e1e0214c" />


---

## Skills Demonstrated

* Network segmentation design
* DMZ architecture
* Firewall rule creation and validation
* pfSense administration
* OPNsense administration
* Static routing between firewalls
* Linux server deployment
* nginx service deployment
* DNS service deployment
* SIEM deployment with Wazuh
* Infrastructure monitoring with Zabbix
* Security logging and traffic validation
* Access control testing
* Technical documentation for GitHub

---

## Future Improvements
* Add a dedicated internal admin workstation.
* Allow Wazuh/Zabbix dashboard access only from the admin workstation.
* Add more Wazuh agents to internal and DMZ hosts.
* Forward pfSense and OPNsense firewall logs to Wazuh.
* Create Wazuh alerts for blocked traffic and suspicious authentication attempts.
* Add vulnerability scanning for the DMZ server.
* Add packet captures with Wireshark to validate traffic paths.
* Create a final attack-and-detect scenario using Wazuh alerts.

---

## Summary

This lab demonstrates how network segmentation can reduce exposure and improve visibility in an enterprise-style environment. By separating the DMZ, Internal Users, and Security networks, the architecture limits unnecessary access between systems and protects sensitive monitoring services.

The project also integrates Wazuh for security monitoring and Zabbix for infrastructure monitoring, making it useful for practicing firewall administration, SIEM deployment, service monitoring, routing, and practical network security validation.
