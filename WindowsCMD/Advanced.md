# Windows CMD Advanced Cheatsheet

## Advanced Batch Scripting

### Advanced Variable Manipulation
```cmd
@echo off
setlocal enabledelayedexpansion

REM Substring extraction
set str=HelloWorld
set sub=!str:~0,5!              & REM Hello
set sub=!str:~-5!               & REM World
set sub=!str:~5,5!              & REM World

REM String replacement
set text=Hello World
set new=!text:World=CMD!        & REM Hello CMD
set new=!text: =_!              & REM Hello_World

REM Remove substring
set path=C:\Users\Docs\file.txt
set filename=!path:C:\Users\Docs\=!    & REM file.txt

REM Extract filename and extension
for %%F in ("%path%") do (
    set filename=%%~nF          & REM file
    set extension=%%~xF         & REM .txt
    set fullname=%%~nxF         & REM file.txt
    set drive=%%~dF             & REM C:
    set folder=%%~pF            & REM \Users\Docs\
    set fullpath=%%~fF          & REM Full path
)

REM String length
set str=Hello
set len=0
:strlen_loop
if defined str (
    set str=!str:~1!
    set /a len+=1
    goto strlen_loop
)
echo Length: !len!
```

### Advanced Loops and Iteration
```cmd
@echo off
setlocal enabledelayedexpansion

REM Loop through command output
for /f "tokens=*" %%a in ('dir /b *.txt') do (
    echo Processing: %%a
)

REM Parse CSV data
for /f "tokens=1,2,3 delims=," %%a in (data.csv) do (
    echo Name: %%a, Age: %%b, City: %%c
)

REM Skip header line
for /f "skip=1 tokens=1,2 delims=," %%a in (data.csv) do (
    echo %%a - %%b
)

REM Process file contents line by line
for /f "usebackq delims=" %%a in ("file with spaces.txt") do (
    echo Line: %%a
)

REM Loop through registry keys
for /f "tokens=*" %%a in ('reg query HKCU\Software') do (
    echo Key: %%a
)

REM Nested loops
for /l %%i in (1,1,3) do (
    for /l %%j in (1,1,3) do (
        echo %%i,%%j
    )
)

REM Loop with counter
set count=0
for %%f in (*.txt) do (
    set /a count+=1
    echo !count!. %%f
)
```

### Advanced Functions
```cmd
@echo off
setlocal enabledelayedexpansion

REM Function with return value via variable
call :add 5 3 result
echo Sum: %result%
goto :end

:add
set /a %3=%1+%2
goto :eof

:end

REM Function with return via ERRORLEVEL
call :checkFile "important.txt"
if !errorlevel! EQU 0 (
    echo File exists
) else (
    echo File not found
)
goto :end2

:checkFile
if exist %1 (
    exit /b 0
) else (
    exit /b 1
)

:end2
```

### Error Handling and Logging
```cmd
@echo off
setlocal enabledelayedexpansion

REM Setup logging
REM Note: Date format is locale-dependent (US format assumed)
set LOGFILE=script_%date:~-4%-%date:~-10,2%-%date:~-7,2%.log
call :log "Script started"

REM Error handling function
call :executeWithLogging "copy file.txt backup\"
if !errorlevel! NEQ 0 (
    call :log "ERROR: Copy failed"
    goto :cleanup
)

call :log "Script completed successfully"
goto :end

:log
echo [%date% %time%] %~1 >> %LOGFILE%
echo [%date% %time%] %~1
goto :eof

:executeWithLogging
call :log "Executing: %~1"
%~1
set exitcode=!errorlevel!
if !exitcode! NEQ 0 (
    call :log "ERROR: Command failed with code !exitcode!"
)
exit /b !exitcode!

:cleanup
call :log "Performing cleanup"
REM Cleanup code here
exit /b 1

:end
call :log "Script ended"
endlocal
```

