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

![Application Rule – Allow Google](./images/app-rule-google.png)

- Source: Workload subnet (`10.0.1.0/24`)
- Protocols: HTTP, HTTPS
- Destination: `www.google.com`

---

### Network Rule – DNS Restriction
Allows DNS queries only to trusted public DNS servers.

![Network Rule – DNS](./images/network-rule-dns.png)

- Protocol: UDP
- Port: 53
- Destination IPs: `8.8.8.8`, `8.8.4.4`

---

### DNAT Rule – Secure RDP Access
Provides RDP access through the firewall without assigning a public IP to the VM.

![DNAT Rule – RDP](./images/dnat-rule-rdp.png)

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

![Allowed Traffic – Google](./images/test-allowed.png)

---

### Blocked Traffic
Outbound access to `microsoft.com` is blocked by default-deny behavior.

![Blocked Traffic – Microsoft](./images/test-blocked.png)

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
