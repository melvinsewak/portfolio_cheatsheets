# Windows CMD Intermediate Cheatsheet

## Batch Scripting Basics

### Creating Batch Files
```cmd
REM Create a simple batch file
REM Save as script.bat

@echo off
echo This is a batch script
pause
```

### Batch File Structure
```cmd
@echo off
REM This is a comment
:: This is also a comment

title My Batch Script
echo Starting script...

REM Your commands here
dir C:\

echo Script complete!
pause
exit /b 0
```

### Variables in Batch
```cmd
@echo off

REM Set variables
set name=John
set age=30
set path_to_file=C:\Documents

REM Display variables
echo Name: %name%
echo Age: %age%

REM User input
set /p username="Enter your name: "
echo Hello, %username%!

REM Arithmetic operations
set /a result=10+5
set /a result=10*2
set /a result=20/4
echo Result: %result%
```

### Conditional Statements
```cmd
@echo off

REM IF statement
if exist file.txt (
    echo File exists
) else (
    echo File does not exist
)

REM Compare strings
set name=John
if "%name%"=="John" (
    echo Name is John
)

REM Compare numbers
set /a num=10
if %num% GTR 5 (
    echo Number is greater than 5
)

REM NOT condition
if not exist backup mkdir backup

REM ELSE IF equivalent
if %num% EQU 10 (
    echo Number is 10
) else if %num% GTR 10 (
    echo Number is greater than 10
) else (
    echo Number is less than 10
)
```

### Comparison Operators
```cmd
REM Numeric comparisons
REM EQU - Equal to
REM NEQ - Not equal to
REM LSS - Less than
REM LEQ - Less than or equal to
REM GTR - Greater than
REM GEQ - Greater than or equal to

REM Example
if %value% GEQ 100 echo Value is 100 or more
```

### Loops
```cmd
@echo off

REM FOR loop - iterate over files
for %%f in (*.txt) do (
    echo Processing %%f
)

REM FOR loop - iterate over numbers
for /l %%i in (1,1,10) do (
    echo Number %%i
)

REM FOR loop - iterate over directories
for /d %%d in (*) do (
    echo Directory: %%d
)

REM FOR loop - recursive file search
for /r C:\Logs %%f in (*.log) do (
    echo Found: %%f
)

REM GOTO loop (while-like)
:loop
set /a counter+=1
echo Counter: %counter%
if %counter% LSS 5 goto loop
```

### Functions (CALL and Labels)
```cmd
@echo off

call :function1
call :function2 "Parameter1" "Parameter2"
goto :end

:function1
echo This is function 1
goto :eof

:function2
echo Function 2 received: %~1 and %~2
goto :eof

:end
echo Script finished
```

## Networking Commands

### Network Testing
```cmd
REM Ping host
ping google.com

REM Ping with specific count
ping -n 10 google.com

REM Continuous ping
ping -t google.com

REM Ping with timeout
ping -w 5000 google.com

REM Ping specific packet size
ping -l 1000 google.com

REM Trace route to host
tracert google.com

REM Test if port is open (via telnet)
telnet example.com 80
```

### IP Configuration
```cmd
REM Display network configuration
ipconfig

REM Display detailed configuration
ipconfig /all

REM Display DNS cache
ipconfig /displaydns

REM Flush DNS cache
ipconfig /flushdns

REM Release IP address
ipconfig /release

REM Renew IP address
ipconfig /renew

REM Display specific adapter
ipconfig /all | findstr /i "IPv4 DNS"
```

### Network Statistics
```cmd
REM Display active connections
netstat

REM Display with addresses and port numbers
netstat -n

REM Display all connections and listening ports
netstat -a

REM Display with executable names
netstat -b

REM Display routing table
netstat -r

REM Display statistics
netstat -e
netstat -s

REM Continuous display with interval
netstat -a 5

REM Find specific connection
netstat -an | findstr :80
netstat -an | findstr ESTABLISHED
```

### Network Share Commands
```cmd
REM Display shared resources
net share

REM Map network drive
net use Z: \\server\share

REM Map with credentials
net use Z: \\server\share /user:domain\username password

REM Delete mapped drive
net use Z: /delete

REM View all network connections
net use

REM Access network share directly
pushd \\server\share
REM Do work
popd
```

