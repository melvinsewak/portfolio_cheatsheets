# Linux Advanced Commands Cheatsheet

## Advanced Text Processing

### sed - Stream Editor
```bash
# Replace text (first occurrence)
sed 's/old/new/' file.txt

# Replace all occurrences
sed 's/old/new/g' file.txt

# Replace and save to file
sed 's/old/new/g' file.txt > output.txt

# Edit file in-place
sed -i 's/old/new/g' file.txt

# Delete lines
sed '3d' file.txt              # Delete line 3
sed '1,5d' file.txt            # Delete lines 1-5
sed '/pattern/d' file.txt      # Delete lines matching pattern

# Print specific lines
sed -n '1,10p' file.txt        # Print lines 1-10
sed -n '/pattern/p' file.txt   # Print matching lines

# Multiple operations
sed -e 's/old/new/g' -e 's/foo/bar/g' file.txt

# Insert/append lines
sed '3i\New Line' file.txt     # Insert before line 3
sed '3a\New Line' file.txt     # Append after line 3

# Replace on specific line
sed '5s/old/new/' file.txt

# Case-insensitive replace
sed 's/old/new/gI' file.txt
```

### awk - Text Processing
```bash
# Print specific columns
awk '{print $1}' file.txt      # First column
awk '{print $1, $3}' file.txt  # First and third columns
awk '{print $NF}' file.txt     # Last column

# Print with custom delimiter
awk -F':' '{print $1}' /etc/passwd

# Conditional printing
awk '$3 > 100' file.txt        # Print if 3rd column > 100
awk '$1 == "root"' /etc/passwd

# Pattern matching
awk '/pattern/ {print $0}' file.txt

# Sum column values
awk '{sum += $1} END {print sum}' file.txt

# Count lines
awk 'END {print NR}' file.txt

# Print with formatting
awk '{printf "%-10s %5s\n", $1, $2}' file.txt

# Built-in variables
# NF - Number of fields
# NR - Number of records (line number)
# FS - Field separator
# OFS - Output field separator

# Complex example
awk -F':' '$3 >= 1000 {print $1, $3}' /etc/passwd
```

## Shell Scripting Basics

### Creating Scripts
```bash
#!/bin/bash
# Script header (shebang)

# Variables
NAME="John"
AGE=30

# Print variables
echo "Name: $NAME"
echo "Age: ${AGE}"

# Command substitution
CURRENT_DATE=$(date +%Y-%m-%d)
FILES=$(ls -1 | wc -l)

# User input
read -p "Enter your name: " USERNAME
echo "Hello, $USERNAME"

# Conditional statements
if [ $AGE -gt 18 ]; then
    echo "Adult"
elif [ $AGE -eq 18 ]; then
    echo "Just turned adult"
else
    echo "Minor"
fi

# Loops
for i in {1..5}; do
    echo "Number: $i"
done

for file in *.txt; do
    echo "Processing: $file"
done

while [ $COUNT -lt 10 ]; do
    echo $COUNT
    ((COUNT++))
done

# Functions
function greet() {
    echo "Hello, $1"
}

greet "Alice"

# Exit status
if [ $? -eq 0 ]; then
    echo "Success"
else
    echo "Failed"
fi
```

### Script Best Practices
```bash
#!/bin/bash

# Enable strict mode
set -euo pipefail  # Exit on error, undefined var, pipe failures

# Variables
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly CONFIG_FILE="${SCRIPT_DIR}/config.conf"

# Functions
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" >&2
}

error_exit() {
    log "ERROR: $1"
    exit 1
}

# Check if file exists
if [[ ! -f "$CONFIG_FILE" ]]; then
    error_exit "Config file not found: $CONFIG_FILE"
fi

# Cleanup on exit
cleanup() {
    log "Cleaning up..."
    rm -f /tmp/temp_file
}
trap cleanup EXIT

# Main logic
main() {
    log "Starting script..."
    # Your code here
    log "Script completed successfully"
}

main "$@"
```

## Advanced Permissions

### ACLs (Access Control Lists)
```bash
# Install ACL tools
sudo apt-get install acl  # Debian/Ubuntu
sudo yum install acl      # RHEL/CentOS

# View ACLs
getfacl file.txt

# Set ACL for user
setfacl -m u:username:rwx file.txt

# Set ACL for group
setfacl -m g:groupname:rx file.txt

# Remove specific ACL
setfacl -x u:username file.txt

# Remove all ACLs
setfacl -b file.txt

# Set default ACL for directory
setfacl -d -m u:username:rwx directory/

# Recursive ACL
setfacl -R -m u:username:rx directory/

# Copy ACLs
getfacl file1.txt | setfacl --set-file=- file2.txt
```

