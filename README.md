# DLL-Proxy-Hijacking-Microsoft-Teams
Tutorial covering how to discover DLLs for Hijacking and how to create proxy DLLS using Microsoft Teams as an example

For a better version with pictures view the blog post:   

https://milosilo.com/hacking/microsoft-teams-proxy-dll-hijacking/


DLL Hijacking using a proxy dll file allows a malicious dll to be ran in a location an application incorrectly attempts to load a dll from, while forwarding all
legitimate commands to the intended dll. This allows malicious code to run without disrupting the executions of the targeted app. This tutorial will cover
how to detect vulnerable DLL file locations used by applications and how to create a proxy dll file that will open up calc.exe. This can be used as a form of
persistence, but also privilege escalation in certain circumstances.

What is a DLL File?
A dynamic-link library (DLL) is a module that contains functions and data that can be used by another module (application or DLL).
A DLL can define two kinds of functions: exported and internal. The exported functions are intended to be called by other modules, as well as from within
the DLL where they are defined. Internal functions are typically intended to be called only from within the DLL where they are defined. Although a DLL can
export data, its data is generally used only by its functions. However, there is nothing to prevent another module from reading or writing that address.
DLLs provide a way to modularize applications so that their functionality can be updated and reused more easily. DLLs also help reduce memory overhead
when several applications use the same functionality at the same time, because although each application receives its own copy of the DLL data, the
applications share the DLL code.
The Windows application programming interface (API) is implemented as a set of DLLs, so any process that uses the Windows API uses dynamic linking.

Why this works:
The search order for dlls in Windows follows a set of rules for searching for dlls when they are requested by the application. If the dll cannot be located
using the pre-search methods, it defaults to the standard search order until it finds the dll. This can create situations where an application tries to load a
DLL from the directory the application resides in before loading the dll from its correct location. When this situation is present, it is easy to place a
malicious dll file that executes a payload while directing all legitimate requests to the intended DLL file.

Required Items:
SysInternals AccessEnum
Mingw Compiler, Release: 8.1.0 x86_64-win32-seh
DLL Template - Written in c

Instructions:
Identify Targetable dlls:
Run AccessEnum on c:\ drive and save the results. (This will take some time. Go get a snack or take the dog for a walk)
Open the text file using Excel, insert a table, and filter on your current username in the "write" column
Review locations for familiarity. You will see patterns of areas you have write access to.

Identify Target DLLs:
Run a ProcMon capture while opening a targeted application. For this example, Teams.exe will be targeted.
Stop the capture once the application has finished starting.
Save the results as a csv so the results are backed up.
Open the Filter menu and select "Filter"
Add Filters:
"Process Name" is <targeted app executable file>(Teams.exe used in example).
"Result" contains "NAMENOTFOUND".
"Path" ends with "dll".
The results will list all available dlls to potential hijacks.
Check the dll location under the "Path" column with AccessEnum results to make sure it is writeable.
Once a dll file is identified you can proceed to create the dll file used to proxy the targeted dll file.
In this example the usp10.dll is not found in "%USERPROFILE%\AppData\Local\Microsoft\Teams\current".

Create DLL File:
Remove the filter for "NAME NOT FOUND" so the location to the targeted dll can be identified.
Copy the targeted dll file out of its location to a working folder.
Run gendef.exe on the dll file to create a DLL definition file:
"C:\temp\mingw64\bin\gendef.exe c:\temp\USP10.dll"
Sometimes gendef.exe fails to create the definition file. In this case use the python script from "DLL Hijack by Proxying" post listed in the
resources section below, and add in the missing library information to resolve incompatibility issues with the script. If you compare the
output with the definition sample file you will see the missing library information at the top.
Definition File Example:
Add desired payload to the attached DLL template where "calc.exe" is located.
Run x86_64-w64-mingw32-gcc.exe to create proxy DLL File:
"C:\temp\mingw64\bin\x86_64-w64-mingw32-gcc.exe -shared -o c:\temp\proxy\USP10.dll c:\temp\template.c c:\temp\USP10.def -s"

Execute:
Place the newly created DLL file in the targeted.dll location of where the NAMENOTFOUND was generated.
"%USERPROFILE%\AppData\Local\Microsoft\Teams\current" used for example.
The next time the application is loaded it will now load the proxy dll and execute the payload, while forwarding all the legitimate commands to the
original dll.

References:
DLL Hijack by Proxying - Python script does not work for creating reference file.  
https://reposhub.com/cpp/miscellaneous/tothi-dll-hijack-by-proxying.html
DLL Search Order
https://docs.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order
Mingw Compiler, Release: 8.1.0 x86_64-win32-seh
https://sourceforge.net/projects/mingw-w64/files/
SysInternals AccessEnum:
https://docs.microsoft.com/en-us/sysinternals/downloads/accessenum
