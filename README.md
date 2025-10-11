# VM-Resource-Contention-Remediation-
This project details the Root Cause Analysis (RCA) and manual remediation of severe resource contention on a Virtual Machine (VM). The issue, reported as VM sluggishness, was traced to system-critical services, including Windows Update (wuauserv), Windows Search, and the protected Windows Update Medic Service (WaaSMedicSvc).

## Disabling Windows Update Services via Services Manager
