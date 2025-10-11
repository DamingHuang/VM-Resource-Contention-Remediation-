# VM-Resource-Contention-Remediation-
This project details the **Root Cause Analysis (RCA)** and manual remediation of severe resource contention on a **Virtual Machine (VM)**. The issue, reported as VM sluggishness, was traced to system-critical services, including **Windows Update (wuauserv)**, **Windows Search**, and the protected **Windows Update Medic Service (WaaSMedicSvc)**.

## Disabling Windows Update Services via Services Manager
Disabling core update services is a common method to reduce background resource consumption and fix a sluggish **Virtual Machine (VM)** by preventing the system's update mechanism from consuming CPU and disk I/O.

### Step-by-Step Procedure

1. Access **Services Manager**: Press the **Windows Key + R** to open the **Run** dialog, type `services.msc`, and press **Enter**. 

![Access Services Manager Screenshot](https://github.com/DamingHuang/LabScreenShots/blob/53279bf018269015be718ae96ef6d755fb30bbb7/VMscreenshot/ss1.png)

2. **Locate Windows Update Service**: In the list of services, find the service named **Windows Update (wuauserv)** ![](https://github.com/DamingHuang/LabScreenShots/blob/53279bf018269015be718ae96ef6d755fb30bbb7/VMscreenshot/ss2.png)and **Windows Update Medic Service (WaaSMedicSvc)** ![](https://github.com/DamingHuang/LabScreenShots/blob/53279bf018269015be718ae96ef6d755fb30bbb7/VMscreenshot/ss3.png).

3. Stopping the Service (Immediate Halt)
    - **Locate** the desired service (e.g., **Windows Update**, or **Windows Update Medic Service**).
    - **Right-click** on the service.
    - Look at the menu:
        - If **Start** is an option, the service is currently stopped. ![](https://github.com/DamingHuang/LabScreenShots/blob/f547d25221167e0a409384054a4bb0b45899a7cb/VMscreenshot/ss4.png)
        - If **Stop** is an option, select **Stop** to immediately halt the service's current operations. ![](https://github.com/DamingHuang/LabScreenShots/blob/f547d25221167e0a409384054a4bb0b45899a7cb/VMscreenshot/ss5.png)

4. Do the same for **Windows Update Medic Service (WaaSMedicSvc)** ![](https://github.com/DamingHuang/LabScreenShots/blob/f547d25221167e0a409384054a4bb0b45899a7cb/VMscreenshot/ss3.png) .

---

### Disabling the Service (Permanent Prevention)

This step prevents the service from starting again upon the next boot or system trigger.

1. **Right-click** on the service (**Windows Update (wuauserv)** ![](https://github.com/DamingHuang/LabScreenShots/blob/f547d25221167e0a409384054a4bb0b45899a7cb/VMscreenshot/ss2.png) and **Windows Update Medic Service (WaaSMedicSvc)** ![](https://github.com/DamingHuang/LabScreenShots/blob/f547d25221167e0a409384054a4bb0b45899a7cb/VMscreenshot/ss3.png)  and select **Properties**.

2. In the **Properties** window, find the **Startup Type** dropdown menu.

3. Change the value to **Disabled**.

4. Click **Apply**, then **OK**. 

![](https://github.com/DamingHuang/LabScreenShots/blob/f547d25221167e0a409384054a4bb0b45899a7cb/VMscreenshot/ss7.png) 

5. Please note that the **Windows Update Medic Service (WaaSMedicSvc)** may result in an **"Access Denied"** error.

6. If it does, **run** the following commands:
    ```console
    # Attempt 1 (PowerShell)
    & sc.exe config WaaSMedicSvc start= disabled
    
    # Attempt 2 (CMD via PowerShell)
    cmd /c "sc config WaaSMedicSvc start= disabled"
    ```

If the commands result in an **'Access Denied'** error, the next step would be **Registry Modification**.

---

## Registry Modification

### Step-by-Step Procedure

1. **Win Key + R** to open **Run**, input `regedit`, and press **Enter**. 

![](https://github.com/DamingHuang/LabScreenShots/blob/36f5470d4885bc5828b815c12a0af5e126c056f7/VMscreenshot/ss8.png)

2. Registry Path:
    `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WaaSMedicSvc`

3. Value Modified: **Start** (`DWORD 32-bit`).

   ![](https://github.com/DamingHuang/LabScreenShots/blob/36f5470d4885bc5828b815c12a0af5e126c056f7/VMscreenshot/ss8.1.png)

4. Value Data Set To: **4** (**Disabled**).

> The **Start** value specifies when the service should be started. It can have one of the following values:
> * **0x0 (Boot)**: Loaded by the boot loader.
> * **0x1 (System)**: Loaded by the I/O subsystem.
> * **0x2 (Automatic)**: Loaded automatically by the Service Control Manager during system startup.
> * **0x3 (Demand)**: Loaded automatically by PnP if it is needed for a device.
> * **0x4 (Disabled)**: The service is disabled and will not be loaded.
>
> [`HKLM\SYSTEM\CurrentControlSet\Services` Registry Tree](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/hklm-system-currentcontrolset-services-registry-tree#start)

5. System Action: **VM was rebooted** to apply the change.

6. If both **Windows Update (wuauserv)** and **Windows Update Medic Service (WaaSMedicSvc)** are disabled, and updates are still running even with the registry disabled, it means **it's more than likely a Task Scheduler issue**.

---

## Disable Update Tasks in Task Scheduler / Task Scheduler Remediation

Windows uses the **Task Scheduler** to launch update processes on a schedule, even if the main service is disabled. You should disable these tasks to ensure updates don't run.

### Step-by-Step Procedure

1. Open the **Task Scheduler** (`taskschd.msc`).
2. Navigate to the following folder in the left pane: **Task Scheduler Library → Microsoft → Windows → UpdateOrchestrator**.
3. In the center pane, you will see tasks like **UpdateAssistant** and **UpdateModelTask**.
4. For each task:
    - **Right-click** the task (e.g., `UpdateOrchestrator`).
        - **USO_UxBroker** (also referred to by its service, `UsoSvc`, which runs in the background) coordinates the entire update process: scanning, downloading, installing, and restarting. If its associated task is active, it will bypass disabled settings to try and ensure updates happen.
        - **Schedule Scan** is the task that triggers the initial check for new updates, which often requires or re-activates the disabled **Windows Update** service (`wuauserv`) to complete the scan.
    - Select **Disable**. (This will change its status from "Ready" to "Disabled").

5. But you may run into the following issue:
    * "The user account you are operating under does not have permission to disable this task.”

 ![](https://github.com/DamingHuang/LabScreenShots/blob/c78a68eaa4bbeb7dbdac28898ce7dd4c7ba4aee1/VMscreenshot/ss10.png)

6. If you run into the permission issue:
    - **Uncheck** the **Enabled** box.
    
   ![](https://github.com/DamingHuang/LabScreenShots/blob/c78a68eaa4bbeb7dbdac28898ce7dd4c7ba4aee1/VMscreenshot/ss11.png)
 
    - Click **Ok**. 
       
       ![](https://github.com/DamingHuang/LabScreenShots/blob/8bec71b0770e5e18f7dd85b22e4c50cee9d2ba88/VMscreenshot/ss11.1.png)
     - Click **Ok**.

     
       ![](https://github.com/DamingHuang/LabScreenShots/blob/46c84ac96d69142810b03d9cd1aa409e030c3f75/VMscreenshot/s12.png)
 


 ---

### Advanced Task Scheduler Workaround

If entering your administrator password doesn't work, it's often because Microsoft has locked down these **Windows Update tasks**. They are often configured to run with the **highest privileges** (**SYSTEM** account) and can't be modified, disabled, or deleted through the normal **Task Scheduler** interface.

To get around this and successfully disable the tasks in `UpdateOrchestrator`, the most common advanced workaround is to use a special administrative tool like **PsExec** to open the **Task Scheduler** under the **SYSTEM** account itself, which bypasses the permissions locks.

 ![](https://github.com/DamingHuang/LabScreenShots/blob/46c84ac96d69142810b03d9cd1aa409e030c3f75/VMscreenshot/s13.png)


#### How to Proceed (If Password Fails)

1. Download **PsExec**: Download the PsTools package from the Microsoft Sysinternals website and **extract** the contents to an easy-to-access folder (e.g., `C:\PSTools`).
2. Open an **Elevated Command Prompt / PowerShell** (right-click and select **Run as administrator**).
3. Run the following commands:
    ```console
    # Change directory to where PsExec is located
    cd C:\Users\[User]\Desktop\PSTOOLS
    
    # Execute Task Scheduler under the SYSTEM account
    psexec.exe -i -s C:\Windows\System32\mmc.exe C:\Windows\System32\taskschd.msc
    ```
    *Replace `[User]` with your User profile name.*
   
 *Example:*
 
 
  cd C:\Users\HDM\Desktop\PSTools

   ![Screenshot of disabled Task Scheduler tasks](screenshots/disabled_tasks.png)

 
 
 
 
 4. A new **Task Scheduler** window will open. This is the window running under the **SYSTEM account**, and you will not be asked for a password to disable the tasks.
    - Navigate to: **Task Scheduler Library → Microsoft → Windows → UpdateOrchestrator**.
    - **Right-click** on the following tasks and select **Disable**:
        1. **USO_UxBroker**

 ![](https://github.com/DamingHuang/LabScreenShots/blob/740c16c46551140292de002c161217297453236b/VMscreenshot/ss15.png)

        2. **Schedule Scan
    - A screenshot of the tasks successfully disabled. 
 ![Screenshot of disabled Task Scheduler tasks](screenshots/disabled_tasks.png)

5. **A screenshot of the tasks successfully disabled** is shown below:
    ![Screenshot of disabled Task Scheduler tasks](screenshots/disabled_tasks.png)

6. After the console is exited, a **“C:\Windows\System32\mmc.exe exited on HDM with error code 0.”** alert will show up on the PowerShell.
    ![Screenshot of disabled Task Scheduler tasks](screenshots/disabled_tasks.png)

