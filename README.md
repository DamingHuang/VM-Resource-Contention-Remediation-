# VM-Resource-Contention-Remediation-
This project details the Root Cause Analysis (RCA) and manual remediation of severe resource contention on a Virtual Machine (VM). The issue, reported as VM sluggishness, was traced to system-critical services, including Windows Update (wuauserv), Windows Search, and the protected Windows Update Medic Service (WaaSMedicSvc).

## Disabling Windows Update Services via Services Manager
Disabling core update services is a common method to reduce background resource consumption and fix a sluggish Virtual Machine (VM) by preventing the system's update mechanism from consuming CPU and disk I/O.

### Step-by-Step Procedure
1. <mark> Access Services Manager: </mark>   Press the <mark>Windows Key + R</mark> to open the Run dialog, type services.msc, and press Enter.
<img width="1164" height="196" alt="image" src="https://github.com/user-attachments/assets/1c545c51-659a-4e06-98e9-54194a7c1a62" />

2. 
