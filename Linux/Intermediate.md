# Linux Intermediate Commands Cheatsheet

## Advanced File Operations

### find - Search for Files
```bash
# Find by name
find /path -name "filename.txt"
find . -name "*.txt"

# Case-insensitive search
find . -iname "file.txt"

# Find directories only
find . -type d -name "dirname"

# Find files only
find . -type f -name "*.log"

# Find by size
find . -size +100M      # Larger than 100MB
find . -size -10k       # Smaller than 10KB

# Find by modification time
find . -mtime -7        # Modified in last 7 days
find . -mtime +30       # Modified more than 30 days ago

# Find and execute command
find . -name "*.tmp" -delete
find . -name "*.txt" -exec chmod 644 {} \;

# Find empty files
find . -type f -empty

# Find by permissions
find . -perm 777
```

### locate - Fast File Search
```bash
# Search using database
locate filename

# Case-insensitive
locate -i filename

# Update database (run as root)
sudo updatedb

# Show statistics
locate -S
```

## Process Management

### ps - Process Status
```bash
# Show user processes
ps

# Show all processes
ps aux
ps -ef

# Show process tree
ps auxf
ps -ejH

# Filter by name
ps aux | grep processname

# Show specific user
ps -u username

# Show with threads
ps -eLf
```

### top - Real-time Process Monitoring
```bash
# Interactive process viewer
top

# Shortcuts in top:
# h - Help
# k - Kill process
# r - Renice process
# P - Sort by CPU
# M - Sort by memory
# q - Quit
# 1 - Show individual CPUs

# Batch mode (non-interactive)
top -b -n 1
```

### htop - Enhanced Process Viewer
```bash
# Better alternative to top
htop

# Features:
# - Color coding
# - Mouse support
# - Tree view (F5)
# - Filter (F4)
# - Search (F3)
```

### kill - Terminate Processes
```bash
# Terminate by PID
kill 1234

# Force kill
kill -9 1234
kill -KILL 1234

# Terminate gracefully
kill -15 1234
kill -TERM 1234

# Kill by name
killall processname
pkill processname

# Kill by pattern
pkill -f "pattern"
```

### jobs, bg, fg - Job Control
```bash
# Run command in background
command &

# List background jobs
jobs

# Bring job to foreground
fg %1

# Send job to background
bg %1

# Suspend current process
Ctrl+Z

# Resume in background
bg

# Disown job (detach from terminal)
disown %1
```

## Redirection and Pipes

### Standard Streams
```bash
# stdin (0), stdout (1), stderr (2)

# Redirect stdout to file (overwrite)
command > output.txt

# Redirect stdout to file (append)
command >> output.txt

# Redirect stderr
command 2> error.txt

# Redirect both stdout and stderr
command > output.txt 2>&1
command &> output.txt

# Redirect stdin
command < input.txt

# Here document
cat << EOF > file.txt
Line 1
Line 2
EOF
```

### Pipes
```bash
# Send output to next command
command1 | command2

# Chain multiple commands
ps aux | grep apache | awk '{print $2}'

# Tee - save and display
command | tee output.txt
command | tee -a output.txt  # Append

# Count lines
ps aux | wc -l

# Sort and unique
cat file.txt | sort | uniq
```

## Text Editors

### nano - Simple Text Editor
```bash
# Open file
nano file.txt

# Keyboard shortcuts:
# Ctrl+O - Save
# Ctrl+X - Exit
# Ctrl+W - Search
# Ctrl+K - Cut line
# Ctrl+U - Paste
# Ctrl+G - Help
```

### vim - Powerful Text Editor
```bash
# Open file
vim file.txt

# Modes:
# Normal mode - Default (Esc)
# Insert mode - i, a, o
# Command mode - :

# Basic commands:
# :w - Save
# :q - Quit
# :wq - Save and quit
# :q! - Quit without saving

# Navigation:
# h, j, k, l - Left, down, up, right
# gg - Go to top
# G - Go to bottom
# 0 - Beginning of line
# $ - End of line

# Editing:
# i - Insert before cursor
# a - Insert after cursor
# o - New line below
# dd - Delete line
# yy - Copy line
# p - Paste
# u - Undo
# Ctrl+r - Redo

# Search:
# /pattern - Search forward
# ?pattern - Search backward
# n - Next match
# N - Previous match
```

## Archive and Compression

### tar - Archive Files
```bash
# Create archive
tar -cvf archive.tar directory/
tar -cvf archive.tar file1 file2

# Extract archive
tar -xvf archive.tar

# Create compressed archive (gzip)
tar -czvf archive.tar.gz directory/

# Extract compressed archive (gzip)
tar -xzvf archive.tar.gz

# Create compressed archive (bzip2)
tar -cjvf archive.tar.bz2 directory/

# Extract compressed archive (bzip2)
tar -xjvf archive.tar.bz2

# List archive contents
tar -tvf archive.tar

# Extract to specific directory
tar -xzvf archive.tar.gz -C /destination/

# Options:
# -c - Create
# -x - Extract
# -v - Verbose
# -f - File
# -z - Gzip compression
# -j - Bzip2 compression
```

### gzip/gunzip - Compression
```bash
# Compress file
gzip file.txt      # Creates file.txt.gz

# Decompress
gunzip file.txt.gz

# Keep original file
gzip -k file.txt

# Compress recursively
gzip -r directory/

# View compressed file
zcat file.txt.gz
```

