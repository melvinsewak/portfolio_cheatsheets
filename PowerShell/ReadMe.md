# PowerShell Cheatsheet

Welcome to the PowerShell cheatsheet! This guide will help you quickly get up to speed with PowerShell scripting and automation.

## Overview

PowerShell is a powerful, cross-platform task automation solution made up of a command-line shell, a scripting language, and a configuration management framework. Built on .NET, PowerShell works with objects rather than text, making it incredibly powerful for system administration and automation tasks.

## Contents

- [Beginner.md](Beginner.md) - Basics, cmdlets, variables, control flow, and fundamental concepts
- [Intermediate.md](Intermediate.md) - Advanced functions, pipeline, objects, modules, and error handling
- [Advanced.md](Advanced.md) - Script modules, classes, DSC, performance optimization, and security

## Quick Reference

### Basic Syntax
```powershell
# Variables
$name = "Alice"
$age = 25
$isAdmin = $true

# Cmdlet (Verb-Noun format)
Get-Process
Get-ChildItem -Path C:\Temp

# Function
function Get-Greeting {
    param($Name)
    return "Hello, $Name!"
}

# Pipeline
Get-Process | Where-Object CPU -gt 100 | Select-Object Name, CPU

# Object
$person = [PSCustomObject]@{
    Name = "Bob"
    Age  = 30
}

# Array
$numbers = 1, 2, 3, 4, 5
```

### Key Features

- **Cross-Platform**: Runs on Windows, Linux, and macOS
- **Object-Oriented**: Works with .NET objects, not just text
- **Consistent Syntax**: Verb-Noun cmdlet naming convention
- **Pipeline Power**: Pass objects between commands seamlessly
- **Automation**: Automate repetitive tasks and system administration
- **Discoverable**: Built-in help system with Get-Help and Get-Command

## Getting Started

### Installation

**Windows**: PowerShell 5.1 is pre-installed. For PowerShell 7+:
```powershell
winget install Microsoft.PowerShell
```

**Linux**:
```bash
# Ubuntu/Debian
sudo apt-get install -y powershell

# Red Hat/CentOS
sudo yum install -y powershell
```

**macOS**:
```bash
brew install --cask powershell
```

### Running PowerShell

**Windows**:
- Windows PowerShell: Search "PowerShell" in Start Menu
- PowerShell 7+: Search "pwsh" in Start Menu

**Linux/macOS**:
```bash
pwsh
```

### Your First Commands
```powershell
# Get help about a cmdlet
Get-Help Get-Process

# List files in current directory
Get-ChildItem

# Get running processes
Get-Process

# Get system information
Get-ComputerInfo

# Find commands
Get-Command *Process*
```

## Additional Resources

- [Official PowerShell Documentation](https://docs.microsoft.com/en-us/powershell/)
- [PowerShell GitHub Repository](https://github.com/PowerShell/PowerShell)
- [PowerShell Gallery](https://www.powershellgallery.com/)
- [PowerShell Community](https://devblogs.microsoft.com/powershell/)
- [Learn PowerShell](https://learn.microsoft.com/en-us/training/browse/?terms=powershell)
