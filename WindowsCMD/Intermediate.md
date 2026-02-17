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
# Ping host
ping google.com

# Ping with specific count
ping -n 10 google.com

# Continuous ping
ping -t google.com

# Ping with timeout
ping -w 5000 google.com

# Ping specific packet size
ping -l 1000 google.com

# Trace route to host
tracert google.com

# Test if port is open (via telnet)
telnet example.com 80
```

### IP Configuration
```cmd
# Display network configuration
ipconfig

# Display detailed configuration
ipconfig /all

# Display DNS cache
ipconfig /displaydns

# Flush DNS cache
ipconfig /flushdns

# Release IP address
ipconfig /release

# Renew IP address
ipconfig /renew

# Display specific adapter
ipconfig /all | findstr /i "IPv4 DNS"
```

### Network Statistics
```cmd
# Display active connections
netstat

# Display with addresses and port numbers
netstat -n

# Display all connections and listening ports
netstat -a

# Display with executable names
netstat -b

# Display routing table
netstat -r

# Display statistics
netstat -e
netstat -s

# Continuous display with interval
netstat -a 5

# Find specific connection
netstat -an | findstr :80
netstat -an | findstr ESTABLISHED
```

### Network Share Commands
```cmd
# Display shared resources
net share

# Map network drive
net use Z: \\server\share

# Map with credentials
net use Z: \\server\share /user:domain\username password

# Delete mapped drive
net use Z: /delete

# View all network connections
net use

# Access network share directly
pushd \\server\share
REM Do work
popd
```

### ARP and Network Info
```cmd
# Display ARP cache
arp -a

# Display specific interface ARP
arp -a -N 192.168.1.1

# Display hostname
hostname

# Query DNS
nslookup google.com
nslookup google.com 8.8.8.8    # Use specific DNS server

# Get network path info
pathping google.com
```

## Process Management

### List Processes
```cmd
# List all running processes
tasklist

# List with detailed info
tasklist /v

# List specific process
tasklist | findstr explorer.exe

# List by memory usage
tasklist /fo table /fi "memusage gt 100000"

# List processes by user
tasklist /fi "username eq %username%"

# List processes with services
tasklist /svc

# List by status
tasklist /fi "status eq running"
```

### Kill Processes
```cmd
# Kill process by name
taskkill /im notepad.exe

# Kill process by PID
taskkill /pid 1234

# Force kill process
taskkill /f /im chrome.exe

# Kill process tree
taskkill /f /t /im process.exe

# Kill multiple processes
taskkill /f /im app1.exe /im app2.exe

# Kill with filter
taskkill /fi "status eq not responding" /f

# Kill remote process
taskkill /s computername /u username /p password /im notepad.exe
```

### System Commands
```cmd
# Shutdown computer
shutdown /s

# Shutdown with timer (seconds)
shutdown /s /t 60

# Restart computer
shutdown /r /t 0

# Log off current user
shutdown /l

# Hibernate
shutdown /h

# Cancel scheduled shutdown
shutdown /a

# Display shutdown GUI
shutdown /i
```

## File Search

### Find Command
```cmd
# Search for text in file
find "search text" filename.txt

# Case-insensitive search
find /i "search text" filename.txt

# Display line numbers
find /n "error" logfile.txt

# Count occurrences
find /c "error" logfile.txt

# Display lines that don't match
find /v "success" logfile.txt

# Search multiple files
find "TODO" *.txt

# Search with pipe
dir /b | find "report"
```

### Findstr Command (Advanced)
```cmd
# Basic search
findstr "pattern" filename.txt

# Case-insensitive search
findstr /i "pattern" filename.txt

# Search multiple files
findstr "error" *.log

# Regular expression search
findstr /r "^Error.*" logfile.txt

# Search recursively in subdirectories
findstr /s "TODO" *.cs

# Display line numbers
findstr /n "pattern" file.txt

# Multiple search patterns
findstr /c:"error" /c:"warning" logfile.txt

# Search from file list
findstr /g:searchlist.txt *.txt

# Exact match only
findstr /x "exact line" file.txt

# Search with wildcards
findstr /l "file.*txt" directory_list.txt
```

### Where Command
```cmd
# Find executable in PATH
where notepad.exe

# Find multiple files
where *.dll

# Search in specific directory
where /r C:\Windows *.exe

# Search with multiple patterns
where $PATH:*.exe

# Quiet mode (just return code)
where /q cmd.exe
```

## Environment Variables

### Setting Variables
```cmd
# Set temporary variable (session only)
set myvar=value
set path=%path%;C:\NewPath

# Set permanent user variable
setx myvar "value"

# Set permanent system variable (requires admin)
setx myvar "value" /m

# Remove variable
set myvar=

# Display all variables
set

# Display specific variable
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
# System paths
%SYSTEMROOT%         # C:\Windows
%SYSTEMDRIVE%        # C:
%PROGRAMFILES%       # C:\Program Files
%PROGRAMFILES(X86)%  # C:\Program Files (x86)
%WINDIR%             # C:\Windows

# User paths
%USERPROFILE%        # C:\Users\Username
%APPDATA%            # Application Data
%LOCALAPPDATA%       # Local Application Data
%TEMP% or %TMP%      # Temp directory
%HOMEPATH%           # \Users\Username

# Session info
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
# Redirect output to file (overwrite)
dir > output.txt

# Append to file
dir >> output.txt

