# Linux Beginner Commands Cheatsheet

## Navigation Commands

### pwd - Print Working Directory
```bash
# Show current directory path
pwd
# Output: /home/username
```

### cd - Change Directory
```bash
# Go to home directory
cd ~
cd

# Go to specific directory
cd /var/log

# Go up one level
cd ..

# Go up two levels
cd ../..

# Go to previous directory
cd -

# Go to root directory
cd /
```

### ls - List Directory Contents
```bash
# Basic listing
ls

# Long format (detailed)
ls -l

# Show hidden files
ls -a

# Long format with human-readable sizes
ls -lh

# Sort by modification time
ls -lt

# Reverse order
ls -lr

# Recursive listing
ls -R

# Combined options
ls -lah
```

## File Operations

### touch - Create Empty File
```bash
# Create single file
touch file.txt

# Create multiple files
touch file1.txt file2.txt file3.txt

# Update file timestamp
touch existing_file.txt
```

### mkdir - Make Directory
```bash
# Create directory
mkdir mydir

# Create nested directories
mkdir -p parent/child/grandchild

# Create multiple directories
mkdir dir1 dir2 dir3

# Create with permissions
mkdir -m 755 mydir
```

### cp - Copy Files
```bash
# Copy file
cp source.txt destination.txt

# Copy to directory
cp file.txt /path/to/directory/

# Copy multiple files
cp file1.txt file2.txt /destination/

# Copy directory (recursive)
cp -r sourcedir destdir

# Copy with confirmation
cp -i source.txt dest.txt

# Preserve attributes
cp -p file.txt copy.txt

# Verbose output
cp -v file.txt copy.txt
```

### mv - Move/Rename Files
```bash
# Rename file
mv oldname.txt newname.txt

# Move file to directory
mv file.txt /path/to/directory/

# Move multiple files
mv file1.txt file2.txt /destination/

# Move directory
mv olddir newdir

# Interactive (ask before overwrite)
mv -i source.txt dest.txt
```

### rm - Remove Files
```bash
# Delete file
rm file.txt

# Delete multiple files
rm file1.txt file2.txt

# Interactive deletion
rm -i file.txt

# Remove directory and contents
rm -r directory

# Force removal (no confirmation)
rm -f file.txt

# Verbose output
rm -v file.txt

# Remove empty directory
rmdir emptydir
```

## Viewing File Contents

### cat - Concatenate and Display
```bash
# Display file contents
cat file.txt

# Display multiple files
cat file1.txt file2.txt

# Display with line numbers
cat -n file.txt

# Concatenate files into new file
cat file1.txt file2.txt > combined.txt

# Append to file
cat file1.txt >> existing.txt
```

### less - View File with Pagination
```bash
# View file (scrollable)
less file.txt

# Navigation in less:
# Space/Page Down - Next page
# b/Page Up - Previous page
# / - Search forward
# ? - Search backward
# q - Quit
# G - Go to end
# g - Go to beginning
```

### more - Simple Pager
```bash
# View file page by page
more file.txt

# Space - Next page
# Enter - Next line
# q - Quit
```

### head - View Beginning of File
```bash
# Show first 10 lines (default)
head file.txt

# Show first N lines
head -n 20 file.txt
head -20 file.txt

# Show first N bytes
head -c 100 file.txt

# Multiple files
head file1.txt file2.txt
```

### tail - View End of File
```bash
# Show last 10 lines (default)
tail file.txt

# Show last N lines
tail -n 20 file.txt
tail -20 file.txt

# Follow file in real-time (logs)
tail -f logfile.txt

# Follow with retry
tail -F logfile.txt

# Show last N lines and follow
tail -n 50 -f logfile.txt
```

## File Permissions

### chmod - Change File Permissions
```bash
# Numeric notation (rwx = 421)
# Owner-Group-Others
chmod 755 file.txt    # rwxr-xr-x
chmod 644 file.txt    # rw-r--r--
chmod 600 file.txt    # rw-------
chmod 777 file.txt    # rwxrwxrwx

# Symbolic notation
chmod u+x file.txt    # Add execute for owner
chmod g-w file.txt    # Remove write for group
chmod o+r file.txt    # Add read for others
chmod a+x file.txt    # Add execute for all

# Recursive
chmod -R 755 directory

# Reference another file
chmod --reference=ref.txt file.txt
```

### chown - Change Ownership
```bash
# Change owner
sudo chown username file.txt

# Change owner and group
sudo chown username:groupname file.txt

# Change group only
sudo chown :groupname file.txt

# Recursive
sudo chown -R username:groupname directory
```

