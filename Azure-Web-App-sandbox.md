_This article is provided with the intent of communicating various runtime execution limitations enforced by the Azure Web Apps platform.  This article will serve as the foremost point of communication regarding this topic and will be updated whenever changes occur._

### Introduction

All Azure Web Apps (as well as Mobile App/Services, WebJobs and Functions) run in a secure environment called a _sandbox_. Each app runs inside its own sandbox, isolating its execution from other instances on the same machine as well as providing an additional degree of security and privacy which would otherwise not be available.  The sandbox mechanism aims to ensure that each app running on a machine will have a minimum guaranteed level of service; furthermore, the runtime limits enforced by the sandbox protects apps from being adversely affected by other resource-intensive apps which may be running on the same machine.

### Purpose of the sandbox

The App Service platform execution environment differs from a local execution environment mainly due to multi-tenancy &mdash; because a single physical machine in the data center can be concurrently executing apps and services belonging to a large number of differing customers, resources are more constrained than in the case of an app running on a single machine.  The sandbox mechanism mitigates the risk of service disruption due to resource contention and depletion in two ways: it (1) ensures that each app receives a minimum guarantee of resources and quality-of-service, and conversely (2) enforces limits so that an app can not disrupt other concurrently-executing apps on the same machine.

A corollary but also important function that the sandbox provides is the ability to meter an app's resource usage (e.g. storage usage, network traffic, and compute time); the resource limits also serve to prevent abuse and attacks by malicious apps.

### Definition of the sandbox

Each app is viewed by the sandbox as a tree of _processes_, rooted by the app's main IIS (`w3wp.exe`) process.  Each app's lifetime begins with the creation of this `w3wp.exe` process, which, in turn, may spawn other child processes, such as `php-cgi.exe` or `node.exe`.  These processes are grouped together and are collectively subject to a _single_ set of limits, all enforced within the underlying Windows kernel.  It is common for a single app to have multiple processes involved in processing an incoming request.  The following table illustrates typical execution profiles and their "process groups," organized by programming language:

| Language | Processes in sandbox |
| -------- | -------------------- |
| C#       | `W3WP.EXE`, `CSC.EXE`, `CVTRES.EXE` |
| node.js  | `W3WP.EXE`, `NODE.EXE` |
| PHP      | `W3WP.EXE`, `PHP-CGI.EXE` |
| Java     | `W3WP.EXE`, `JAVA.EXE` |

The vast majority of the execution limits outlined in this article apply to the entire sandbox: that is, to all the processes in the tree.  For example, the memory allocated by `php-cgi.exe` and `w3wp.exe` _both_ count towards the _same_ memory quota.  Spawning additional processes will _not_ allow you to consume more memory!

**TL;DR: Each Azure Web App (formerly known as site) has its own sandbox, consisting of _all_ the processing used in the app.**  

### App Service Plans and Sandbox Limits

In the context of the sandbox, the limits which apply may depend on the type of plan in which your application runs.  For example, some limits only apply to applications running in Free/Shared modes; these limits are lifted when running your application in Basic/Standard/Premium modes.  If the plan affects the limits, it will be called out.  

### General Sandbox Restrictions