### Menu System
```cmd
@echo off
setlocal enabledelayedexpansion

:menu
cls
echo ========================================
echo         MAIN MENU
echo ========================================
echo 1. Backup Files
echo 2. Restore Files
echo 3. Clean Temp Files
echo 4. System Info
echo 5. Exit
echo ========================================
set /p choice="Enter your choice (1-5): "

if "%choice%"=="1" call :backup
if "%choice%"=="2" call :restore
if "%choice%"=="3" call :cleanup
if "%choice%"=="4" call :sysinfo
if "%choice%"=="5" goto :end

goto menu

:backup
echo Backing up files...
xcopy /s /y *.txt backup\
echo Backup completed
pause
goto :eof

:restore
echo Restoring files...
xcopy /s /y backup\*.txt .
echo Restore completed
pause
goto :eof

:cleanup
echo Cleaning temporary files...
del /q %temp%\*.*
echo Cleanup completed
pause
goto :eof

:sysinfo
systeminfo
pause
goto :eof

:end
echo Goodbye!
endlocal
exit /b 0
```

## Scheduled Tasks (SCHTASKS)

### Creating Scheduled Tasks
```cmd
REM Create daily task
schtasks /create /tn "DailyBackup" /tr "C:\Scripts\backup.bat" /sc daily /st 02:00

REM Create weekly task
schtasks /create /tn "WeeklyReport" /tr "C:\Scripts\report.bat" /sc weekly /d MON /st 09:00

REM Create task on system startup
schtasks /create /tn "StartupTask" /tr "C:\Scripts\startup.bat" /sc onstart

REM Create task on user logon
schtasks /create /tn "LogonTask" /tr "C:\Scripts\logon.bat" /sc onlogon

REM Create task with highest privileges
schtasks /create /tn "AdminTask" /tr "C:\Scripts\admin.bat" /sc daily /st 01:00 /rl highest

REM Create task with specific user
schtasks /create /tn "UserTask" /tr "C:\Scripts\task.bat" /sc daily /st 10:00 /ru "DOMAIN\Username" /rp "password"

REM Create task that runs every 30 minutes
schtasks /create /tn "FrequentTask" /tr "C:\Scripts\frequent.bat" /sc minute /mo 30

REM Create task on idle
schtasks /create /tn "IdleTask" /tr "C:\Scripts\idle.bat" /sc onidle /i 10
```

### Managing Scheduled Tasks
```cmd
REM List all scheduled tasks
schtasks /query

REM List with detailed format
schtasks /query /fo list /v

REM List specific task
schtasks /query /tn "DailyBackup" /v /fo list

REM Run task immediately
schtasks /run /tn "DailyBackup"

REM Stop running task
schtasks /end /tn "DailyBackup"

REM Delete task
schtasks /delete /tn "DailyBackup" /f

REM Change task schedule
schtasks /change /tn "DailyBackup" /st 03:00

REM Enable task
schtasks /change /tn "DailyBackup" /enable

REM Disable task
schtasks /change /tn "DailyBackup" /disable

REM Query task status
schtasks /query /tn "DailyBackup" /fo list | findstr Status
```

### Advanced Task Configuration
```cmd
REM Create task with multiple triggers
schtasks /create /tn "MultiTask" /tr "C:\Scripts\task.bat" /sc daily /st 09:00
schtasks /create /tn "MultiTask" /tr "C:\Scripts\task.bat" /sc weekly /d FRI /st 17:00

REM Create task with conditions
schtasks /create /tn "PowerTask" /tr "C:\Scripts\task.bat" /sc daily /st 02:00 ^
    /ru "SYSTEM" /rl highest ^
    /f

REM Export task to XML
schtasks /query /tn "DailyBackup" /xml > backup_task.xml

REM Create task from XML
schtasks /create /tn "ImportedTask" /xml backup_task.xml

REM Remote task management
schtasks /create /s RemotePC /u username /p password /tn "RemoteTask" /tr "C:\task.bat" /sc daily /st 01:00
```

## Registry Operations (REG)

