# Windows CMD Beginner Cheatsheet

## Basic Navigation

### Change Directory
```cmd
REM Navigate to a specific directory
cd C:\Users\YourName

REM Go up one directory level
cd ..

REM Go up multiple directory levels
cd ..\..

REM Go to root of current drive
cd \

REM Change to a different drive
D:

REM Change drive and directory in one command
cd /d D:\Projects

REM View current directory
cd
```

### List Directory Contents
```cmd
REM List files and directories
dir

REM List with details
dir /w      & REM Wide format
dir /p      & REM Pause after each screen
dir /s      & REM Include subdirectories
dir /a      & REM Show hidden files
dir /od     & REM Order by date
dir /on     & REM Order by name

REM List specific file types
dir *.txt
dir *.exe
dir Document*.*

REM Combine switches
dir /a /od /p
```

## File Operations

### Create Files
```cmd
REM Create empty file
type nul > newfile.txt

REM Create file with content
echo Hello World > greeting.txt

REM Append to existing file
echo More text >> greeting.txt
```

### Copy Files
```cmd
REM Copy single file
copy file.txt destination\

REM Copy to specific location with new name
copy file.txt C:\Backup\newname.txt

REM Copy multiple files
copy *.txt D:\Backup\

REM Copy with verification
copy /v file.txt destination\

REM Prompt before overwriting
copy /-y file.txt destination\

REM Suppress prompts (force overwrite)
copy /y file.txt destination\
```

### Move Files
```cmd
REM Move file to different directory
move file.txt C:\NewLocation\

REM Move and rename
move oldname.txt C:\NewLocation\newname.txt

REM Move multiple files
move *.log C:\Logs\

REM Move with overwrite prompt
move /-y file.txt destination\
```

### Delete Files
```cmd
REM Delete single file
del file.txt

REM Delete multiple files
del *.tmp

REM Delete with confirmation prompt
del /p file.txt

REM Force delete read-only files
del /f file.txt

REM Delete files in subdirectories
del /s *.bak

REM Quiet mode (no confirmation)
del /q file.txt

REM Delete all files in directory
del *.*
```

### Rename Files
```cmd
REM Rename single file
ren oldname.txt newname.txt

REM Rename multiple files (pattern-based)
ren *.txt *.bak

REM Rename with full path
ren C:\Temp\file.txt newfile.txt
```

## Directory Operations

### Create Directories
```cmd
REM Create single directory
mkdir NewFolder

REM Create directory with full path
mkdir C:\Projects\MyApp

REM Create nested directories
mkdir Parent\Child\GrandChild
```

### Remove Directories
```cmd
REM Remove empty directory
rmdir EmptyFolder

REM Remove directory and all contents
rmdir /s FolderWithFiles

REM Quiet mode (no confirmation)
rmdir /s /q FolderWithFiles

REM Remove directory tree
rd /s /q C:\Temp\OldProject
```

## Viewing File Contents

### Display File Contents
```cmd
REM Display entire file
type filename.txt

REM Display file with pause
type filename.txt | more

REM Display multiple files
type file1.txt file2.txt

REM Display with line numbers (using findstr)
findstr /n "^" filename.txt
```

### Page Through Files
```cmd
REM View file page by page
more filename.txt

REM View from specific line
more /s +10 filename.txt

REM Display with expanded tabs
more /t4 filename.txt
```

## System Information

### Version Information
```cmd
REM Display Windows version
ver

REM Display detailed system information
systeminfo

REM Quick system info
systeminfo | findstr /C:"OS Name" /C:"OS Version"
```

### Date and Time
```cmd
REM Display current date
date

REM Display current time
time

REM Set date (requires admin)
date 12-25-2024

REM Set time (requires admin)
time 14:30:00

REM Display date and time in command output
echo Current date: %date%
echo Current time: %time%
```

### Computer Information
```cmd
REM Display computer name
hostname

REM Display username
echo %username%

REM Display user domain
echo %userdomain%

REM Display all environment variables
set
```

## Getting Help

### Command Help
```cmd
REM Get help for any command
help dir
help copy
help

REM Alternative help syntax
dir /?
copy /?
del /?

REM List all available commands
help

REM More information on specific command
help cd | more
```

## Path Management

### Display Current Path
```cmd
REM Show full path of current directory
cd

REM Show current drive
echo %cd%

REM Show path environment variable
echo %path%
```

### Working with Paths
```cmd
REM Absolute path
cd C:\Users\Documents

REM Relative path from current location
cd ..\Downloads

REM Path with spaces (use quotes)
cd "C:\Program Files"

REM Short names for paths with spaces
dir /x    & REM Shows short names
cd PROGRA~1   & REM Short name for Program Files
```

## Command Line Basics

### Clear Screen
```cmd
REM Clear the console
cls
```

### Exit Command Prompt
```cmd
REM Close command prompt window
exit

REM Close with specific exit code
exit /b 5
```

### Run Commands
```cmd
REM Run command and keep window open
cmd /k dir

REM Run command and close window
cmd /c dir

REM Run multiple commands on one line
dir & echo Done

REM Run second command only if first succeeds
mkdir newfolder && cd newfolder
```