### Understanding Permissions
```
-rwxr-xr-x  1 user group 4096 Jan 1 12:00 file.txt
│││││││││
│└┬┘└┬┘└┬┘
│ │  │  └─ Others: r-x (read, execute)
│ │  └──── Group: r-x (read, execute)
│ └─────── Owner: rwx (read, write, execute)
└───────── File type: - (regular file), d (directory), l (link)

Numeric values:
r (read) = 4
w (write) = 2
x (execute) = 1
```

## Basic Text Processing

### grep - Search Text
```bash
# Search for pattern in file
grep "pattern" file.txt

# Case-insensitive search
grep -i "pattern" file.txt

# Show line numbers
grep -n "pattern" file.txt

# Count matches
grep -c "pattern" file.txt

# Show lines that don't match
grep -v "pattern" file.txt

# Search in multiple files
grep "pattern" *.txt

# Recursive search in directory
grep -r "pattern" /path/to/dir

# Show only filenames
grep -l "pattern" *.txt
```

### wc - Word Count
```bash
# Count lines, words, characters
wc file.txt

# Count lines only
wc -l file.txt

# Count words only
wc -w file.txt

# Count characters only
wc -c file.txt

# Multiple files
wc file1.txt file2.txt
```

### sort - Sort Lines
```bash
# Sort alphabetically
sort file.txt

# Sort in reverse
sort -r file.txt

# Sort numerically
sort -n numbers.txt

# Sort by column
sort -k 2 file.txt

# Remove duplicates
sort -u file.txt

# Case-insensitive sort
sort -f file.txt
```

## Getting Help

### man - Manual Pages
```bash
# View command manual
man ls
man grep
man bash

# Search man pages
man -k keyword

# Show all sections
man -a command

# Navigation:
# Space - Next page
# b - Previous page
# / - Search
# q - Quit
```

### --help Option
```bash
# Quick help
ls --help
grep --help
cp --help

# Usually shows:
# - Usage syntax
# - Available options
# - Brief descriptions
```

### which - Locate Command
```bash
# Find command path
which python
which ls

# Show all matches
which -a python
```

### whereis - Locate Binary, Source, Manual
```bash
# Find command files
whereis ls
whereis python

# Shows:
# - Binary location
# - Source location
# - Manual page location
```

## Command Structure

### Basic Syntax
```bash
command [options] [arguments]

# Examples:
ls -l /home        # command + option + argument
cp -r dir1 dir2    # command + option + arguments
grep -i "text" file.txt
```

### Options
```bash
# Short options (single dash)
ls -l
ls -a
ls -la    # Combined

# Long options (double dash)
ls --all
ls --human-readable
grep --ignore-case "pattern" file.txt
```

## Best Practices (Do's)

✅ **Use tab completion** - Save time and avoid typos
```bash
cd /home/us[Tab]  # Completes to /home/username/
```

✅ **Check before destructive operations**
```bash
# Use -i for interactive mode
rm -i file.txt
mv -i source.txt dest.txt
```

✅ **Use ls before cd** - Verify directory exists
```bash
ls /path/to/directory
cd /path/to/directory
```

✅ **Read man pages** - Understand commands fully
```bash
man ls
man grep
```

✅ **Use meaningful names** - For files and directories
```bash
# Good
mkdir project_backup
touch meeting_notes.txt

# Avoid
mkdir x
touch f.txt
```

✅ **Double-check paths** - Especially with rm
```bash
# Check current directory
pwd

# List before removing
ls directory/
rm -r directory/
```

## Common Mistakes (Don'ts)

❌ **Don't use rm -rf without being certain**
```bash
# DANGEROUS - Can delete everything!
rm -rf /

# Always specify exact path
rm -rf /path/to/specific/directory
```

❌ **Don't ignore case sensitivity**
```bash
# Linux is case-sensitive
File.txt ≠ file.txt ≠ FILE.txt
```

❌ **Don't forget sudo for system operations**
```bash
# Will fail (permission denied)
apt install package

# Correct
sudo apt install package
```

❌ **Don't use spaces in filenames without quotes**
```bash
# Wrong
cd my folder

# Correct
cd "my folder"
cd my\ folder
```

❌ **Don't chain commands without understanding them**
```bash
# Risky - second command runs even if first fails
cd /nonexistent; rm -rf *

# Better - use && (second runs only if first succeeds)
cd /nonexistent && rm -rf *
```

❌ **Don't ignore error messages**
```bash
# Always read and understand errors
cp file.txt /protected/
# cp: cannot create regular file '/protected/file.txt': Permission denied

# Fix: use sudo or correct permissions
```

❌ **Don't use wildcards carelessly**
```bash
# Be specific
rm *.txt  # Removes all .txt files in current directory

# Better - verify first
ls *.txt
rm *.txt
```