### Query Registry
```cmd
REM Query registry key
reg query HKLM\Software\Microsoft\Windows\CurrentVersion

REM Query specific value
reg query HKLM\Software\Microsoft\Windows\CurrentVersion /v ProgramFilesDir

REM Query recursively
reg query HKCU\Software /s /f "myapp"

REM Query with pattern
reg query HKLM\Software /f "Microsoft" /k

REM Export registry key
reg export HKCU\Software\MyApp backup.reg
```

### Add/Modify Registry
```cmd
REM Add registry key
reg add HKCU\Software\MyApp

REM Add string value
reg add HKCU\Software\MyApp /v Setting1 /t REG_SZ /d "Value" /f

REM Add DWORD value
reg add HKCU\Software\MyApp /v Setting2 /t REG_DWORD /d 1 /f

REM Add binary value
reg add HKCU\Software\MyApp /v Setting3 /t REG_BINARY /d ffff /f

REM Add multi-string value
reg add HKCU\Software\MyApp /v Paths /t REG_MULTI_SZ /d "C:\Path1\0C:\Path2" /f

REM Modify existing value
reg add HKCU\Software\MyApp /v Setting1 /t REG_SZ /d "NewValue" /f
```

### Delete Registry
```cmd
REM Delete registry value
reg delete HKCU\Software\MyApp /v Setting1 /f

REM Delete entire key
reg delete HKCU\Software\MyApp /f

REM Delete recursively
reg delete HKCU\Software\MyApp /va /f
```

### Registry Best Practices
```cmd
REM Always backup before modifying
reg export HKCU\Software\MyApp backup_%date:~-4%%date:~-10,2%%date:~-7,2%.reg

REM Check if key exists before modifying
reg query HKCU\Software\MyApp >nul 2>&1
if !errorlevel! EQU 0 (
    echo Key exists
    reg add HKCU\Software\MyApp /v NewSetting /t REG_SZ /d "Value" /f
) else (
    echo Key does not exist
    reg add HKCU\Software\MyApp
    reg add HKCU\Software\MyApp /v NewSetting /t REG_SZ /d "Value" /f
)
```

## Service Management (SC)

### Query Services
```cmd
REM List all services
sc query

REM Query specific service
sc query wuauserv

REM Query service status
sc query wuauserv | findstr STATE

REM List all services with detailed info
sc query state= all

REM Query service configuration
sc qc wuauserv

REM Query service description
sc qdescription wuauserv

REM Query service failure actions
sc qfailure wuauserv
```

### Service Control
```cmd
REM Start service
sc start wuauserv

REM Stop service
sc stop wuauserv

REM Pause service
sc pause wuauserv

REM Continue paused service
sc continue wuauserv

REM Restart service (stop then start)
sc stop wuauserv && sc start wuauserv
```

### Service Configuration
```cmd
REM Create new service
sc create MyService binPath= "C:\Program Files\MyApp\service.exe" start= auto

REM Change service startup type
sc config wuauserv start= auto      & REM Automatic
sc config wuauserv start= demand    & REM Manual
sc config wuauserv start= disabled  & REM Disabled

REM Change service account
sc config MyService obj= "NT AUTHORITY\LocalService" password= ""

REM Set service description
sc description MyService "My Custom Service Description"

REM Set service failure actions
sc failure MyService reset= 86400 actions= restart/60000/restart/60000/restart/60000

REM Delete service
sc delete MyService
```

### Remote Service Management
```cmd
REM Query remote service
sc \\RemotePC query wuauserv

REM Start remote service
sc \\RemotePC start wuauserv

REM Configure remote service
sc \\RemotePC config wuauserv start= auto
```

## Disk Management