### Echo Command
```cmd
REM Display message
echo Hello World

REM Display without newline (limited)
echo | set /p="Loading..."

REM Turn command echoing off/on
echo off
echo on

REM Display blank line
echo.
```

## Wildcards and Patterns

### Using Wildcards
```cmd
REM * matches any characters
dir *.txt        & REM All .txt files
dir file*.*      & REM Files starting with "file"
dir *report*     & REM Files containing "report"

REM ? matches single character
dir file?.txt    & REM file1.txt, fileA.txt, etc.
dir test??.log   & REM test01.log, testAB.log, etc.

REM Combining wildcards
dir *.* /s       & REM All files in all subdirectories
del temp*.tmp    & REM Delete temp files
```

## Environment Variables

### Common Variables
```cmd
REM User and system info
echo %username%        & REM Current user
echo %computername%    & REM Computer name
echo %userdomain%      & REM Domain name

REM Paths
echo %cd%             & REM Current directory
echo %path%           & REM System PATH
echo %homepath%       & REM User home directory
echo %temp%           & REM Temp folder
echo %windir%         & REM Windows directory
echo %programfiles%   & REM Program Files directory

REM System
echo %os%             & REM Operating system
echo %processor_architecture%
echo %number_of_processors%
```

### Using Variables in Commands
```cmd
REM Navigate using variables
cd %userprofile%\Documents
cd %temp%

REM Create backup with date
copy file.txt backup_%date:~-4,4%%date:~-10,2%%date:~-7,2%.txt
```

## Best Practices (Do's)

✅ **Use quotes for paths with spaces**
```cmd
REM Good
cd "C:\Program Files\MyApp"
copy "My Document.txt" "C:\Backup\"

REM Bad - will fail
cd C:\Program Files\MyApp
```

✅ **Verify before bulk operations**
```cmd
REM First, view what will be affected
dir *.tmp

REM Then perform the operation
del *.tmp
```

✅ **Use /? to check command options**
```cmd
REM Always check available options
copy /?
del /?
```

✅ **Use descriptive directory names**
```cmd
REM Good
mkdir ProjectBackup_2024
mkdir CustomerReports

REM Less clear
mkdir bak
mkdir stuff
```

✅ **Double-check before deleting**
```cmd
REM Use /p for confirmation
del /p *.txt

REM Or verify first with dir
dir *.txt
```

✅ **Use cd to verify location before operations**
```cmd
REM Always know where you are
cd
dir

REM Then perform operations
copy *.txt backup\
```

## Common Mistakes (Don'ts)

❌ **Don't use del *.* without caution**
```cmd
REM Dangerous - deletes all files
del *.*    & REM Better: del /p *.*

REM Very dangerous with /s
del /s *.*   & REM Deletes in subdirectories too!
```

❌ **Don't forget drive letter when changing drives**
```cmd
REM Wrong - just changes default, not current location
cd D:\Projects

REM Correct - actually switches to drive D
cd /d D:\Projects
REM Or
D:
cd \Projects
```

❌ **Don't ignore path spaces**
```cmd
REM Wrong
cd C:\Program Files    & REM Fails

REM Correct
cd "C:\Program Files"
cd C:\Progra~1         & REM Short name alternative
```

❌ **Don't use move for backup (use copy)**
```cmd
REM Wrong - original is gone
move important.txt backup\

REM Correct - keeps original
copy important.txt backup\
```

❌ **Don't assume case sensitivity**
```cmd
REM CMD is case-insensitive
CD C:\Users           & REM Same as
cd c:\users           & REM Same as
Cd C:\USERS           & REM Same

REM But filenames preserve case when created
```

❌ **Don't mix forward and back slashes**
```cmd
REM Wrong
cd C:/Users\Documents

REM Correct - use backslashes
cd C:\Users\Documents

REM Note: Some commands accept forward slashes
```

❌ **Don't delete system directories**
```cmd
REM Never delete these
rmdir C:\Windows      & REM DON'T!
del C:\System32\*.*   & REM DON'T!

REM Be very careful with:
rmdir %systemroot%    & REM DON'T!
del %windir%\*.*      & REM DON'T!
```

❌ **Don't forget file extensions**
```cmd
REM Unclear
ren report report_old    & REM What file type?

REM Better - explicit
ren report.txt report_old.txt
```

## Quick Reference

### Essential Commands
```cmd
dir              & REM List directory contents
cd               & REM Change directory
mkdir            & REM Create directory
rmdir            & REM Remove directory
copy             & REM Copy files
move             & REM Move files
del              & REM Delete files
ren              & REM Rename files
type             & REM Display file contents
cls              & REM Clear screen
exit             & REM Exit CMD
help             & REM Get help
```

### Common Switches
```cmd
/s      & REM Include subdirectories
/p      & REM Prompt for confirmation
/q      & REM Quiet mode
/a      & REM Include hidden files
/f      & REM Force operation
/y      & REM Yes to all prompts
/-y     & REM Prompt for confirmation
/?      & REM Display help
```

### Navigation Shortcuts
```cmd
.       & REM Current directory
..      & REM Parent directory
\       & REM Root directory
~       & REM User home (in variables)
%cd%    & REM Current directory path
```
