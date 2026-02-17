# Windows CMD Beginner Cheatsheet

## Basic Navigation

### Change Directory
```cmd
# Navigate to a specific directory
cd C:\Users\YourName

# Go up one directory level
cd ..

# Go up multiple directory levels
cd ..\..

# Go to root of current drive
cd \

# Change to a different drive
D:

# Change drive and directory in one command
cd /d D:\Projects

# View current directory
cd
```

### List Directory Contents
```cmd
# List files and directories
dir

# List with details
dir /w      # Wide format
dir /p      # Pause after each screen
dir /s      # Include subdirectories
dir /a      # Show hidden files
dir /od     # Order by date
dir /on     # Order by name

# List specific file types
dir *.txt
dir *.exe
dir Document*.*

# Combine switches
dir /a /od /p
```

## File Operations

### Create Files
```cmd
# Create empty file
type nul > newfile.txt

# Create file with content
echo Hello World > greeting.txt

# Append to existing file
echo More text >> greeting.txt
```

### Copy Files
```cmd
# Copy single file
copy file.txt destination\

# Copy to specific location with new name
copy file.txt C:\Backup\newname.txt

# Copy multiple files
copy *.txt D:\Backup\

# Copy with verification
copy /v file.txt destination\

# Prompt before overwriting
copy /-y file.txt destination\

# Suppress prompts (force overwrite)
copy /y file.txt destination\
```

### Move Files
```cmd
# Move file to different directory
move file.txt C:\NewLocation\

# Move and rename
move oldname.txt C:\NewLocation\newname.txt

# Move multiple files
move *.log C:\Logs\

# Move with overwrite prompt
move /-y file.txt destination\
```

### Delete Files
```cmd
# Delete single file
del file.txt

# Delete multiple files
del *.tmp

# Delete with confirmation prompt
del /p file.txt

# Force delete read-only files
del /f file.txt

# Delete files in subdirectories
del /s *.bak

# Quiet mode (no confirmation)
del /q file.txt

# Delete all files in directory
del *.*
```

### Rename Files
```cmd
# Rename single file
ren oldname.txt newname.txt

# Rename multiple files (pattern-based)
ren *.txt *.bak

# Rename with full path
ren C:\Temp\file.txt newfile.txt
```

## Directory Operations

### Create Directories
```cmd
# Create single directory
mkdir NewFolder

# Create directory with full path
mkdir C:\Projects\MyApp

# Create nested directories
mkdir Parent\Child\GrandChild
```

### Remove Directories
```cmd
# Remove empty directory
rmdir EmptyFolder

# Remove directory and all contents
rmdir /s FolderWithFiles

# Quiet mode (no confirmation)
rmdir /s /q FolderWithFiles

# Remove directory tree
rd /s /q C:\Temp\OldProject
```

## Viewing File Contents

### Display File Contents
```cmd
# Display entire file
type filename.txt

# Display file with pause
type filename.txt | more

# Display multiple files
type file1.txt file2.txt

# Display with line numbers (using findstr)
findstr /n "^" filename.txt
```

### Page Through Files
```cmd
# View file page by page
more filename.txt

# View from specific line
more /s +10 filename.txt

# Display with expanded tabs
more /t4 filename.txt
```

## System Information

### Version Information
```cmd
# Display Windows version
ver

# Display detailed system information
systeminfo

# Quick system info
systeminfo | findstr /C:"OS Name" /C:"OS Version"
```

### Date and Time
```cmd
# Display current date
date

# Display current time
time

# Set date (requires admin)
date 12-25-2024

# Set time (requires admin)
time 14:30:00

# Display date and time in command output
echo Current date: %date%
echo Current time: %time%
```

### Computer Information
```cmd
# Display computer name
hostname

# Display username
echo %username%

# Display user domain
echo %userdomain%

# Display all environment variables
set
```

## Getting Help

### Command Help
```cmd
# Get help for any command
help dir
help copy
help

# Alternative help syntax
dir /?
copy /?
del /?

# List all available commands
help

# More information on specific command
help cd | more
```

## Path Management

### Display Current Path
```cmd
# Show full path of current directory
cd

# Show current drive
echo %cd%

# Show path environment variable
echo %path%
```

### Working with Paths
```cmd
# Absolute path
cd C:\Users\Documents

# Relative path from current location
cd ..\Downloads

# Path with spaces (use quotes)
cd "C:\Program Files"

# Short names for paths with spaces
dir /x    # Shows short names
cd PROGRA~1   # Short name for Program Files
```

## Command Line Basics

### Clear Screen
```cmd
# Clear the console
cls
```

### Exit Command Prompt
```cmd
# Close command prompt window
exit

# Close with specific exit code
exit /b 5
```

### Run Commands
```cmd
# Run command and keep window open
cmd /k dir

# Run command and close window
cmd /c dir

# Run multiple commands on one line
dir & echo Done

# Run second command only if first succeeds
mkdir newfolder && cd newfolder
```