# Redirect stderr to file
dir nonexistent 2> error.txt

# Redirect both stdout and stderr
dir > output.txt 2>&1

# Redirect to NUL (discard output)
dir > nul

# Redirect stderr to NUL
dir 2> nul

# Redirect both to NUL
dir > nul 2>&1
```

### Input Redirection
```cmd
# Read input from file
sort < input.txt

# Combine input and output redirection
sort < unsorted.txt > sorted.txt

# Use file as command input
findstr "pattern" < input.txt
```

### Pipes
```cmd
# Pipe output to another command
dir | more
dir | find "txt"
dir | sort

# Multiple pipes
dir /s | findstr /i "log" | more

# Combine with redirection
dir | find "txt" > textfiles.txt

# Process and filter
systeminfo | findstr /i "total physical memory"
netstat -an | findstr "ESTABLISHED"
tasklist | findstr /i "chrome"
```

### Advanced Redirection
```cmd
# Tee-like behavior (display and save)
dir | find "txt" | (echo. && type con && echo.) > output.txt

# Multiple commands with redirection
(echo Line 1 && echo Line 2 && echo Line 3) > output.txt

# Append multiple command outputs
echo First >> log.txt
echo Second >> log.txt

# Conditional execution with pipes
dir | findstr "file.txt" && echo Found || echo Not found
```

## Command Chaining

### Sequential Execution
```cmd
# Run commands in sequence (always)
command1 & command2 & command3

# Example
cd C:\Projects & dir & echo Done

# Each runs regardless of success
mkdir newfolder & cd newfolder & echo Created and moved
```

### Conditional Execution
```cmd
# Run second only if first succeeds
command1 && command2

# Example - create and enter directory
mkdir newfolder && cd newfolder

# Run second only if first fails
command1 || command2

# Example - create if not exists
exist file.txt || echo File not found

# Combine conditional operators
mkdir backup && copy *.txt backup\ || echo Failed to backup
```

### Grouping Commands
```cmd
# Group with parentheses
(command1 & command2 & command3)

# Conditional groups
(mkdir logs && cd logs && echo Log directory ready) || echo Setup failed

# Redirect grouped output
(echo Line 1 && echo Line 2 && echo Line 3) > output.txt
```

## File Attributes and Properties

### View and Modify Attributes
```cmd
# Display file attributes
attrib filename.txt

# Set read-only
attrib +r filename.txt

# Remove read-only
attrib -r filename.txt

# Set hidden
attrib +h filename.txt

# Set system file
attrib +s filename.txt

# Set archive attribute
attrib +a filename.txt

# Combine attributes
attrib +r +h filename.txt

# Apply to directory and contents
attrib +r /s /d C:\ImportantFolder\*.*
```

### File Comparison
```cmd
# Compare two files
fc file1.txt file2.txt

# Binary comparison
fc /b file1.exe file2.exe

# Case-insensitive comparison
fc /c file1.txt file2.txt

# Show only line count differences
fc /n file1.txt file2.txt

# Compare with specific tab size
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
# Good - prevents errors with empty variables
if "%username%"=="admin" echo Admin user

# Bad - fails if variable is empty
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
# Good - returns after completion
call other_script.bat

# Bad - transfers control, doesn't return
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
# Always check available options
findstr /?
for /?
if /?
```

## Common Mistakes (Don'ts)

❌ **Don't forget @echo off in batch files**
```cmd
# Without @echo off - messy output showing all commands
echo Starting...
dir
copy file.txt backup\

# With @echo off - clean output
@echo off
echo Starting...
dir
copy file.txt backup\
```

❌ **Don't use single % in batch files for variables**
```cmd
# Wrong in .bat file
for %f in (*.txt) do echo %f

# Correct in .bat file
for %%f in (*.txt) do echo %%f

# Note: Single % works in command line
for %f in (*.txt) do echo %f
```

❌ **Don't forget to handle spaces in paths**
```cmd
# Wrong
if exist C:\Program Files\app echo Found

# Correct
if exist "C:\Program Files\app" echo Found
```

❌ **Don't use outdated network commands**
```cmd
# Deprecated
nbtstat

# Better alternatives
Get-NetTCPConnection (PowerShell)
netstat -an
```

❌ **Don't forget to enable delayed expansion when needed**
```cmd
# Wrong - counter won't increment properly
set count=0
for %%f in (*.txt) do (
    set /a count+=1
    echo %count%    # Always shows 0
)

# Correct
setlocal enabledelayedexpansion
set count=0
for %%f in (*.txt) do (
    set /a count+=1
    echo !count!    # Shows incremented value
)
```

❌ **Don't kill critical system processes**
```cmd
# DON'T DO THIS
taskkill /f /im explorer.exe    # Kills Windows Explorer
taskkill /f /im csrss.exe       # Kills critical system process
taskkill /f /im winlogon.exe    # Kills logon process
```

❌ **Don't perform network operations without error checking**
```cmd
# Wrong
net use Z: \\server\share
copy *.txt Z:\

# Correct
net use Z: \\server\share
if %errorlevel% EQU 0 (
    copy *.txt Z:\
) else (
    echo Failed to connect to network share
)
```

❌ **Don't use GOTO excessively**
```cmd
# Bad - spaghetti code
:start
echo Enter choice
goto process

:process
goto validate

:validate
goto end

# Better - use CALL for structure
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