### Special Permissions
```bash
# setuid (Set User ID)
chmod u+s /path/to/file
chmod 4755 /path/to/file

# setgid (Set Group ID)
chmod g+s /path/to/directory
chmod 2755 /path/to/directory

# Sticky bit (only owner can delete)
chmod +t /path/to/directory
chmod 1777 /path/to/directory

# View special permissions
ls -l /path/to/file
# -rwsr-xr-x (setuid)
# -rwxr-sr-x (setgid)
# drwxrwxrwt (sticky bit)
```

## Package Management

### APT (Debian/Ubuntu)
```bash
# Update package list
sudo apt update

# Upgrade packages
sudo apt upgrade
sudo apt full-upgrade

# Install package
sudo apt install package-name

# Remove package
sudo apt remove package-name
sudo apt purge package-name  # Remove with config files

# Search for package
apt search keyword
apt-cache search keyword

# Show package info
apt show package-name

# List installed packages
apt list --installed

# Clean up
sudo apt autoremove
sudo apt autoclean
```

### YUM/DNF (RHEL/CentOS/Fedora)
```bash
# Update system
sudo yum update       # RHEL/CentOS 7
sudo dnf update       # RHEL/CentOS 8+, Fedora

# Install package
sudo yum install package-name
sudo dnf install package-name

# Remove package
sudo yum remove package-name
sudo dnf remove package-name

# Search package
yum search keyword
dnf search keyword

# Show package info
yum info package-name
dnf info package-name

# List installed packages
yum list installed
dnf list installed

# Clean cache
sudo yum clean all
sudo dnf clean all
```

## Systemd and Services

### systemctl - Service Management
```bash
# Start service
sudo systemctl start service-name

# Stop service
sudo systemctl stop service-name

# Restart service
sudo systemctl restart service-name

# Reload configuration
sudo systemctl reload service-name

# Enable service (start on boot)
sudo systemctl enable service-name

# Disable service
sudo systemctl disable service-name

# Check service status
systemctl status service-name

# Show if service is enabled
systemctl is-enabled service-name

# Show if service is active
systemctl is-active service-name

# List all services
systemctl list-units --type=service

# List failed services
systemctl --failed

# View service logs
journalctl -u service-name

# Follow logs in real-time
journalctl -u service-name -f
```

### Creating Custom Service
```bash
# Create service file
sudo nano /etc/systemd/system/myapp.service

# Service file content:
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=username
WorkingDirectory=/path/to/app
ExecStart=/path/to/app/start.sh
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target

# Reload systemd
sudo systemctl daemon-reload

# Enable and start
sudo systemctl enable myapp
sudo systemctl start myapp
```

## Cron Jobs and Scheduling

### crontab - Schedule Jobs
```bash
# Edit crontab
crontab -e

# List crontabs
crontab -l

# Remove crontab
crontab -r

# Crontab format:
# * * * * * command
# │ │ │ │ │
# │ │ │ │ └─ Day of week (0-7, 0 and 7 = Sunday)
# │ │ │ └─── Month (1-12)
# │ │ └───── Day of month (1-31)
# │ └─────── Hour (0-23)
# └───────── Minute (0-59)

# Examples:
# Run every minute
* * * * * /path/to/script.sh

# Run every hour
0 * * * * /path/to/script.sh

# Run daily at 2:30 AM
30 2 * * * /path/to/script.sh

# Run every Monday at 9 AM
0 9 * * 1 /path/to/script.sh

# Run first day of month
0 0 1 * * /path/to/script.sh

# Run every 15 minutes
*/15 * * * * /path/to/script.sh

# Run on weekdays at 6 PM
0 18 * * 1-5 /path/to/script.sh

# Multiple times
0 8,12,16 * * * /path/to/script.sh
```

### anacron - For Non-24/7 Systems
```bash
# Edit /etc/anacrontab
sudo nano /etc/anacrontab

# Format: period delay job-id command
# Run daily
1 5 daily-job /path/to/daily-script.sh

# Run weekly
7 10 weekly-job /path/to/weekly-script.sh

# Run monthly
@monthly 15 monthly-job /path/to/monthly-script.sh
```