### DISKPART Commands
```cmd
REM Launch diskpart (requires admin)
diskpart

REM Within diskpart:
list disk              & REM List all disks
list volume            & REM List all volumes
list partition         & REM List partitions on selected disk

select disk 1          & REM Select disk by number
select volume C        & REM Select volume by letter

detail disk            & REM Show disk details
detail volume          & REM Show volume details

clean                  & REM Clean disk (removes all data)
create partition primary size=10000
format fs=ntfs quick label="Data"
assign letter=E

extend size=5000       & REM Extend volume
shrink desired=5000    & REM Shrink volume

active                 & REM Mark partition as active
```

### Disk Utilities
```cmd
REM Check disk for errors
chkdsk C: /f

REM Check and repair
chkdsk C: /f /r

REM Check disk without fixing
chkdsk C:

REM Format drive
format E: /fs:ntfs /q /v:DATA

REM Convert to NTFS
convert D: /fs:ntfs

REM Display disk usage
dir C:\ /s | find "bytes free"

REM Disk cleanup (requires admin)
cleanmgr /d C:

REM Defragment drive
defrag C: /u /v

REM Optimize SSD
defrag C: /o
```

### Volume Shadow Copy (VSS)
```cmd
REM List shadow copies
vssadmin list shadows

REM List shadow storage
vssadmin list shadowstorage

REM Create shadow copy
vssadmin create shadow /for=C:

REM Delete shadow copies
vssadmin delete shadows /for=C: /all

REM Resize shadow storage
vssadmin resize shadowstorage /for=C: /on=C: /maxsize=10GB
```

## Advanced Network Operations

### Network Configuration
```cmd
REM Display network adapters
netsh interface show interface

REM Display IP configuration
netsh interface ip show config

REM Set static IP
netsh interface ip set address "Local Area Connection" static 192.168.1.100 255.255.255.0 192.168.1.1

REM Set to DHCP
netsh interface ip set address "Local Area Connection" dhcp

REM Set DNS servers
netsh interface ip set dns "Local Area Connection" static 8.8.8.8 primary
netsh interface ip add dns "Local Area Connection" 8.8.4.4 index=2

REM Enable/disable adapter
netsh interface set interface "Local Area Connection" enabled
netsh interface set interface "Local Area Connection" disabled

REM Show wireless networks
netsh wlan show networks

REM Show wireless profiles
netsh wlan show profiles

REM Export wireless profile
netsh wlan export profile name="WiFiName" folder=C:\Backup

REM Import wireless profile
netsh wlan add profile filename="WiFiName.xml"
```

### Firewall Management
```cmd
REM Show firewall status
netsh advfirewall show allprofiles

REM Enable firewall
netsh advfirewall set allprofiles state on

REM Disable firewall (not recommended)
netsh advfirewall set allprofiles state off

REM Add firewall rule
netsh advfirewall firewall add rule name="Allow Port 80" dir=in action=allow protocol=TCP localport=80

REM Delete firewall rule
netsh advfirewall firewall delete rule name="Allow Port 80"

REM Block program
netsh advfirewall firewall add rule name="Block App" dir=out program="C:\App\app.exe" action=block

REM Show all firewall rules
netsh advfirewall firewall show rule name=all

REM Reset firewall to defaults
netsh advfirewall reset
```

### Advanced Network Diagnostics
```cmd
REM Route table
route print
route add 192.168.2.0 mask 255.255.255.0 192.168.1.1
route delete 192.168.2.0

REM Network statistics by protocol
netstat -s

REM Display routing table
netstat -r

REM Show network interfaces
netstat -e

REM Test network connectivity
pathping google.com

REM NBT statistics
nbtstat -a computername
nbtstat -c    & REM Cache contents

REM Network packet capture
netsh trace start capture=yes tracefile=C:\capture.etl
netsh trace stop
```

## Performance and Monitoring

### System Performance
```cmd
REM Performance counters
typeperf "\Processor(_Total)\% Processor Time" -sc 10

REM Monitor multiple counters
typeperf "\Processor(_Total)\% Processor Time" "\Memory\Available MBytes" -sc 10

REM Output to file
typeperf "\Processor(_Total)\% Processor Time" -o perf.csv -f CSV -sc 100

REM System performance report
perfmon /report

REM Create performance baseline
logman create counter PerfLog -c "\Processor(_Total)\% Processor Time" -f bin -o C:\PerfLog
logman start PerfLog
REM ... wait ...
logman stop PerfLog
```