The sandbox generally aims to restrict access to _shared_ components of Windows.  Unfortunately, many core components of Windows have been designed as shared components: the [registry](https://en.wikipedia.org/wiki/Windows_Registry), [cryptography](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service), and [graphics](https://en.wikipedia.org/wiki/Graphics_Device_Interface) subsystems, among others.  This section outlines the limitations placed on usage of these often essential, yet shared components.

#### Writing to registry

Apps may not write to any location in the registry regardless of any [permissions](https://en.wikipedia.org/wiki/Access_control_list) present on the key.  This blanket limitation is implemented as a global restriction without exceptions.

While writing is not allowed, the reading and enumeration of keys and values are permitted.  Access to keys and values for these purposes are governed by the ACLs present on the key.

**In short,  there is no hive or key which is writable, including `HKEY_CURRENT_USER`.  However, reading and enumeration of keys are permitted, subject to the permissions present on the key.**

#### Access to Event Log

While apps are not permitted to read from or write to the Event Log service _per se_, a virtualized event log is available which requires no change to app code to use.  Events written using the normal event logging APIs will be written to an XML file located at `%SYSTEMDRIVE%\home\logfiles\eventlog.xml`.  This virtualized event log has a rotating limit of 1000 events &mdash; events written past this limit still be written, but will cause the oldest events to be removed from the file. This virtualization is done to enable better debugging of apps as well as to support applications which require, as part of their basic function, the registration and use of event sources.  Technically, this virtualization is done in user-mode within the application process itself using [API hooking](http://research.microsoft.com/en-us/projects/detours/).  

#### Access to out-of-process COM servers

Windows Servers have a bunch of COM servers configured and available for consumption by   default; however the sandbox prevents access to all out-of-proc COM servers.  For example, a sandboxed application cannot call into WMI, or into MSIServer.  

#### Use of Console

Applications do not have access to their console.  Some console APIs are virtualized (faked) in user mode but you should consider console access a non-starter within sandbox.  

#### Win32k.sys (User32/GDI32) Restrictions

For the sake of radical attack surface area reduction, the sandbox prevents almost all of the Win32k.sys APIs from being called, which practically means that most of User32/GDI32 system calls are blocked.  For most applications this is not an issue since most Azure Web Apps do not require access to Windows UI functionality (they are web applications after all). 

However one common pattern that is affected is PDF file generation.  There are several cases of note:

##### Microsoft SQL Reporting Framework manually generating PDFs

This usage pattern does not work for Free/Shared plans due to the sandbox.  However, a small set of APIs are opened for use in Basic+ plans; these APIs are designed to enable SQL Reporting framework creation of PDFs.  So these libraries should be usable for PDF generation in Basic+.  

##### PDF generation from HTML

There are multiple libraries used to convert HTML to PDF.  Many Windows/.NET specific versions leverage IE APIs and therefore leverage User32/GDI32 extensively.  These APIs are largely blocked in the sandbox (regardless of plan) and therefore these frameworks do not work in the sandbox.  

There are some frameworks that do not leverage User32/GDI32 extensively (wkhtmltopdf, for example) and we are working on enabling these in Basic+ the same way we enabled SQL Reporting.

#### Process Enumeration/Job-Assignment

Processes within a sandbox are able to access each other via their handles and retrieve the handles using `OpenProcess()` the access is restricted to within the sandbox; a process cannot open/see other processes outside the sandbox.  

For example, if you use the System.Diagnostics.Process class to enumerate processes you will only see the processes within the sandbox.  

The sandbox itself runs in a Job controlled by the Azure Web App administrative service (the moral equivalent of WAS service in Windows).  A process within the sandbox can create a its own nested Job for child processes to occupy.  

#### Service Control Manager (SCM) Access

Applications cannot access SCM to manage/enumerate/start/stop services.  Since some legacy code does require access to SCM, these APIs are faked in user mode to keep the applications working; but these stubs are essentially not functional.  

### File System Restrictions/Considerations

Applications are highly restricted in terms of their access of the file system.  

#### Home directory access (d:\home)

Every Azure Web App has a home directory stored/backed by Azure Storage.  This network share is where applications store their content.  This directory is available for the sandbox with read/write access.  

As a convenience for our customers, the sandbox implements a dynamic symbolic link in kernel mode which maps `d:\home` to the customer home directory.  This is done to remove the need of the customer to keep referencing their own network share path when accessing the site.  No matter where the site runs, or how many sites run on a VM, each can access their home directory using `d:\home`.  

#### Local directory access (d:\local)

Every Azure Web App has a local directory which is temporary and is deleted when the run is no longer running on the VM.  This directory is a place to store temporary data for the application.  The sandbox implements a dynamic symbolic link which maps `d:\local` to point to this directory.  The application naturally has read/write access to this directory.

Note that the `d:\local` folder in the scm site (where Kudu runs) is not the same as the one in the main site (where the web app runs). As a result, they cannot see each other's local files.

#### System drive access

Applications can read the system drive as controlled by ACLs on the drive.  Applications cannot write to any location on the system drive, as implemented in the sandbox (i.e. even if ACLs allowed writing to a location on system drive, the sandbox would restrict it).  

#### Symbolic link creation

While sandboxed applications can follow/open existing symbolic links, they cannot create symbolic links (or any other reparse point) anywhere.  

### Networking Restrictions/Considerations

There are manifold restrictions in terms of network access from an Azure Web App.  This section outlines limitations _specific_ to Azure App Service; apps are, in addition, still subject to Azure's [own networking restrictions](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-faq/).

#### Network endpoint listening

The only way an application can be accessed via the internet is through the already-exposed HTTP (`80`) and HTTPS (`443`) TCP ports; applications may not listen on other ports for packets arriving from the internet.  
However, applications may create a socket which can listen for connections from within the sandbox.  For example, two processes within the same app may communicate with one another via TCP sockets; connection attempts incoming from outside the sandbox, albeit they be on the same machine, will fail.  See the next topic for additional detail.

#### Local Address Requests

Connection attempts to local addresses (_e.g._ `localhost`, `127.0.0.1`) and the machine's own IP will fail, _except_ if another process in the _same_ sandbox has created a listening socket on the destination port.  

Rejected connection attempts, such as the following example which attempts to connect to `127.0.0.1:80`, from .NET will result in the following exception:
```
Exception Details: System.Net.Sockets.SocketException: An attempt was made to access a socket in a way forbidden by its access permissions 127.0.0.1:80
```

#### Private Address Requests

Applications cannot connect to private IP address as defined by IANA (https://en.wikipedia.org/wiki/Private_network).  This definition is augmented to further prevent connections to 100.* IP addresses.  

#### Use of Raw Sockets

Raw sockets (created with socket type SOCK_RAW) are not available to applications.  A side effect of this is you cannot do SYN attacks from an Azure Web App.  

#### Restricted Outgoing Ports

Regardless of address, applications cannot connect to anywhere using ports 445, 137, 138, and 139. In other words, even if connecting to a non-private IP address or the address of a virtual network, connections to ports 445, 137, 138, and 139 are not permitted.

#### Named Pipes

Within the sandbox, applications can create named pipes which accessible only within the sandbox (much like how listen sockets are allowed within sandbox). 

### Virtual Networks
Azure Web Apps may set up their [_virtual networks_](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-overview), or _VNets_ in order to facilitate connectivity between Azure and on-premise intranets.  This special case of network connectivity is also handled differently in the sandbox.  In particular, the aforementioned restrictions on private and local addresses are ignored if the target network interface belongs to the app.  Other VNet adapters on the same machine cannot be accessed, and all other network limitations are still apply.

Additionally, the sandbox automatically affinitizes [`connect`](https://msdn.microsoft.com/en-us/library/windows/desktop/ms737625%28v=vs.85%29.aspx) and [`send`](https://msdn.microsoft.com/en-us/library/windows/desktop/ms740149%28v=vs.85%29.aspx) operations destined for VNet addresses to the correct VNet interface, in order to improve ease-of-use of the VNet feature.

### Numerical Sandbox Limits

Given the above background information, the following are numerical limits which apply to applications in the sandbox.  Note that "unlimited" still means memory/resource limits may apply; however the sandbox does not explicitly restrict.  

#### Per-Sandbox (per-app/per-site) Numerical Limits

These limits apply to each sandbox and therefore represent the sum usage of all the processes within the given sandbox.  

| Limit name | Description | Free/Shared Limit | Basic+ Limit |
| ---------- | ----------- | ----------------- | ------------ |
| Threads | Number of threads| 512 | Unlimited |
| Processes | Number of processes | 32 | Unlimited |
| Connections | Number of bound sockets outstanding | 250 | Unlimited |
| Named Pipes | Number of named pipes | 128 | 128 |
| Listen Sockets | Number of listen sockets | 256 | 256 |

#### Cross VM Numerical Limits

These limits apply only for customers of Basic or higher plans; in other words customers running on their own dedicated VMs.  These limits are there to protect the entire VM even though one particular site may be with its limits described above.  The limits are different depending on the size of VM configured.  

| Limit name | Description | Small (A1) | Medium (A2) | Large (A3)
| ---------- | ----------- | ---------- | ----------- | ----------
| Connections | Number of connections across entire VM | 1920 | 3968 | 8064

#### .NET Threadpool and ASP.NET Settings

The maxConcurrentRequestsPerCPU setting specifies how many simultaneous requests ASP.NET allows per CPU. This setting is different depending on the size of VM configured.

| Setting name |  Small (A1) | Medium (A2) | Large (A3)
| ---------- | ----------- | ---------- | ----------- 
| maxConcurrentRequestsPerCpu |  7500 | 7500 | 18750
| Number of Cores | 1 | 2 | 4
| Actual MaxConcurrentRequestsValue | 7500 | 15000 | 75000

These are the default values of some other .NET Threadpool settings

| Setting name |  Value
| ---------- | ----------- 
| maxWorkerThreads | 8191
| minWorkerThreads | 4
| maxIoThreads| 1000
| minIoThreads| 4

If you want to alter these settings, you can call ThreadPool.SetMinThreads and ThreadPool.SetMaxThreads functions in the Application_Start event of global.asax

### Unsupported frameworks

Here is a list of frameworks and scenarios that have been found to be not be usable due to one or more of the restrictions above. It's conceivable that some will be supported in the future as the sandbox evolves.

PDF generators failing due to restriction mentioned above:

- Syncfusion
- Siberix
- NReco (uses wkhtmltopdf)
- Spire.PDF

The following PDF generators *are* supported:

- SQL Reporting framework: requires the site to run in Basic or higher
- EVOPDF: See http://www.evopdf.com/azure-html-to-pdf-converter.aspx for vendor solution
- Telerik reporting: requires the site to run in Basic or higher. More info [here](http://www.telerik.com/forums/out-of-memory-in-azure-websites#yDA-ydAnBEOF0qKbHljCvw)
- Rotativa / wkhtmltopdf: requires the site to run in Basic or higher.


Other scenarios that are not supported:

- PhantomJS/Selenium: tries to connect to local address, and also uses GDI+.