### ARP and Network Info
```cmd
REM Display ARP cache
arp -a

REM Display specific interface ARP
arp -a -N 192.168.1.1

REM Display hostname
hostname

REM Query DNS
nslookup google.com
nslookup google.com 8.8.8.8    # Use specific DNS server

REM Get network path info
pathping google.com
```

## Process Management

### List Processes
```cmd
REM List all running processes
tasklist

REM List with detailed info
tasklist /v

REM List specific process
tasklist | findstr explorer.exe

REM List by memory usage
tasklist /fo table /fi "memusage gt 100000"

REM List processes by user
tasklist /fi "username eq %username%"

REM List processes with services
tasklist /svc

REM List by status
tasklist /fi "status eq running"
```

### Kill Processes
```cmd
REM Kill process by name
taskkill /im notepad.exe

REM Kill process by PID
taskkill /pid 1234

REM Force kill process
taskkill /f /im chrome.exe

REM Kill process tree
taskkill /f /t /im process.exe

REM Kill multiple processes
taskkill /f /im app1.exe /im app2.exe

REM Kill with filter
taskkill /fi "status eq not responding" /f

REM Kill remote process
taskkill /s computername /u username /p password /im notepad.exe
```

### System Commands
```cmd
REM Shutdown computer
shutdown /s

REM Shutdown with timer (seconds)
shutdown /s /t 60

REM Restart computer
shutdown /r /t 0

REM Log off current user
shutdown /l

REM Hibernate
shutdown /h

REM Cancel scheduled shutdown
shutdown /a

REM Display shutdown GUI
shutdown /i
```

## File Search

### Find Command
```cmd
REM Search for text in file
find "search text" filename.txt

REM Case-insensitive search
find /i "search text" filename.txt

REM Display line numbers
find /n "error" logfile.txt

REM Count occurrences
find /c "error" logfile.txt

REM Display lines that don't match
find /v "success" logfile.txt

REM Search multiple files
find "TODO" *.txt

REM Search with pipe
dir /b | find "report"
```

### Findstr Command (Advanced)
```cmd
REM Basic search
findstr "pattern" filename.txt

REM Case-insensitive search
findstr /i "pattern" filename.txt

REM Search multiple files
findstr "error" *.log

REM Regular expression search
findstr /r "^Error.*" logfile.txt

REM Search recursively in subdirectories
findstr /s "TODO" *.cs

REM Display line numbers
findstr /n "pattern" file.txt

REM Multiple search patterns
findstr /c:"error" /c:"warning" logfile.txt

REM Search from file list
findstr /g:searchlist.txt *.txt

REM Exact match only
findstr /x "exact line" file.txt

REM Search with wildcards
findstr /l "file.*txt" directory_list.txt
```

### Where Command
```cmd
REM Find executable in PATH
where notepad.exe

REM Find multiple files
where *.dll

REM Search in specific directory
where /r C:\Windows *.exe

REM Search with multiple patterns
where $PATH:*.exe

REM Quiet mode (just return code)
where /q cmd.exe
```

## Environment Variables

### Setting Variables
```cmd
REM Set temporary variable (session only)
set myvar=value
set path=%path%;C:\NewPath

REM Set permanent user variable
setx myvar "value"

REM Set permanent system variable (requires admin)
setx myvar "value" /m

REM Remove variable
set myvar=

REM Display all variables
set

REM Display specific variable
set path
echo %path%
```

### Working with Variables
```cmd
@echo off

REM String manipulation
set str=Hello World
echo %str%                    # Hello World
echo %str:~0,5%              # Hello (substring)
echo %str:~-5%               # World (last 5 chars)
echo %str:Hello=Hi%          # Hi World (replace)

REM Path manipulation
set filepath=C:\Users\Docs\file.txt
echo %filepath:~0,2%         # C: (drive)
echo %filepath:~3%           # Users\Docs\file.txt

REM Remove quotes
set "quoted=This has quotes"
set unquoted=%quoted:"=%

REM Array-like behavior
set arr[0]=First
set arr[1]=Second
set arr[2]=Third
echo %arr[1]%                # Second
```