### Event Log Management
```cmd
REM Query event logs
wevtutil qe System /c:10 /f:text

REM Query with filter
wevtutil qe System /q:"*[System[(Level=2)]]" /f:text /c:5

REM Export event log
wevtutil epl System C:\Logs\system.evtx

REM Clear event log
wevtutil cl Application

REM List all logs
wevtutil el

REM Get log information
wevtutil gli System

REM Archive log
wevtutil al C:\Logs\archived.evtx /l:en-US
```

### WMI Queries
```cmd
REM System information
wmic computersystem get manufacturer,model,name

REM BIOS information
wmic bios get serialnumber,version

REM CPU information
wmic cpu get name,numberofcores,maxclockspeed

REM Memory information
wmic memorychip get capacity,manufacturer,speed

REM Disk information
wmic diskdrive get model,size,status

REM Network adapter info
wmic nic get name,macaddress,speed

REM Installed software
wmic product get name,version

REM Running processes
wmic process get name,processid,executablepath

REM Services
wmic service get name,state,startmode

REM Logged on users
wmic computersystem get username

REM Startup programs
wmic startup get caption,command
```

## Troubleshooting and Diagnostics

### System File Checker
```cmd
REM Scan system files
sfc /scannow

REM Verify only (no repair)
sfc /verifyonly

REM Scan specific file
sfc /scanfile=C:\Windows\System32\kernel32.dll

REM Offline scan
sfc /scannow /offbootdir=C:\ /offwindir=D:\Windows
```

### DISM (Deployment Image Servicing)
```cmd
REM Check image health
dism /online /cleanup-image /checkhealth

REM Scan image health
dism /online /cleanup-image /scanhealth

REM Repair image
dism /online /cleanup-image /restorehealth

REM Cleanup component store
dism /online /cleanup-image /startcomponentcleanup

REM Remove superseded components
dism /online /cleanup-image /startcomponentcleanup /resetbase

REM Analyze component store
dism /online /cleanup-image /analyzecomponentstore
```

### Network Troubleshooting
```cmd
REM Reset TCP/IP stack
netsh int ip reset
netsh winsock reset

REM Reset firewall
netsh advfirewall reset

REM Release and renew IP
ipconfig /release
ipconfig /renew

REM Flush DNS cache
ipconfig /flushdns

REM Register DNS
ipconfig /registerdns

REM Reset network adapter
netsh interface set interface "Ethernet" disabled
netsh interface set interface "Ethernet" enabled
```

### Advanced Troubleshooting Script
```cmd
@echo off
setlocal enabledelayedexpansion

REM Create diagnostic report
set REPORTFILE=diagnostic_%computername%_%date:~-4%%date:~-10,2%%date:~-7,2%.txt

echo ========================================>> %REPORTFILE%
echo SYSTEM DIAGNOSTIC REPORT>> %REPORTFILE%
echo Generated: %date% %time%>> %REPORTFILE%
echo ========================================>> %REPORTFILE%
echo.>> %REPORTFILE%

echo Collecting system information...
systeminfo >> %REPORTFILE%
echo.>> %REPORTFILE%

echo Collecting network information...
ipconfig /all >> %REPORTFILE%
echo.>> %REPORTFILE%

echo Collecting active connections...
netstat -ano >> %REPORTFILE%
echo.>> %REPORTFILE%

echo Collecting running processes...
tasklist /v >> %REPORTFILE%
echo.>> %REPORTFILE%

echo Collecting running services...
sc query state= all >> %REPORTFILE%
echo.>> %REPORTFILE%

echo Collecting event log errors (last 24 hours)...
wevtutil qe System /q:"*[System[(Level=2) and TimeCreated[timediff(@SystemTime) <= 86400000]]]" /f:text /c:20 >> %REPORTFILE%
echo.>> %REPORTFILE%

echo Report saved to: %REPORTFILE%
echo.
echo Opening report...
notepad %REPORTFILE%

endlocal
```

