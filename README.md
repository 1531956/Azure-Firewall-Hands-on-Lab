# Azure Firewall – Secure Workload with Controlled Egress & DNAT

## Overview
This project demonstrates how to secure a workload subnet in Microsoft Azure using **Azure Firewall**, **user-defined routes (UDRs)**, and **least-privilege traffic rules**.  
The focus is on **traffic flow control, outbound filtering, and secure inbound access**, not on basic portal navigation.

---

## Architecture

![Azure Firewall Architecture Diagram](./images/architecture.png)

**Components**
- Azure Virtual Network
- AzureFirewallSubnet
- Workload subnet (VM)
- Azure Firewall (Standard)
- Route table forcing all outbound traffic through the firewall
- Internet

**Traffic Flow**
- All outbound traffic from the workload subnet is routed to Azure Firewall
- Only explicitly allowed destinations are reachable
- Inbound RDP access is provided via DNAT (no public IP on the VM)

---

## Security Objectives
- Enforce **default-deny outbound traffic**
- Allow outbound access **only to approved destinations**
- Restrict DNS traffic to known resolvers
- Provide **controlled inbound access** without exposing the VM directly

---

## Firewall Configuration

### Application Rule – Controlled Web Access
Allows outbound HTTP/HTTPS traffic **only** to Google.
<img width="1918" height="818" alt="ApplicationRule" src="https://github.com/user-attachments/assets/7f1a24b6-e077-4af4-ae40-83a4c929a21c" />
<img width="1918" height="821" alt="AppRuleSuccess" src="https://github.com/user-attachments/assets/636d6138-d94e-49d8-ba59-d869f468e1bc" />

- Source: Workload subnet (`10.0.1.0/24`)
- Protocols: HTTP, HTTPS
- Destination: `www.google.com`

---

### Network Rule – DNS Restriction
Allows DNS queries only to trusted public DNS servers.
<img width="1918" height="817" alt="NetworkRule" src="https://github.com/user-attachments/assets/7d6e73a0-6a6e-4a9c-8916-d0fb6e798fc8" />
<img width="1918" height="818" alt="NetRuleSuccess" src="https://github.com/user-attachments/assets/1b3f87cf-4abe-4ba2-98d6-44e84698d6e5" />

- Protocol: UDP
- Port: 53
- Destination IPs: `8.8.8.8`, `8.8.4.4`

---

### DNAT Rule – Secure RDP Access
Provides RDP access through the firewall without assigning a public IP to the VM.
<img width="1918" height="820" alt="DNATRule" src="https://github.com/user-attachments/assets/f5d41460-dfec-4c31-b542-091fa84ad508" />
<img width="1918" height="820" alt="DNATRuleSuccess" src="https://github.com/user-attachments/assets/cb1d8993-3ab6-4efd-ac17-8393645201a1" />

- Public IP: Azure Firewall
- Port: TCP 3389
- Translated address: VM private IP

---

## Routing (UDR)

![Route Table – Force Firewall](./images/route-table.png)

- Destination: `0.0.0.0/0`
- Next hop: Virtual appliance
- Next hop IP: Azure Firewall private IP

This ensures **all outbound traffic** from the workload subnet is inspected by the firewall.

---

## Validation & Testing

### Allowed Traffic
Outbound access to `google.com` is successful.
<img width="1916" height="1015" alt="RDPGoogle" src="https://github.com/user-attachments/assets/2ea0f32f-d05d-41fe-9344-671439a219f6" />

---

### Blocked Traffic
Outbound access to `microsoft.com` is blocked by default-deny behavior.
<img width="1918" height="1016" alt="RDPMicrosoft" src="https://github.com/user-attachments/assets/43c7d539-0ae6-4dd3-937b-a40f96951903" />

Error observed:
> Action: Deny. Reason: No rule matched.

---

## Key Takeaways
- Azure Firewall requires **UDRs** to control traffic flow
- Application rules enable **FQDN-based egress filtering**
- DNS must be explicitly allowed or outbound access will fail
- DNAT provides secure inbound access without exposing workloads
- Default-deny policies are effective when paired with explicit allow rules

---

## Tools & Services Used
- Microsoft Azure
- Azure Firewall (Standard)
- Azure Virtual Network
- User-Defined Routes (UDR)
- Firewall Policy (Application, Network, DNAT rules)

---

## Credits
This project was inspired by an Azure Firewall lab demonstrated by  
[A Guide To Cloud](https://www.youtube.com/@AGuideToCloud).
