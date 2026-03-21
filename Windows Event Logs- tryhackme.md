Windows Event Logs — TryHackMe Writeup
Hi hello everyone! 👋 Hope you all are doing good! So let's start with today's learning. I completed a few tasks from the Windows Event Logs room on TryHackMe, and here's everything I studied — written from memory as part of my daily writeup habit. Let's get into it! 🚀

Task 1 — What Are Windows Event Logs?

Event logs are records stored by the Windows operating system that capture everything happening inside your system. This includes hardware activity, software behavior, and application events. Whenever something significant occurs — like a user logging in, a USB drive being inserted, or a service crashing — Windows records it as a log entry with an Event ID, along with the date and time it occurred.
Event ID is simply a unique number assigned to each type of event. It makes it easy to search, filter, and identify specific events quickly without scrolling through thousands of entries.

Types of Logs:

Custom Logs — User-defined logs that capture diagnostic or operational data from specific applications or systems.
Directory Logs — Related to Active Directory (a Windows service that manages users, computers, and permissions in a network).
File Logs — Capture file-related activity on the system.

Event Types:
Success Audit: A security access attempt was successful.

Failure Audit: A security access attempt failed (opposite of Success Audit).

Error: Indicates a loss of data or loss of functionality.

Warning: Not immediately critical, but may cause problems in the future.

Information: General info about what an event did.

Viewing Logs — Event Viewer (GUI):

Event Viewer is a built-in graphical interface in Windows for reading logs. It has three main panels:

Left Panel — Navigation tree: Custom Views, Windows Logs (Application, Security, Setup, System, Forwarded Events).
Middle Panel — Displays log entries with Date/Time, Source, Event ID, and Level.
Right Panel — Action options like Clear Log, Filter Current Log, and Properties.

💡 You can easily search for specific events using the "Filter Current Log" option on the right panel.

Task 2 — wevtutil.exe (CLI Log Viewer)
wevtutil.exe is a command-line tool that lets you work with Windows event logs directly from PowerShell or Command Prompt — no GUI needed, just pure text commands.
To see all available usage options:
wevtutil.exe /?
Common Commands:
CommandFull FormDescriptionelenum-logsLists all available logs on the systemglget-logGets configuration information for a specific log
Example — List all logs:
wevtutil el
This prints every log name available on the system, one per line. Simple and fast! ⚡

Task 3 — Get-WinEvent (PowerShell Cmdlet)
Get-WinEvent is a powerful PowerShell cmdlet used to query and filter event logs in great detail. Think of it as a smarter, more flexible version of wevtutil.

List All Logs:
powershellGet-WinEvent -ListLog *
Lists all logs on the system. 
Note: it does not show a total count of logs.

List All Providers:
powershellGet-WinEvent -ListProvider *
Displays logs in a structured format showing:

Name — The provider (the source/application that generated the log)
LogLinks — The log file it writes to

Filter Logs by Provider (Most Important! 🔥):
powershellGet-WinEvent -LogName Application | Where-Object { $_.ProviderName -Match 'WLMS' }

This filters the Application log to show only events from the WLMS provider. You can replace 'WLMS' with any provider name you want to investigate — super useful during threat hunting!
For advanced filtering using FilterHashtable, check the official Microsoft docs:
🔗 Creating Get-WinEvent Queries with FilterHashtable

Final Thoughts 💭
Windows Event Logs are one of the most important tools in a SOC analyst's or blue teamer's arsenal. Whether you're investigating a security incident or just monitoring system health — knowing how to read, filter, and query logs using Event Viewer, wevtutil, and Get-WinEvent is a foundational skill.
That's it for today! If you found this helpful, follow me for more TryHackMe writeups. Keep learning, keep hacking! 🧠🔐

Room: Windows Event Logs | Platform: TryHackMe
Written from memory after completing the tasks — part of my daily writeup habit ✍️