## Best Practices (Do's)

✅ **Always backup before registry changes**
```cmd
@echo off
REM Backup before modification
set BACKUPKEY=HKCU\Software\MyApp
set BACKUPFILE=registry_backup_%date:~-4%%date:~-10,2%%date:~-7,2%.reg
reg export %BACKUPKEY% %BACKUPFILE%
echo Backup created: %BACKUPFILE%

REM Make changes
reg add %BACKUPKEY% /v Setting /t REG_SZ /d "Value" /f
```

✅ **Use error handling in production scripts**
```cmd
@echo off
setlocal enabledelayedexpansion

REM Execute with error handling
call :executeCommand "net stop MyService"
if !errorlevel! NEQ 0 goto :error

call :executeCommand "copy important.txt backup\"
if !errorlevel! NEQ 0 goto :error

echo All operations completed successfully
goto :end

:executeCommand
echo Executing: %~1
%~1
exit /b !errorlevel!

:error
echo ERROR: Operation failed
REM Rollback or cleanup
exit /b 1

:end
endlocal
exit /b 0
```

✅ **Document complex batch scripts**
```cmd
@echo off
REM ================================================
REM Script: advanced_maintenance.bat
REM Purpose: Perform system maintenance tasks
REM Author: IT Department
REM Version: 2.0
REM Last Modified: 2024-01-15
REM
REM Requirements:
REM   - Administrator privileges
REM   - Windows 10 or later
REM
REM Parameters:
REM   %1 - Operation mode (backup|clean|repair)
REM   %2 - Target drive (optional, default C:)
REM
REM Exit Codes:
REM   0 - Success
REM   1 - General error
REM   2 - Invalid parameters
REM   3 - Insufficient privileges
REM ================================================
```

✅ **Use SETLOCAL and ENDLOCAL**
```cmd
@echo off
setlocal enabledelayedexpansion

REM Variables are local to this script
set myvar=value

REM Do work

endlocal
REM myvar no longer exists outside this scope
```

✅ **Implement logging for important operations**
```cmd
@echo off
set LOGFILE=operations_%date:~-4%%date:~-10,2%%date:~-7,2%.log

call :log "INFO" "Script started"
call :log "INFO" "Performing backup"

copy *.txt backup\ >nul 2>&1
if %errorlevel% EQU 0 (
    call :log "SUCCESS" "Backup completed"
) else (
    call :log "ERROR" "Backup failed with code %errorlevel%"
)

goto :end

:log
echo [%date% %time%] [%~1] %~2 >> %LOGFILE%
echo [%date% %time%] [%~1] %~2
goto :eof

:end
call :log "INFO" "Script ended"
```

✅ **Test in safe environment first**
```cmd
REM Use echo to preview commands
set DRYRUN=1

if %DRYRUN%==1 (
    echo [DRY RUN] Would execute: del *.tmp
) else (
    del *.tmp
)
```

## Common Mistakes (Don'ts)

❌ **Don't modify system services without understanding**
```cmd
REM DANGEROUS - Don't disable critical services
sc config wuauserv start= disabled     & REM Windows Update (can cause issues)
sc config cryptsvc start= disabled     & REM Cryptographic Services (breaks system)
sc config wscsvc start= disabled       & REM Security Center (security risk)
```

❌ **Don't edit registry without backup**
```cmd
REM Wrong - no backup
reg delete HKLM\Software\Important /f

REM Correct - backup first
reg export HKLM\Software\Important backup.reg
reg delete HKLM\Software\Important /f
```

❌ **Don't use GOTO excessively in modern scripts**
```cmd
REM Bad - hard to maintain
:start
goto middle
:middle
goto end
:end

REM Better - use functions
call :initialize
call :process
call :cleanup
goto :end

:initialize
REM Init code
goto :eof

:process
REM Process code
goto :eof

:cleanup
REM Cleanup code
goto :eof

:end
```

