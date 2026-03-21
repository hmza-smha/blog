
[Back](./README.md)

To **start**, **stop**, and **restart** a Windows Service using **PowerShell**, use the following built-in cmdlets:

---

## 🔹 Start a Service

```powershell
Start-Service -Name "AIAgentService"
```

---

## 🔹 Stop a Service

```powershell
Stop-Service -Name "AIAgentService"
```

You can force it to stop (if it’s stuck) using `-Force`:

```powershell
Stop-Service -Name "AIAgentService" -Force
```

---

## 🔹 Restart a Service

```powershell
Restart-Service -Name "AIAgentService"
```

> Replace `"AIAgentService"` with your actual service name as registered.

---

## 🔍 Check Service Status

```powershell
Get-Service -Name "AIAgentService"
```

Returns something like:

```
Status   Name               DisplayName
------   ----               -----------
Running  AIAgentService     AI Background Agent
```

---





## ✅ Step-by-Step: Convert to Windows Service

### 1. **Modify `Program.cs` to support Windows Service**

Add `.UseWindowsService()` to your host builder:

```csharp
var host = Host.CreateDefaultBuilder(args)
    .UseWindowsService() // 👈 ADD THIS LINE
    .ConfigureAppConfiguration((context, config) =>
    {
        config.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true);
    })
    .ConfigureServices((context, services) =>
    {
        // your existing service registration code
    })
    .Build();

await host.RunAsync();
```

---

### 2. **Publish your application**

Run the following command to publish your app to a folder:

```bash
dotnet publish -c Release -o C:\MyServices\AIAgentService
```

---

### 3. **Register as a Windows Service**

You can use PowerShell or `sc.exe` to register your app as a service.

#### 🟢 Option A: PowerShell

```powershell
New-Service -Name "AIAgentService" `
            -BinaryPathName "C:\MyServices\AIAgentService\YourApp.exe" `
            -DisplayName "AI Background Agent" `
            -Description "Runs AI background jobs with Hangfire" `
            -StartupType Automatic
```

Then start it:

```powershell
Start-Service -Name "AIAgentService"
```


To **remove (uninstall) an existing Windows Service**, you can use either PowerShell or the `sc.exe` command-line tool.

---


## 🔧 Option 1: Remove using PowerShell

```powershell
# Stop the service (if running)
Stop-Service -Name "AIAgentService" -Force

# Remove the service
Remove-Service -Name "AIAgentService"
```

> ⚠️ `Remove-Service` is available in **PowerShell 6+**. If you're using older Windows PowerShell (5.1), use `sc delete` below.

---

## 🔧 Option 2: Remove using `sc.exe` (works everywhere)

```cmd
sc stop AIAgentService
sc delete AIAgentService
```

> Replace `AIAgentService` with your actual service name if different.

---

## 🔍 Confirm it’s gone

After deletion, refresh the **Services MMC** (`services.msc`) to verify the service is removed.


---
## Tables

### **1. Job**

* **Purpose:** Stores serialized background jobs, their method calls, arguments, creation time, and current state reference.

### **2. State**

* **Purpose:** Records job states (e.g., `Enqueued`, `Processing`, `Failed`, `Succeeded`) along with timestamps and reasons. Keeps full job lifecycle history.

### **3. JobParameter**

* **Purpose:** Stores additional key-value parameters attached to jobs for extended metadata or configuration.

### **4. JobQueue**

* **Purpose:** Holds jobs waiting to be processed, organized by queue name. Workers dequeue from here.

### **5. Hash**

* **Purpose:** Stores arbitrary key-value data in a dictionary-like way, used for recurring job info, server heartbeats, etc.

### **6. Counter**

* **Purpose:** Holds increments/decrements for counters like statistics (jobs succeeded, failed), with optional expiry.

### **7. AggregatedCounter**

* **Purpose:** Stores pre-aggregated summary counters for efficient statistics querying.

### **8. Server**

* **Purpose:** Contains info about Hangfire server instances running workers — their name, heartbeat, queues served, etc.

### **9. List**

* **Purpose:** Manages named lists for storing job IDs (e.g., for processing or scheduled jobs).

### **10. Set**

* **Purpose:** Manages named sets (unique collections) for jobs or other IDs, used in scheduling or recurring jobs.

### **11. Schema**

* **Purpose:** Holds version info of the Hangfire schema installed in the database (for migration and compatibility).

---

### Summary of how they fit together:

* **`Job` + `State` + `JobParameter`**: Job metadata and state tracking.
* **`JobQueue`**: Queues for background processing.
* **`Server`**: Worker server tracking.
* **`Hash`, `Counter`, `AggregatedCounter`**: Configuration and stats storage.
* **`List`, `Set`**: Collections for scheduling and job management.
* **`Schema`**: Version control of DB schema.




---
[Back](./README.md)