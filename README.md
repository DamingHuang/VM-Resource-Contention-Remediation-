# VM-Resource-Contention-Remediation-
This project details the Root Cause Analysis (RCA) and manual remediation of severe resource contention on a Virtual Machine (VM). The issue, reported as VM sluggishness, was traced to system-critical services, including Windows Update (wuauserv), Windows Search, and the protected Windows Update Medic Service (WaaSMedicSvc).

## Disabling Windows Update Services via Services Manager
Disabling core update services is a common method to reduce background resource consumption and fix a sluggish Virtual Machine (VM) by preventing the system's update mechanism from consuming CPU and disk I/O.

### Step-by-Step Procedure
1. <mark> Access Services Manager: </mark>   Press the <mark>Windows Key + R</mark> to open the Run dialog, type services.msc, and press Enter.
![Access Services Manager Screenshot](https://github.com/DamingHuang/LabScreenShots/blob/53279bf018269015be718ae96ef6d755fb30bbb7/VMscreenshot/ss1.png)

2. <mark>Locate Windows Update Service:</mark> In the list of services, find the service named <mark>Windows Update (wuauserv)</mark> ![](https://github.com/DamingHuang/LabScreenShots/blob/53279bf018269015be718ae96ef6d755fb30bbb7/VMscreenshot/ss2.png) & <mark>Windows Update Medic Service (WaaSMedicSvc)![](https://github.com/DamingHuang/LabScreenShots/blob/53279bf018269015be718ae96ef6d755fb30bbb7/VMscreenshot/ss3.png)</mark> 

3. Stopping the Service (Immediate Halt)
   - Locate the desired service (e.g., Windows Update, or Windows Update Medic Service).
   - Right-click on the service.
   - Look at the menu:
       - If Start is an option, the service is currently stopped.   
       ![](https://github.com/DamingHuang/LabScreenShots/blob/f547d25221167e0a409384054a4bb0b45899a7cb/VMscreenshot/ss4.png)
       - If Stop is an option, select Stop to immediately halt the service's current operations.
       ![](https://github.com/DamingHuang/LabScreenShots/blob/f547d25221167e0a409384054a4bb0b45899a7cb/VMscreenshot/ss5.png)

5. Do the same for Windows Update Medic Service (WaaSMedicSvc)  ![](https://github.com/DamingHuang/LabScreenShots/blob/f547d25221167e0a409384054a4bb0b45899a7cb/VMscreenshot/ss3.png)  
### Disabling the Service (Permanent Prevention)
This step prevents the service from starting again upon the next boot or system trigger.

1. <mark> Right-click</mark>  on the service (<mark> Windows Update (wuauserv)![](https://github.com/DamingHuang/LabScreenShots/blob/53279bf018269015be718ae96ef6d755fb30bbb7/VMscreenshot/ss2.png) </mark>& <mark> Windows Update Medic Service (WaaSMedicSvc)![](https://github.com/DamingHuang/LabScreenShots/blob/53279bf018269015be718ae96ef6d755fb30bbb7/VMscreenshot/ss3.png)</mark> 
</mark>    and select <mark> Properties</mark>. 

2. In the Properties window, find the <mark>Startup Type</mark>  dropdown menu.
3. Change the value to <mark>Disabled</mark>.
4. Click <mark>Apply</mark>, then <mark>OK</mark>.

![](https://github.com/DamingHuang/LabScreenShots/blob/fa95df15d15a02f35f9a5b75be43cf9e0825d863/VMscreenshot/ss7.png)

5. Please note that the <mark>Windows Update Medic Service (WaaSMedicSvc)</mark> may result in an <mark>"Access Denied"</mark> error. 
6. If it does, <mark>run</mark> the following command:



    - <mark>& sc.exe config WaaSMedicSvc start= disabled </mark>
    - <mark>cmd /c "sc config WaaSMedicSvc start= disabled" </mark>


If the commands result in an <mark> 'Access Denied'</mark>  error, the next step would be <mark> Registry</mark>  Modification.

```console
PS C:\Windows\system32> & sc.exe config WaaSMedicSvc start= disabled
[SC] ChangeServiceConfig FAILED 5:


Access is denied.


PS C:\Windows\system32> cmd /c "sc config WaaSMedicSvc start= disabled"
[SC] ChangeServiceConfig FAILED 5:


Access is denied.


PS C:\Windows\system32>
```

## Registry Modification.
### Step-by-Step Procedure

1. <mark>Win Key + R </mark> =<mark> Run</mark>, input <mark>regedit</mark>

![](https://github.com/DamingHuang/LabScreenShots/blob/36f5470d4885bc5828b815c12a0af5e126c056f7/VMscreenshot/ss8.png)

2. Registry Path:
   
   <mark>HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WaaSMedicSvc</mark>

3. Value Modified: <mark>Start</mark> (DWORD 32-bit).
   
   ![](https://github.com/DamingHuang/LabScreenShots/blob/36f5470d4885bc5828b815c12a0af5e126c056f7/VMscreenshot/ss8.1.png)

5. Value Data Set To: <mark>4</mark> (Disabled). 

> The Start value specifies when the service should be started. It can have one of the following values:
> * 0x0 (Boot): Loaded by the boot loader.
> * 0x1 (System): Loaded by the I/O subsystem.
> * 0x2 (Automatic): Loaded automatically by the Service Control Manager during system startup.
> * 0x3 (Demand): Loaded automatically by PnP if it is needed for a device.
> * 0x4 (Disabled): The service is disabled and will not be loaded.
>
> [HKLM\SYSTEM\CurrentControlSet\Services Registry Tree](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/hklm-system-currentcontrolset-services-registry-tree#start)

6. System Action: VM was rebooted to apply the change.
7. If both **Windows Update (wuauserv)** and **Windows Update Medic Service (WaaSMedicSvc)** are disabled, and updates are still running even with the registry disabled, it means **it's more than likely a Task Scheduler issue**.
