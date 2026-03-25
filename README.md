# azure-hybrid-lab

Fully documented Azure hybrid cloud lab validating AZ-104 Azure Administrator competencies. Connects an on-premises Windows Server 2022 Active Directory domain to Azure Entra ID through Azure AD Connect, with RBAC enforcement, Azure Monitor alerting, and Microsoft Intune endpoint management.

---

## Lab Components

| Component | Technology | Purpose |
|-----------|-----------|---------|
| On-prem identity | Windows Server 2022 AD | Domain controller for contoso.local |
| Hybrid sync | Azure AD Connect | Password hash sync to Entra ID |
| Cloud VMs | Azure B1s (Win + Ubuntu) | Workload VMs in segmented VNet |
| Network security | NSG rules | Subnet-level traffic control |
| Identity governance | RBAC | Least-privilege role assignments |
| Monitoring | Azure Monitor + Log Analytics | VM metrics, KQL queries, alert rules |
| Endpoint management | Microsoft Intune | Compliance policies, app deployment |

---

## Skills Demonstrated

- **AZ-104:** VNet/subnet design, NSG configuration, VM deployment, hybrid identity
- **Azure Entra ID:** User sync, SSO, UPN matching, AD Connect configuration
- **RBAC:** Subscription/RG/resource-level assignment, least privilege enforcement
- **Azure Monitor:** Log Analytics workspace, diagnostic settings, KQL queries, metric alerts
- **Microsoft Intune:** Device enrollment, compliance policies, config profiles, app deployment

---

## Outcome

Fully functional hybrid environment demonstrating end-to-end AZ-104 skills: on-premises AD synced to Entra ID, VMs deployed in a segmented VNet with NSG rules, RBAC applied at appropriate scopes, monitoring alerts active, and Intune enforcing device compliance - all documented with step-by-step procedures.