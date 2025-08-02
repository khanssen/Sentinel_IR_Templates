
# Playbook: Block-AADUser-And-Notify

##  Purpose
This Logic App playbook is designed to automatically respond to credential-based attacks detected by Microsoft Sentinel. When triggered by an alert, it disables the affected Azure AD user account, notifies the security team via Microsoft Teams, and sends an email to a designated security group.

---

##  Trigger
- This playbook is triggered by an **Azure Sentinel analytics rule** that emits alerts containing a `Properties.Entities[].UPN` structure.

---

##  Actions Performed
1. Parses the alert to extract affected user(s)
2. Sends a request to **Microsoft Graph API** to disable the user account
3. Posts a **MessageCard** to a **Microsoft Teams channel** via webhook
4. Sends an **email notification** to the configured security group using the Office 365 connector

---

##  Assumptions & Prerequisites

### Sentinel / Logic App
- Microsoft Sentinel is deployed and connected to **Microsoft Entra ID (Azure AD)**.
- The triggering Sentinel analytics rule emits alerts with a `Properties.Entities` array that includes a valid `UPN`.
- Logic App is linked to Sentinel via an **Automation Rule** and is permitted to respond to alerts.
- The Logic App is authorized to use **HTTP**, **Graph API**, and **Office 365** connectors.

### Azure AD / Graph API
- The Logic App has permission to call Graph API (e.g., `User.ReadWrite.All`) using a **Managed Identity** or service principal.
- The target users are **cloud-only or hybrid** accounts that support programmatic disablement.
- The UPN exists and can be resolved by Microsoft Graph.

### Office 365 Email
- The `office365` connector is provisioned and authenticated in the Logic App resource group.
- The email address (e.g., `secops@yourdomain.com`) exists and can receive automated alerts.

### Microsoft Teams
- A Teams **channel is configured** with an **Incoming Webhook connector**.
- The webhook URL used in the playbook is valid and active.

### Runtime & Network
- Logic App runtime has outbound access to:
  - `graph.microsoft.com`
  - Office 365 mail endpoint
  - Teams webhook endpoint
- Network Security Groups or firewalls do not block HTTP POST traffic.

---

##  Known Gaps or To-Do
- This version assumes a **single user per alert**. For multiple users, wrap the user-related actions inside a `For_each` loop on `Entities`.
- Graph API call currently assumes Graph token management is handled externally or via connector authentication.

---

##  Outputs
- User account disabled in Azure AD
- Teams alert posted to SOC/IR channel
- Email sent to security group for ticketing or awareness

---
