[Back](./README.md)


# Plugin Architecture

**Plugin Architecture in .NET** is a design approach that lets you build applications that can be **extended with new features (plugins) without modifying the core system**.

Think of it like adding apps to your phone: the phone (main app) works on its own, but you can install plugins (apps) to add new capabilities.

---

## 🔧 Core Idea

A .NET application is split into:

* **Host (Main Application)** → the core system
* **Plugins (Modules/Extensions)** → separate components loaded at runtime

The host doesn’t need to know plugin details in advance—it just knows a **contract (interface)**.

---

## 🧱 Basic Structure

### 1. Shared Contract (Interface)

You define a common interface that all plugins must implement:

```csharp
public interface IPlugin
{
    string Name { get; }
    void Execute();
}
```

---

### 2. Plugin Implementation

Each plugin is a separate project (DLL):

```csharp
public class HelloPlugin : IPlugin
{
    public string Name => "Hello Plugin";

    public void Execute()
    {
        Console.WriteLine("Hello from plugin!");
    }
}
```

---

### 3. Host Application

The main app dynamically loads plugins:

```csharp
var pluginFiles = Directory.GetFiles("Plugins", "*.dll");

foreach (var file in pluginFiles)
{
    var assembly = Assembly.LoadFrom(file);
    var types = assembly.GetTypes()
        .Where(t => typeof(IPlugin).IsAssignableFrom(t) && !t.IsInterface);

    foreach (var type in types)
    {
        var plugin = (IPlugin)Activator.CreateInstance(type);
        Console.WriteLine($"Running: {plugin.Name}");
        plugin.Execute();
    }
}
```

---

## ⚙️ How It Works in .NET

.NET provides tools that make plugin architecture powerful:

* **Reflection** → load assemblies dynamically
* **AssemblyLoadContext** → isolate plugins (important in .NET Core / .NET 5+)
* **Dependency Injection** → manage plugin dependencies
* **MEF (Managed Extensibility Framework)** → built-in plugin system (older but still used)

---

## 🧠 Key Benefits

### ✅ 1. Extensibility

Add new features without touching the core code.

### ✅ 2. Loose Coupling

Main app doesn’t depend on plugin implementations.

### ✅ 3. Scalability

Multiple teams can build plugins independently.

### ✅ 4. Maintainability

Easier updates and bug fixes.

---

## ⚠️ Challenges

* Version compatibility issues
* Dependency conflicts
* Security risks (loading unknown DLLs)
* Plugin isolation complexity

---

## 🧩 Real-World Examples

* IDEs like Visual Studio (extensions/plugins)
* CMS systems (modular features)
* Game engines (mods)
* Enterprise apps (custom business modules)

---

## 🧪 Modern .NET Approach

In **.NET 6+ / .NET 7+**, a common pattern is:

* Use **`AssemblyLoadContext`** for isolation
* Define shared interfaces in a **shared project**
* Load plugins from a **Plugins folder**
* Optionally use **DI container** to wire everything

---

## 🧭 When to Use Plugin Architecture

Use it when:

* You expect **frequent feature extensions**
* You want **third-party integrations**
* Your app needs to be **modular**

Avoid it if:

* The app is small/simple
* Performance is extremely critical
* You don’t need runtime extensibility

---

## 💡 Simple Analogy

* **Host app** = Operating system
* **Plugins** = Apps you install
* **Interface** = Rules every app must follow


---

## Is each plugin a separate project?
Short answer: **Yes—plugins are *usually* separate projects**, but they don’t *have to be*. Let me explain the nuance so you really understand how it works in .NET.

---

## ✅ The Common (Recommended) Setup

In most real-world .NET systems, you’ll have:

### 🧩 1. Shared Contract Project

* Contains interfaces like `IPlugin`
* Referenced by both host and plugins

### 🧩 2. Host Application

* Loads DLLs dynamically
* Does NOT reference plugin projects directly

### 🧩 3. Plugin Projects (Separate Class Libraries)

* Each plugin is its own `.csproj`
* Compiled into its own `.dll`

👉 This is the **standard plugin architecture**

---

## 🗂️ Typical Folder Structure

```
Solution
│
├── Plugin.Abstractions   (IPlugin interface)
├── HostApp               (Main app)
├── Plugin.Hello          (DLL 1)
├── Plugin.Logging        (DLL 2)
```

At runtime:

```
HostApp/
 └── Plugins/
      ├── Plugin.Hello.dll
      ├── Plugin.Logging.dll
```

---

## 🔥 Why Separate Projects?

### 1. Independent Development

Each plugin can be:

* Built separately
* Versioned independently
* Maintained by different teams

---

### 2. True Runtime Loading

If plugins were inside the same project:

* They’d already be compiled into the app
* No real “dynamic loading” benefit

---

### 3. Decoupling

The host:

* Knows only `IPlugin`
* Doesn’t reference plugin implementations

---

## ⚠️ Can Plugins Be in the Same Project?