### Common Environment Variables
```cmd
REM System paths
%SYSTEMROOT%         # C:\Windows
%SYSTEMDRIVE%        # C:
%PROGRAMFILES%       # C:\Program Files
%PROGRAMFILES(X86)%  # C:\Program Files (x86)
%WINDIR%             # C:\Windows

REM User paths
%USERPROFILE%        # C:\Users\Username
%APPDATA%            # Application Data
%LOCALAPPDATA%       # Local Application Data
%TEMP% or %TMP%      # Temp directory
%HOMEPATH%           # \Users\Username

REM Session info
%USERNAME%           # Current username
%COMPUTERNAME%       # Computer name
%USERDOMAIN%         # Domain name
%DATE%               # Current date
%TIME%               # Current time
%RANDOM%             # Random number
%CD%                 # Current directory
%CMDCMDLINE%         # Command line
%ERRORLEVEL%         # Last error code
```

## Redirection and Pipes

### Output Redirection
```cmd
REM Redirect output to file (overwrite)
dir > output.txt

REM Append to file
dir >> output.txt

REM Redirect stderr to file
dir nonexistent 2> error.txt

REM Redirect both stdout and stderr
dir > output.txt 2>&1

REM Redirect to NUL (discard output)
dir > nul

REM Redirect stderr to NUL
dir 2> nul

REM Redirect both to NUL
dir > nul 2>&1
```

### Input Redirection
```cmd
REM Read input from file
sort < input.txt

REM Combine input and output redirection
sort < unsorted.txt > sorted.txt

REM Use file as command input
findstr "pattern" < input.txt
```

### Pipes
```cmd
REM Pipe output to another command
dir | more
dir | find "txt"
dir | sort

REM Multiple pipes
dir /s | findstr /i "log" | more

REM Combine with redirection
dir | find "txt" > textfiles.txt

REM Process and filter
systeminfo | findstr /i "total physical memory"
netstat -an | findstr "ESTABLISHED"
tasklist | findstr /i "chrome"
```

### Advanced Redirection
```cmd
REM Tee-like behavior (display and save)
dir | find "txt" | (echo. && type con && echo.) > output.txt

REM Multiple commands with redirection
(echo Line 1 && echo Line 2 && echo Line 3) > output.txt

REM Append multiple command outputs
echo First >> log.txt
echo Second >> log.txt

REM Conditional execution with pipes
dir | findstr "file.txt" && echo Found || echo Not found
```

## Command Chaining

### Sequential Execution
```cmd
REM Run commands in sequence (always)
command1 & command2 & command3

REM Example
cd C:\Projects & dir & echo Done

REM Each runs regardless of success
mkdir newfolder & cd newfolder & echo Created and moved
```

### Conditional Execution
```cmd
REM Run second only if first succeeds
command1 && command2

REM Example - create and enter directory
mkdir newfolder && cd newfolder

REM Run second only if first fails
command1 || command2

REM Example - create if not exists
exist file.txt || echo File not found

REM Combine conditional operators
mkdir backup && copy *.txt backup\ || echo Failed to backup
```

### Grouping Commands
```cmd
REM Group with parentheses
(command1 & command2 & command3)

REM Conditional groups
(mkdir logs && cd logs && echo Log directory ready) || echo Setup failed

REM Redirect grouped output
(echo Line 1 && echo Line 2 && echo Line 3) > output.txt
```

## File Attributes and Properties

### View and Modify Attributes
```cmd
REM Display file attributes
attrib filename.txt

REM Set read-only
attrib +r filename.txt

REM Remove read-only
attrib -r filename.txt

REM Set hidden
attrib +h filename.txt

REM Set system file
attrib +s filename.txt

REM Set archive attribute
attrib +a filename.txt

REM Combine attributes
attrib +r +h filename.txt

REM Apply to directory and contents
attrib +r /s /d C:\ImportantFolder\*.*
```

### File Comparison
```cmd
REM Compare two files
fc file1.txt file2.txt

REM Binary comparison
fc /b file1.exe file2.exe

REM Case-insensitive comparison
fc /c file1.txt file2.txt

REM Show only line count differences
fc /n file1.txt file2.txt

REM Compare with specific tab size
fc /t file1.txt file2.txt
```

## Best Practices (Do's)

✅ **Use @echo off in batch scripts**
```cmd
@echo off
REM Cleaner output without showing commands
echo Starting process...
```