### Echo Command
```cmd
# Display message
echo Hello World

# Display without newline (limited)
echo | set /p="Loading..."

# Turn command echoing off/on
echo off
echo on

# Display blank line
echo.
```

## Wildcards and Patterns

### Using Wildcards
```cmd
# * matches any characters
dir *.txt        # All .txt files
dir file*.*      # Files starting with "file"
dir *report*     # Files containing "report"

# ? matches single character
dir file?.txt    # file1.txt, fileA.txt, etc.
dir test??.log   # test01.log, testAB.log, etc.

# Combining wildcards
dir *.* /s       # All files in all subdirectories
del temp*.tmp    # Delete temp files
```

## Environment Variables

### Common Variables
```cmd
# User and system info
echo %username%        # Current user
echo %computername%    # Computer name
echo %userdomain%      # Domain name

# Paths
echo %cd%             # Current directory
echo %path%           # System PATH
echo %homepath%       # User home directory
echo %temp%           # Temp folder
echo %windir%         # Windows directory
echo %programfiles%   # Program Files directory

# System
echo %os%             # Operating system
echo %processor_architecture%
echo %number_of_processors%
```

### Using Variables in Commands
```cmd
# Navigate using variables
cd %userprofile%\Documents
cd %temp%

# Create backup with date
copy file.txt backup_%date:~-4,4%%date:~-10,2%%date:~-7,2%.txt
```

## Best Practices (Do's)

✅ **Use quotes for paths with spaces**
```cmd
# Good
cd "C:\Program Files\MyApp"
copy "My Document.txt" "C:\Backup\"

# Bad - will fail
cd C:\Program Files\MyApp
```

✅ **Verify before bulk operations**
```cmd
# First, view what will be affected
dir *.tmp

# Then perform the operation
del *.tmp
```

✅ **Use /? to check command options**
```cmd
# Always check available options
copy /?
del /?
```

✅ **Use descriptive directory names**
```cmd
# Good
mkdir ProjectBackup_2024
mkdir CustomerReports

# Less clear
mkdir bak
mkdir stuff
```

✅ **Double-check before deleting**
```cmd
# Use /p for confirmation
del /p *.txt

# Or verify first with dir
dir *.txt
```

✅ **Use cd to verify location before operations**
```cmd
# Always know where you are
cd
dir

# Then perform operations
copy *.txt backup\
```

## Common Mistakes (Don'ts)

❌ **Don't use del *.* without caution**
```cmd
# Dangerous - deletes all files
del *.*    # Better: del /p *.*

# Very dangerous with /s
del /s *.*   # Deletes in subdirectories too!
```

❌ **Don't forget drive letter when changing drives**
```cmd
# Wrong - just changes default, not current location
cd D:\Projects

# Correct - actually switches to drive D
cd /d D:\Projects
# Or
D:
cd \Projects
```

❌ **Don't ignore path spaces**
```cmd
# Wrong
cd C:\Program Files    # Fails

# Correct
cd "C:\Program Files"
cd C:\Progra~1         # Short name alternative
```

❌ **Don't use move for backup (use copy)**
```cmd
# Wrong - original is gone
move important.txt backup\

# Correct - keeps original
copy important.txt backup\
```

❌ **Don't assume case sensitivity**
```cmd
# CMD is case-insensitive
CD C:\Users           # Same as
cd c:\users           # Same as
Cd C:\USERS           # Same

# But filenames preserve case when created
```

❌ **Don't mix forward and back slashes**
```cmd
# Wrong
cd C:/Users\Documents

# Correct - use backslashes
cd C:\Users\Documents

# Note: Some commands accept forward slashes
```

❌ **Don't delete system directories**
```cmd
# Never delete these
rmdir C:\Windows      # DON'T!
del C:\System32\*.*   # DON'T!

# Be very careful with:
rmdir %systemroot%    # DON'T!
del %windir%\*.*      # DON'T!
```

❌ **Don't forget file extensions**
```cmd
# Unclear
ren report report_old    # What file type?

# Better - explicit
ren report.txt report_old.txt
```

## Quick Reference

### Essential Commands
```cmd
dir              # List directory contents
cd               # Change directory
mkdir            # Create directory
rmdir            # Remove directory
copy             # Copy files
move             # Move files
del              # Delete files
ren              # Rename files
type             # Display file contents
cls              # Clear screen
exit             # Exit CMD
help             # Get help
```

### Common Switches
```cmd
/s      # Include subdirectories
/p      # Prompt for confirmation
/q      # Quiet mode
/a      # Include hidden files
/f      # Force operation
/y      # Yes to all prompts
/-y     # Prompt for confirmation
/?      # Display help
```

### Navigation Shortcuts
```cmd
.       # Current directory
..      # Parent directory
\       # Root directory
~       # User home (in variables)
%cd%    # Current directory path
```