## Performance Monitoring

### vmstat - Virtual Memory Statistics
```bash
# Display statistics
vmstat

# Update every 2 seconds
vmstat 2

# 10 updates, 2 seconds apart
vmstat 2 10

# Memory in MB
vmstat -S m

# Disk statistics
vmstat -d
```

### iostat - I/O Statistics
```bash
# Display I/O statistics
iostat

# Update every 2 seconds
iostat 2

# Show extended statistics
iostat -x

# Show specific device
iostat -x sda

# Human-readable
iostat -h
```

### sar - System Activity Reporter
```bash
# CPU usage
sar -u 1 5

# Memory usage
sar -r 1 5

# I/O statistics
sar -b 1 5

# Network statistics
sar -n DEV 1 5

# Load average
sar -q 1 5

# View historical data
sar -f /var/log/sysstat/sa01
```

## Advanced Networking

### netstat - Network Statistics
```bash
# Show all connections
netstat -a

# Show listening ports
netstat -l

# Show TCP connections
netstat -t

# Show UDP connections
netstat -u

# Show program names
sudo netstat -p

# Show routing table
netstat -r

# Numeric addresses
netstat -n

# Combined (popular)
sudo netstat -tulpn
```

### ss - Socket Statistics (modern netstat)
```bash
# Show all sockets
ss -a

# Show listening sockets
ss -l

# Show TCP sockets
ss -t

# Show UDP sockets
ss -u

# Show process using socket
ss -p

# Numeric addresses
ss -n

# Combined
ss -tulpn

# Show specific port
ss -tulpn | grep :80
```

### iptables - Firewall Management
```bash
# List rules
sudo iptables -L
sudo iptables -L -n -v

# Allow incoming on port
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Block IP address
sudo iptables -A INPUT -s 192.168.1.100 -j DROP

# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Set default policies
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Delete rule by number
sudo iptables -D INPUT 3

# Flush all rules
sudo iptables -F

# Save rules (Ubuntu/Debian)
sudo iptables-save > /etc/iptables/rules.v4

# Restore rules
sudo iptables-restore < /etc/iptables/rules.v4
```

## Best Practices (Do's)

✅ **Use version control for scripts**
```bash
# Initialize git repo for scripts
git init ~/scripts
```

✅ **Test scripts in safe environment first**
```bash
# Use VM or container
docker run -it ubuntu bash
```

✅ **Use logging in cron jobs**
```bash
# Redirect output to log file
* * * * * /path/to/script.sh >> /var/log/myscript.log 2>&1
```

✅ **Monitor system resources regularly**
```bash
# Create monitoring script
#!/bin/bash
echo "=== $(date) ===" >> /var/log/system-mon.log
vmstat 1 5 >> /var/log/system-mon.log
iostat -x >> /var/log/system-mon.log
```

✅ **Use systemd for service management**
```bash
# Modern and reliable
sudo systemctl enable myservice
```

✅ **Set appropriate permissions on scripts**
```bash
chmod 750 script.sh  # rwxr-x---
chown root:admin script.sh
```

## Common Mistakes (Don'ts)

❌ **Don't run sed without backup**
```bash
# Risky
sed -i 's/old/new/g' important-file.txt

# Safe
sed -i.bak 's/old/new/g' important-file.txt
```

❌ **Don't hardcode paths in cron**
```bash
# Bad - PATH may be different in cron
* * * * * python script.py

# Good - use full paths
* * * * * /usr/bin/python /full/path/to/script.py
```

❌ **Don't ignore script exit codes**
```bash
# Bad
#!/bin/bash
backup_database
cleanup_temp

# Good
#!/bin/bash
set -e  # Exit on error
backup_database || { echo "Backup failed"; exit 1; }
cleanup_temp
```

❌ **Don't use root crontab unnecessarily**
```bash
# Run as specific user when possible
sudo crontab -u username -e
```

❌ **Don't forget to escape special characters in sed**
```bash
# Bad - / in path conflicts with sed delimiter
sed 's//old/path//new/path/g' file.txt

# Good - use different delimiter
sed 's|/old/path|/new/path|g' file.txt
```

❌ **Don't modify iptables without saving**
```bash
# Rules are lost on reboot if not saved
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
# Remember to save!
sudo iptables-save > /etc/iptables/rules.v4
```
