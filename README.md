# Network-Segmentation
## Objective

## Lab Architecture

## Network Zones

## IP Addressing Plan

## Firewall Policy Matrix
| Source | Destination | Service | Policy Decision | Why? |
|---|---|---|---|---|
| Internet | DMZ Web Server | HTTP/HTTPS | Allow only required public web ports | The DMZ web server is intentionally public-facing, so external users should be able to access the hosted web service. Only HTTP/HTTPS are allowed to reduce the exposed attack surface. |
| Internet | DMZ Web Server | SSH | Deny | SSH is an administrative service and should not be exposed to untrusted internet hosts. Remote administration should only be allowed from trusted internal admin systems. |
| Internet | Internal Network | Any | Deny by default | Internal enterprise systems should never be directly reachable from the internet. This prevents external scanning, exploitation, and unauthorized access to private assets. |
| Internet | DNS Server | DNS UDP/TCP 53 | Allow only public DNS queries | This is allowed only because the DNS server is treated as a public authoritative DNS server in the DMZ. It should answer public records only and must not expose internal DNS records or allow recursive queries. |
| Internet | DMZ Firewall | Management GUI / SSH | Deny | Firewall management interfaces are highly sensitive and should not be accessible from untrusted networks. Exposing them increases the risk of firewall compromise. |
| Internal Client | Internet | HTTP/HTTPS | Allow with filtering | Internal users may need web access for normal business activity, updates, and research. Filtering should be applied to reduce access to malicious or unnecessary destinations. |
| Internal Client | Internet | DNS | Deny direct public DNS | Internal clients should not query public DNS servers directly. DNS requests should go through the internal DNS service to preserve visibility, enforce policy, and support internal name resolution. |
| Internal Client | DMZ Web Server | HTTP/HTTPS | Allow | Internal users may need to access the organization’s public-facing or DMZ-hosted web service. Access is limited to web protocols only. |
| Internal Client | DMZ Web Server | SSH | Deny | Regular internal users should not administer DMZ servers. Blocking SSH limits unauthorized administrative access and reduces lateral movement risk. |
| Internal Admin | DMZ Web Server | SSH | Allow only from trusted admin host/subnet | Administrative access to the DMZ web server is required for maintenance, but it should be restricted to authorized admin systems rather than the entire internal network. |
| Internal Client | Active Directory | LDAP/Kerberos/DNS/SMB | Allow required AD services only | Internal clients require Active Directory services for authentication, domain membership, name resolution, and policy access. Only required AD-related services should be allowed. |
| Internal Client | SIEM | Web UI | Deny | Regular users should not access the SIEM interface because it may expose sensitive security events, logs, and investigation data. |
| Internal Admin / Security Analyst | SIEM | Web UI / SSH | Allow only from authorized security/admin systems | Security analysts and administrators require controlled access to the SIEM for monitoring, investigation, and maintenance. Access should be restricted to trusted users and systems. |
| DMZ Web Server | Internet | HTTP/HTTPS | Allow limited outbound web access | The DMZ web server may require limited outbound access for package updates or external service dependencies. This should be restricted and monitored to reduce command-and-control or data exfiltration risk. |
| DMZ Web Server | Internal Network | Any | Deny by default | A DMZ host is exposed to higher risk. If compromised, it should not be able to pivot into internal enterprise systems. |
| DNS Server | Internet | DNS UDP/TCP 53 | Deny outbound recursive DNS | The DMZ DNS server is treated as authoritative-only, so it should not resolve arbitrary external domains. This prevents it from being abused as an open resolver. |
| DNS Server | Internal Network | Any | Deny by default | A public-facing DNS server should not access internal systems. This protects internal infrastructure if the DMZ DNS server is compromised. |
| Active Directory | Internet | Any | Deny by default | Domain controllers and AD services should not initiate general internet traffic. Keeping AD isolated reduces exposure and protects critical identity infrastructure. |
| Active Directory | Internal Client | Kerberos/LDAP/DNS/SMB | Allow required AD responses only | Active Directory must respond to legitimate internal client authentication, directory, DNS, and domain-related requests. Access should remain limited to required services. |
| Active Directory | DMZ Web Server | Any | Deny by default | Active Directory should not directly communicate with public-facing DMZ servers unless there is a specific business requirement. This reduces identity infrastructure exposure. |
| Active Directory | SIEM | Syslog / Windows Event Forwarding / Agent | Allow security log forwarding | Authentication and directory logs are important for detecting suspicious activity, failed logins, privilege abuse, and lateral movement. These events should be forwarded to the SIEM. |
| DMZ Web Server | SIEM | Log forwarding / Agent traffic | Allow only required logging traffic | The DMZ web server should send security and application logs to the SIEM so activity on the public-facing server can be monitored. Only the required logging ports should be allowed. |
| SIEM | Internet | Updates / Threat intelligence | Allow limited outbound access | The SIEM may need outbound access for software updates, signature updates, and threat intelligence feeds. This access should be limited to trusted destinations where possible. |
| SIEM | Active Directory | LDAP/LDAPS | Deny unless explicitly required | The SIEM should not query Active Directory unless AD integration is required for authentication or enrichment. Denying this by default limits unnecessary access to identity infrastructure. |
| DMZ Firewall | DMZ Web Server | Forwarded HTTP/HTTPS | Allow controlled forwarding | The DMZ firewall forwards approved public web traffic to the DMZ web server. Only HTTP/HTTPS should be forwarded to match the intended public service. |
| DMZ Firewall | Internal Network | Any | Deny by default | The perimeter/DMZ firewall should not provide direct access from the DMZ or internet-facing side into the internal network. This preserves separation between semi-trusted and trusted zones. |
| Enterprise Firewall | Active Directory | Required internal services | Allow required internal traffic only | The enterprise firewall must permit necessary internal traffic to Active Directory so internal clients can authenticate and use domain services. The rule should be limited to required ports and trusted internal sources. |
| Enterprise Firewall | SIEM | Syslog / management logs | Allow firewall log forwarding | Firewall logs should be sent to the SIEM to support monitoring, incident detection, and auditability of allowed and blocked traffic. |
| Enterprise Firewall | Internet | Updates / NTP / DNS | Allow limited firewall system traffic | The firewall may need limited outbound access for updates, time synchronization, and name resolution. This should be restricted to trusted services and not treated as general internet access. |
| Internal Admin | DMZ Firewall | Management GUI / SSH | Allow only from trusted admin host/subnet | Firewall administration should be restricted to authorized internal admin systems. This prevents unauthorized users from modifying perimeter security controls. |
| Internal Admin | Enterprise Firewall | Management GUI / SSH | Allow only from trusted admin host/subnet | The enterprise firewall controls access to sensitive internal resources, so management access must be tightly restricted to trusted administrators and systems. |
## Test Plan

## Packet Capture Evidence

## Security Analysis

## Lessons Learned

## Future Improvements