### zip/unzip - ZIP Archives
```bash
# Create zip
zip archive.zip file1 file2
zip -r archive.zip directory/

# Extract zip
unzip archive.zip

# Extract to directory
unzip archive.zip -d /destination/

# List contents
unzip -l archive.zip

# Test archive integrity
unzip -t archive.zip
```

## Networking Commands

### ping - Test Connectivity
```bash
# Ping host
ping google.com

# Limit number of pings
ping -c 4 google.com

# Ping with interval
ping -i 2 google.com

# Set packet size
ping -s 100 google.com
```

### curl - Transfer Data
```bash
# Download file
curl -O https://example.com/file.txt

# Save with custom name
curl -o filename.txt https://example.com/file.txt

# Follow redirects
curl -L https://example.com

# POST request
curl -X POST -d "data=value" https://api.example.com

# Send JSON
curl -X POST -H "Content-Type: application/json" \
  -d '{"key":"value"}' https://api.example.com

# Show headers
curl -I https://example.com

# Verbose output
curl -v https://example.com

# Download with progress bar
curl -# -O https://example.com/largefile.iso
```

### wget - Download Files
```bash
# Download file
wget https://example.com/file.txt

# Download with custom name
wget -O filename.txt https://example.com/file.txt

# Continue interrupted download
wget -c https://example.com/largefile.iso

# Download in background
wget -b https://example.com/file.txt

# Mirror website
wget -m https://example.com

# Limit download speed
wget --limit-rate=200k https://example.com/file.txt
```

### ssh - Secure Shell
```bash
# Connect to remote server
ssh username@hostname

# Use specific port
ssh -p 2222 username@hostname

# Run command on remote
ssh username@hostname "ls -la"

# Use SSH key
ssh -i ~/.ssh/id_rsa username@hostname

# X11 forwarding
ssh -X username@hostname

# Keep connection alive
ssh -o ServerAliveInterval=60 username@hostname
```

### scp - Secure Copy
```bash
# Copy to remote
scp file.txt username@hostname:/path/

# Copy from remote
scp username@hostname:/path/file.txt .

# Copy directory (recursive)
scp -r directory/ username@hostname:/path/

# Use specific port
scp -P 2222 file.txt username@hostname:/path/

# Preserve attributes
scp -p file.txt username@hostname:/path/
```

## Disk Usage

### df - Disk Space
```bash
# Show disk space
df

# Human-readable format
df -h

# Show specific filesystem
df -h /

# Show inode usage
df -i

# Show filesystem type
df -T
```

### du - Directory Usage
```bash
# Show directory size
du -sh directory/

# Show all files and directories
du -ah directory/

# Show top-level directories only
du -h --max-depth=1

# Sort by size
du -sh * | sort -h

# Exclude files
du -sh --exclude="*.log" directory/
```

## System Information

### uname - System Information
```bash
# Show all information
uname -a

# Show kernel name
uname -s

# Show kernel release
uname -r

# Show machine hardware
uname -m

# Show processor type
uname -p
```

### hostname - Show/Set Hostname
```bash
# Show hostname
hostname

# Show FQDN
hostname -f

# Show IP address
hostname -I

# Set hostname (requires root)
sudo hostnamectl set-hostname newhostname
```

### uptime - System Uptime
```bash
# Show uptime and load average
uptime

# Human-readable format
uptime -p

# Since when system is up
uptime -s
```

### date - Date and Time
```bash
# Show current date and time
date

# Custom format
date "+%Y-%m-%d %H:%M:%S"
date "+%Y%m%d"

# Show UTC time
date -u

# Convert timestamp
date -d @1609459200
date -d "2021-01-01"

# Set date (requires root)
sudo date -s "2021-01-01 12:00:00"
```

## Best Practices (Do's)

✅ **Use find for complex searches**
```bash
# Combine multiple criteria
find . -type f -name "*.log" -mtime +30 -size +100M
```

✅ **Check process before killing**
```bash
ps aux | grep process_name
kill -15 PID  # Try graceful shutdown first
```

✅ **Use tar with compression**
```bash
# More efficient
tar -czvf backup.tar.gz directory/
```

✅ **Test SSH connection before scripts**
```bash
ssh -o ConnectTimeout=5 user@host "echo OK"
```

✅ **Use pipes for efficiency**
```bash
# One pass through data
cat file.txt | grep "pattern" | sort | uniq -c
```

✅ **Monitor long-running processes**
```bash
# Run with nohup
nohup long_running_command &

# Or use screen/tmux
screen -S session_name
```

## Common Mistakes (Don'ts)

❌ **Don't kill process with -9 immediately**
```bash
# Bad
kill -9 PID

# Good - try graceful first
kill -15 PID
sleep 2
kill -9 PID  # Only if needed
```

❌ **Don't use locate without updating**
```bash
# May show outdated results
locate filename

# Better
sudo updatedb && locate filename
```

❌ **Don't extract archives without checking contents**
```bash
# Check first
tar -tzf archive.tar.gz

# Then extract
tar -xzf archive.tar.gz
```

❌ **Don't use wget/curl without -O for important files**
```bash
# May save with wrong name
wget https://example.com/download

# Better
wget -O specific_name.txt https://example.com/download
```

❌ **Don't ignore stderr**
```bash
# Bad - errors hidden
command > output.txt

# Good - capture both
command > output.txt 2>&1
```

❌ **Don't use plain text passwords in commands**
```bash
# Bad - visible in history
ssh user@host -p password

# Good - use SSH keys
ssh -i ~/.ssh/id_rsa user@host
```
