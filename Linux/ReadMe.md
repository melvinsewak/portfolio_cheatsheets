# Linux Commands Cheatsheet

Welcome to the Linux Commands cheatsheet! This guide will help you quickly master the Linux command line.

## Overview

Linux is a powerful, open-source operating system kernel that powers millions of servers, desktops, and devices worldwide. The Linux command line (terminal/shell) provides direct access to the system, enabling efficient automation, system administration, and development workflows.

## Contents

- [Beginner.md](Beginner.md) - Basic navigation, file operations, and essential commands
- [Intermediate.md](Intermediate.md) - Process management, networking, text processing, and system tools
- [Advanced.md](Advanced.md) - Shell scripting, advanced text processing, system administration, and automation

## Quick Reference

### Basic Navigation
```bash
# Show current directory
pwd

# List files
ls -la

# Change directory
cd /path/to/directory

# Go to home directory
cd ~
```

### File Operations
```bash
# Create file
touch filename.txt

# Copy file
cp source.txt destination.txt

# Move/rename file
mv oldname.txt newname.txt

# Delete file
rm filename.txt
```

### Viewing Files
```bash
# Display entire file
cat file.txt

# View with pagination
less file.txt

# Show first 10 lines
head file.txt

# Show last 10 lines
tail file.txt

# Follow file updates in real-time
tail -f logfile.txt
```

## Key Features

- **Powerful**: Direct system access and control
- **Efficient**: Fast execution and low resource usage
- **Scriptable**: Automate repetitive tasks
- **Portable**: Works across different Linux distributions
- **Flexible**: Combine commands with pipes and redirection

## Getting Started

### Accessing the Terminal

**Ubuntu/Debian**: `Ctrl + Alt + T`  
**macOS**: `Cmd + Space`, type "Terminal"  
**Windows**: Use WSL (Windows Subsystem for Linux) or Git Bash

### Basic Command Structure

```bash
command [options] [arguments]
```

Example:
```bash
ls -l /home/user
#  │  │    └── argument (path)
#  │  └── option (long format)
#  └── command
```

## Essential Tips

✅ **Use Tab completion** - Press Tab to auto-complete commands and file names  
✅ **Check command history** - Use `history` or press Up arrow  
✅ **Use man pages** - Type `man command` for detailed documentation  
✅ **Be careful with sudo** - Always verify commands before running with root privileges

## Additional Resources

- [Linux Command Line Basics](https://ubuntu.com/tutorials/command-line-for-beginners)
- [GNU Coreutils Manual](https://www.gnu.org/software/coreutils/manual/)
- [Linux Journey](https://linuxjourney.com/)
- [The Linux Documentation Project](https://tldp.org/)