✅ **Quote variables in conditionals**
```cmd
REM Good - prevents errors with empty variables
if "%username%"=="admin" echo Admin user

REM Bad - fails if variable is empty
if %username%==admin echo Admin user
```

✅ **Use ERRORLEVEL for error handling**
```cmd
@echo off
copy file.txt backup\
if %errorlevel% NEQ 0 (
    echo Copy failed!
    exit /b 1
)
echo Copy successful
```

✅ **Use delayed expansion for variables in loops**
```cmd
@echo off
setlocal enabledelayedexpansion

set count=0
for %%f in (*.txt) do (
    set /a count+=1
    echo File !count!: %%f
)
```

✅ **Use CALL for nested batch files**
```cmd
REM Good - returns after completion
call other_script.bat

REM Bad - transfers control, doesn't return
other_script.bat
```

✅ **Document your batch scripts**
```cmd
@echo off
REM Script: backup.bat
REM Purpose: Backup important files
REM Author: Your Name
REM Date: 2024-01-01

echo Starting backup...
```

✅ **Use /? for command help**
```cmd
REM Always check available options
findstr /?
for /?
if /?
```

## Common Mistakes (Don'ts)

❌ **Don't forget @echo off in batch files**
```cmd
REM Without @echo off - messy output showing all commands
echo Starting...
dir
copy file.txt backup\

REM With @echo off - clean output
@echo off
echo Starting...
dir
copy file.txt backup\
```

❌ **Don't use single % in batch files for variables**
```cmd
REM Wrong in .bat file
for %f in (*.txt) do echo %f

REM Correct in .bat file
for %%f in (*.txt) do echo %%f

REM Note: Single % works in command line
for %f in (*.txt) do echo %f
```

❌ **Don't forget to handle spaces in paths**
```cmd
REM Wrong
if exist C:\Program Files\app echo Found

REM Correct
if exist "C:\Program Files\app" echo Found
```

❌ **Don't use outdated network commands**
```cmd
REM Deprecated
nbtstat

REM Better alternatives
Get-NetTCPConnection (PowerShell)
netstat -an
```

❌ **Don't forget to enable delayed expansion when needed**
```cmd
REM Wrong - counter won't increment properly
set count=0
for %%f in (*.txt) do (
    set /a count+=1
    echo %count%    # Always shows 0
)

REM Correct
setlocal enabledelayedexpansion
set count=0
for %%f in (*.txt) do (
    set /a count+=1
    echo !count!    # Shows incremented value
)
```

❌ **Don't kill critical system processes**
```cmd
REM DON'T DO THIS
taskkill /f /im explorer.exe    # Kills Windows Explorer
taskkill /f /im csrss.exe       # Kills critical system process
taskkill /f /im winlogon.exe    # Kills logon process
```

❌ **Don't perform network operations without error checking**
```cmd
REM Wrong
net use Z: \\server\share
copy *.txt Z:\

REM Correct
net use Z: \\server\share
if %errorlevel% EQU 0 (
    copy *.txt Z:\
) else (
    echo Failed to connect to network share
)
```

❌ **Don't use GOTO excessively**
```cmd
REM Bad - spaghetti code
:start
echo Enter choice
goto process

:process
goto validate

:validate
goto end

REM Better - use CALL for structure
call :getInput
call :process
call :validate
goto :end
```

## Quick Reference

### Batch Script Template
```cmd
@echo off
setlocal enabledelayedexpansion
title My Script

REM Script header
echo ========================================
echo Script Name
echo ========================================
echo.

REM Main logic here

:end
echo.
echo Script completed
pause
endlocal
exit /b 0
```

### Network Quick Commands
```cmd
ipconfig /all              # Full network config
ipconfig /flushdns         # Clear DNS cache
netstat -an                # All connections
ping -t hostname           # Continuous ping
tracert hostname           # Trace route
nslookup hostname          # DNS lookup
```

### Process Quick Commands
```cmd
tasklist                   # List processes
tasklist /v                # Verbose process list
taskkill /f /im name.exe   # Force kill process
taskkill /pid 1234 /f      # Kill by PID
```

### Search Quick Commands
```cmd
find "text" file.txt       # Find text in file
findstr /s /i "text" *.*   # Recursive case-insensitive
where filename.exe         # Find executable
dir /s filename.txt        # Find file recursively
```
