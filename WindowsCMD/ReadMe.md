# Windows CMD Commands Cheatsheet

Welcome to the Windows Command Prompt (CMD) cheatsheet! This guide will help you master the Windows command line.

## Overview

Windows Command Prompt (CMD) is the command-line interpreter for Windows operating systems. It provides a text-based interface for executing commands, running scripts, and performing system administration tasks.

## Contents

- [Beginner.md](Beginner.md) - Basic navigation, file operations, and essential commands
- [Intermediate.md](Intermediate.md) - Batch scripting, networking, and system management
- [Advanced.md](Advanced.md) - Advanced scripting, automation, and system administration

## Quick Reference

### Basic Navigation
```cmd
REM Show current directory
cd

REM Change directory
cd C:\Users\Username

REM Go to parent directory
cd ..

REM List files
dir

REM List with details
dir /a
```

### File Operations
```cmd
REM Create file
type nul > filename.txt
echo content > file.txt

REM Copy file
copy source.txt destination.txt

REM Move file
move oldname.txt newname.txt

REM Delete file
del filename.txt
```

### System Information
```cmd
REM Show system information
systeminfo

REM Show IP configuration
ipconfig

REM Show active processes
tasklist

REM Show computer name
hostname
```

## Key Features

- **Native to Windows**: Built into every Windows installation
- **Scripting**: Create batch (.bat) files for automation
- **System Management**: Control Windows services and settings
- **Network Tools**: Built-in networking commands
- **Backwards Compatible**: Supports legacy DOS commands

## Getting Started

### Opening Command Prompt

**Method 1**: Press `Win + R`, type `cmd`, press Enter

**Method 2**: Search "Command Prompt" in Start menu

**Method 3**: `Win + X`, select "Command Prompt" or "Windows PowerShell"

**Run as Administrator**: Right-click Command Prompt, select "Run as administrator"

### Basic Command Structure

```cmd
command [options] [arguments]
```

Example:
```cmd
dir /a /s C:\Users
│   │  │  └── argument (path)
│   │  └── option (include subdirectories)
│   └── option (all files)
└── command
```

## Essential Tips

✅ **Use Tab completion** - Press Tab to auto-complete file and folder names  
✅ **Use F7 for command history** - View and select previous commands  
✅ **Use /? for help** - Type `command /?` for help on any command  
✅ **Run as Administrator when needed** - Many system commands require elevated privileges  
✅ **Use quotes for paths with spaces** - Example: `cd "C:\Program Files"`

## CMD vs PowerShell

**CMD (Command Prompt)**:
- Legacy command-line interface
- Simple text-based commands
- Best for basic tasks and batch files
- Limited scripting capabilities

**PowerShell**:
- Modern command-line shell
- Object-oriented
- More powerful scripting
- Cross-platform (PowerShell Core)

## Common Keyboard Shortcuts

- **F1**: Paste last command one character at a time
- **F3**: Repeat last command
- **F7**: Display command history
- **F9**: Run command from history by number
- **Ctrl + C**: Cancel running command
- **Ctrl + V** or **Right-click**: Paste
- **Alt + Enter**: Toggle fullscreen
- **Up/Down arrows**: Navigate command history

## Additional Resources

- [Windows CMD Reference](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands)
- [SS64 CMD Reference](https://ss64.com/nt/)
- [Command-Line Reference A-Z](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands)