Technically… yes.

You *could* define multiple plugin classes in one project:

```csharp
public class PluginA : IPlugin { }
public class PluginB : IPlugin { }
```

But then:

* You lose dynamic loading
* You lose modular deployment
* You’re not really using plugin architecture properly

👉 That becomes just a **modular design**, not a true plugin system

---

## 🧠 Key Insight

The **real power of plugin architecture** comes from:

> “The host doesn’t know about plugins at compile time.”

If your plugins are in the same project:

* The host *does* know them
* So it defeats the purpose

---

## 🧪 Advanced Scenario (Still Separate, but Smarter)

Sometimes plugins:

* Are distributed as NuGet packages
* Are downloaded at runtime
* Are loaded in isolated contexts (`AssemblyLoadContext`)

But they are still:
👉 **separate compiled assemblies (DLLs)**

---

## 🧭 Rule of Thumb

* ✔ Want runtime extensibility → **Separate projects (DLLs)**
* ✔ Want clean architecture only → same project is fine
* ❌ Want plugins but tightly coupled → not real plugins

---

## 💡 Simple Mental Model

* Same project → “features”
* Separate DLLs → “plugins”

---

## Example:

Let’s build a **simple, real plugin architecture example in .NET** step by step so you can clearly see how separate DLLs work together.

---

# 🧪 Example: Basic Plugin System in .NET

We’ll create **3 projects**:

1. `Plugin.Abstractions` → shared interface
2. `Plugin.Hello` → plugin (DLL)
3. `HostApp` → main app that loads plugins

---

## 🧩 1. Plugin.Abstractions (Shared Contract)

Create a **Class Library**:

```csharp
// IPlugin.cs
public interface IPlugin
{
    string Name { get; }
    void Execute();
}
```

👉 Both the host and plugins reference this project.

---

## 🔌 2. Plugin Project (Plugin.Hello)

Create another **Class Library** and reference `Plugin.Abstractions`.

```csharp
using System;

public class HelloPlugin : IPlugin
{
    public string Name => "Hello Plugin";

    public void Execute()
    {
        Console.WriteLine("Hello from the plugin!");
    }
}
```

👉 Build this → you’ll get:

```
Plugin.Hello.dll
```

---

## 🖥️ 3. Host Application

Create a **Console App** and reference only `Plugin.Abstractions`.

---

### 📁 Put Plugins in a Folder

Inside your host output folder:

```
HostApp/bin/Debug/net8.0/
 └── Plugins/
      └── Plugin.Hello.dll
```

---

### ⚙️ Host Code (Dynamic Loading)

```csharp
using System.Reflection;

var pluginFolder = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "Plugins");

var pluginFiles = Directory.GetFiles(pluginFolder, "*.dll");

foreach (var file in pluginFiles)
{
    var assembly = Assembly.LoadFrom(file);

    var pluginTypes = assembly.GetTypes()
        .Where(t => typeof(IPlugin).IsAssignableFrom(t) && !t.IsInterface);

    foreach (var type in pluginTypes)
    {
        var plugin = (IPlugin)Activator.CreateInstance(type);

        Console.WriteLine($"Loaded: {plugin.Name}");
        plugin.Execute();
    }
}
```

---

## ▶️ Output

```
Loaded: Hello Plugin
Hello from the plugin!
```

---

## 🧠 What Just Happened?

Step-by-step:

1. Host scans `Plugins/` folder
2. Finds `.dll` files
3. Loads them using **Reflection**
4. Looks for classes implementing `IPlugin`
5. Creates instances dynamically
6. Executes them

👉 The host NEVER referenced `Plugin.Hello` directly.

---

## 🔥 Add Another Plugin (No Host Changes!)

Create a new project:

```csharp
public class TimePlugin : IPlugin
{
    public string Name => "Time Plugin";

    public void Execute()
    {
        Console.WriteLine(DateTime.Now);
    }
}
```

Copy `Plugin.Time.dll` into `Plugins/` folder.

---

### ▶️ New Output

```
Loaded: Hello Plugin
Hello from the plugin!

Loaded: Time Plugin
4/5/2026 10:15:00 PM
```

👉 You added a feature **without touching the host code** 💥

---

## ⚠️ Important Notes

### ✔ Must Share the Same Interface

Both host and plugins must use the same `IPlugin` assembly.

---

### ✔ DLL Placement Matters

If the DLL isn’t in the `Plugins` folder → it won’t load.

---

### ✔ Version Compatibility

If interface changes → old plugins may break.

---

## 🚀 Bonus: Cleaner Version (Optional Filter)

You can restrict loading like this:

```csharp
.Where(t => typeof(IPlugin).IsAssignableFrom(t)
         && !t.IsInterface
         && !t.IsAbstract);
```

---

## 💡 Big Picture

This example shows:

* ✅ True runtime extensibility
* ✅ Loose coupling
* ✅ Independent deployment

---
[Back](./README.md)