❌ **Don't hardcode credentials**
```cmd
REM NEVER DO THIS
net use Z: \\server\share /user:admin Password123

REM Better - prompt for credentials
set /p username="Enter username: "
set /p password="Enter password: "
net use Z: \\server\share /user:%username% %password%

REM Even better - use saved credentials
net use Z: \\server\share /savecred
```

❌ **Don't ignore return codes**
```cmd
REM Wrong - no error checking
net stop MyService
reg delete HKCU\Software\MyApp /f
del C:\ImportantFile.txt

REM Correct - check each operation
net stop MyService
if %errorlevel% NEQ 0 echo Failed to stop service && exit /b 1

reg delete HKCU\Software\MyApp /f
if %errorlevel% NEQ 0 echo Failed to delete registry key && exit /b 1

del C:\ImportantFile.txt
if %errorlevel% NEQ 0 echo Failed to delete file && exit /b 1
```

❌ **Don't perform destructive operations without confirmation**
```cmd
REM Wrong - no confirmation
format E: /fs:ntfs /q

REM Correct - ask first
set /p confirm="Are you sure you want to format E:? (Y/N): "
if /i "%confirm%"=="Y" (
    format E: /fs:ntfs /q
) else (
    echo Operation cancelled
)
```

❌ **Don't leave debugging code in production**
```cmd
REM Remove before deployment
echo DEBUG: Variable value is %myvar%
pause
echo Press any key to continue with deletion...

REM Production version
REM Clean, no debug output
if exist %myvar% del %myvar%
```

❌ **Don't mix administrative and non-admin operations**
```cmd
REM Bad - requires admin but doesn't check
reg add HKLM\Software\MyApp /v Setting /t REG_SZ /d "Value" /f

REM Good - check for admin rights
net session >nul 2>&1
if %errorlevel% NEQ 0 (
    echo This script requires administrator privileges
    echo Right-click and select "Run as administrator"
    pause
    exit /b 1
)

REM Now safe to perform admin operations
reg add HKLM\Software\MyApp /v Setting /t REG_SZ /d "Value" /f
```

## Quick Reference

### Advanced Script Template
```cmd
@echo off
setlocal enabledelayedexpansion
title Advanced Script Template

REM ================================================
REM Script Configuration
REM ================================================
set VERSION=1.0
set LOGFILE=script_%date:~-4%%date:~-10,2%%date:~-7,2%.log
set ERRORLEVEL_SUCCESS=0
set ERRORLEVEL_ERROR=1

REM ================================================
REM Admin Check
REM ================================================
net session >nul 2>&1
if %errorlevel% NEQ 0 (
    echo ERROR: Administrator privileges required
    exit /b 3
)

REM ================================================
REM Main Logic
REM ================================================
call :log "INFO" "Script started - Version %VERSION%"

REM Your code here

call :log "INFO" "Script completed successfully"
goto :end

REM ================================================
REM Functions
REM ================================================

:log
echo [%date% %time%] [%~1] %~2 >> %LOGFILE%
echo [%date% %time%] [%~1] %~2
goto :eof

:error
call :log "ERROR" "%~1"
exit /b %ERRORLEVEL_ERROR%

REM ================================================
REM End
REM ================================================
:end
endlocal
exit /b %ERRORLEVEL_SUCCESS%
```

### Essential Advanced Commands
```cmd
schtasks /create /tn "Task" /tr "script.bat" /sc daily /st 02:00
reg query HKLM\Software
sc query wuauserv
diskpart
netsh interface ip show config
wmic computersystem get name
sfc /scannow
dism /online /cleanup-image /restorehealth
```

### Performance and Diagnostics
```cmd
typeperf "\Processor(_Total)\% Processor Time" -sc 10
perfmon /report
wevtutil qe System /c:10 /f:text
systeminfo | findstr /i "memory"
```
