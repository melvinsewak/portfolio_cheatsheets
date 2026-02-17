# PowerShell Beginner Cheatsheet

## Basic Syntax and Cmdlets

### Understanding Cmdlets
Cmdlets (pronounced "command-lets") follow a Verb-Noun naming convention.
```powershell
# Get a list of running processes
Get-Process

# Get files and folders
Get-ChildItem

# Get services
Get-Service

# Stop a process
Stop-Process -Name notepad

# Start a service
Start-Service -Name Spooler

# Common Verbs: Get, Set, New, Remove, Start, Stop, Restart, Test
```

### Aliases
```powershell
# PowerShell has built-in aliases for common commands
ls        # Alias for Get-ChildItem
dir       # Alias for Get-ChildItem
cd        # Alias for Set-Location
pwd       # Alias for Get-Location
cat       # Alias for Get-Content
rm        # Alias for Remove-Item
cp        # Alias for Copy-Item
mv        # Alias for Move-Item

# Find aliases
Get-Alias
Get-Alias -Name ls
Get-Alias -Definition Get-ChildItem
```

## Variables and Data Types

### Variable Declaration
```powershell
# Variables start with $
$name = "Alice"
$age = 25
$price = 19.99
$isActive = $true

# Multiple assignment
$x, $y, $z = 1, 2, 3

# Automatic variables (built-in)
$PSVersionTable    # PowerShell version info
$HOME              # User's home directory
$PWD               # Current directory
$_                 # Current pipeline object
```

### Data Types
```powershell
# String
$text = "Hello World"
$multiline = @"
This is a
multiline string
"@

# Integer
$count = 42

# Double/Float
$pi = 3.14159

# Boolean
$isTrue = $true
$isFalse = $false

# Array
$colors = @("Red", "Green", "Blue")

# Hash table
$person = @{
    Name = "Bob"
    Age  = 30
}

# DateTime
$now = Get-Date
$birthday = [DateTime]"1990-01-15"

# Check type
$age.GetType().Name          # Int32
$text.GetType().FullName     # System.String

# Type casting
[int]$number = "42"
[string]$text = 100
[DateTime]$date = "2024-01-01"
```

### String Manipulation
```powershell
# String concatenation
$firstName = "John"
$lastName = "Doe"
$fullName = $firstName + " " + $lastName

# String interpolation (double quotes)
$greeting = "Hello, $firstName!"
$message = "Today is $(Get-Date -Format 'yyyy-MM-dd')"

# String methods
$text = "  PowerShell  "
$text.ToUpper()              # "  POWERSHELL  "
$text.ToLower()              # "  powershell  "
$text.Trim()                 # "PowerShell"
$text.Replace("Shell", "")   # "  Power  "
$text.Length                 # 14
$text.Contains("Shell")      # True
$text.StartsWith("Power")    # False (has spaces)

# Single quotes (literal strings - no interpolation)
$literal = 'Hello, $name'    # Outputs: Hello, $name
```

## Operators

### Arithmetic Operators
```powershell
$a = 10
$b = 3

$a + $b    # 13 - Addition
$a - $b    # 7  - Subtraction
$a * $b    # 30 - Multiplication
$a / $b    # 3.333... - Division
$a % $b    # 1  - Modulus (remainder)

# Increment/Decrement
$x = 5
$x++       # 6
$x--       # 5
$x += 3    # 8
$x -= 2    # 6
$x *= 2    # 12
$x /= 3    # 4
```

### Comparison Operators
```powershell
# PowerShell comparison operators (case-insensitive by default)
$x = 10
$y = 20

$x -eq $y       # False - Equal
$x -ne $y       # True  - Not equal
$x -gt $y       # False - Greater than
$x -lt $y       # True  - Less than
$x -ge 10       # True  - Greater than or equal
$x -le 5        # False - Less than or equal

# Case-sensitive versions
$text = "Hello"
$text -ceq "hello"    # False (case-sensitive)
$text -ieq "hello"    # True  (case-insensitive, explicit)

# String matching
"PowerShell" -like "*Shell"      # True
"PowerShell" -notlike "Bash*"    # True
"test123" -match "\d+"           # True (regex)
"abc" -notmatch "\d+"            # True

# Containment
$array = 1, 2, 3, 4, 5
$array -contains 3               # True
$array -notcontains 10           # True
3 -in $array                     # True
10 -notin $array                 # True
```

### Logical Operators
```powershell
$a = $true
$b = $false

$a -and $b    # False - AND
$a -or $b     # True  - OR
-not $a       # False - NOT
!$a           # False - NOT (alternative)

# Practical example
$age = 25
$hasLicense = $true
if ($age -ge 18 -and $hasLicense) {
    Write-Output "Can drive"
}
```

## Control Flow

### If/Else Statements
```powershell
# Basic if
$temperature = 75
if ($temperature -gt 70) {
    Write-Output "It's warm"
}

# If-Else
$age = 17
if ($age -ge 18) {
    Write-Output "Adult"
}
else {
    Write-Output "Minor"
}

# If-ElseIf-Else
$score = 85
if ($score -ge 90) {
    Write-Output "Grade: A"
}
elseif ($score -ge 80) {
    Write-Output "Grade: B"
}
elseif ($score -ge 70) {
    Write-Output "Grade: C"
}
else {
    Write-Output "Grade: F"
}

# Multiple conditions
$age = 25
$country = "USA"
if ($age -ge 21 -and $country -eq "USA") {
    Write-Output "Can drink alcohol"
}
```

### Switch Statements
```powershell
# Basic switch
$day = "Monday"
switch ($day) {
    "Monday"    { Write-Output "Start of work week" }
    "Friday"    { Write-Output "End of work week" }
    "Saturday"  { Write-Output "Weekend!" }
    "Sunday"    { Write-Output "Weekend!" }
    default     { Write-Output "Midweek day" }
}

# Switch with multiple matches
$color = "Red"
switch ($color) {
    "Red"   { Write-Output "Stop" }
    "Yellow" { Write-Output "Caution" }
    "Green" { Write-Output "Go" }
}

# Switch with wildcard
$file = "document.txt"
switch -Wildcard ($file) {
    "*.txt"  { Write-Output "Text file" }
    "*.jpg"  { Write-Output "Image file" }
    "*.ps1"  { Write-Output "PowerShell script" }
}

# Switch with regex
$input = "User123"
switch -Regex ($input) {
    "^\d+$"        { Write-Output "Only numbers" }
    "^[A-Za-z]+$"  { Write-Output "Only letters" }
    "^User\d+$"    { Write-Output "Valid username" }
}
```

## Loops

### For Loop
```powershell
# Basic for loop
for ($i = 0; $i -lt 5; $i++) {
    Write-Output "Count: $i"
}

# For loop with custom increment
for ($i = 0; $i -le 10; $i += 2) {
    Write-Output $i  # 0, 2, 4, 6, 8, 10
}

# Nested for loops
for ($i = 1; $i -le 3; $i++) {
    for ($j = 1; $j -le 3; $j++) {
        Write-Output "i=$i, j=$j"
    }
}
```

### ForEach Loop
```powershell
# ForEach with array
$colors = @("Red", "Green", "Blue")
foreach ($color in $colors) {
    Write-Output "Color: $color"
}

# ForEach with range
foreach ($num in 1..5) {
    Write-Output $num
}

# ForEach with files
$files = Get-ChildItem -Path C:\Temp
foreach ($file in $files) {
    Write-Output "$($file.Name) - $($file.Length) bytes"
}

# ForEach with hash table
$person = @{
    Name = "Alice"
    Age  = 30
    City = "Seattle"
}
foreach ($key in $person.Keys) {
    Write-Output "$key : $($person[$key])"
}
```

### While Loop
```powershell
# While loop
$count = 0
while ($count -lt 5) {
    Write-Output "Count: $count"
    $count++
}

# While with condition
$password = ""
while ($password -ne "secret") {
    $password = Read-Host "Enter password"
}
Write-Output "Access granted!"

# Do-While (executes at least once)
$count = 0
do {
    Write-Output "Count: $count"
    $count++
} while ($count -lt 5)

# Do-Until (opposite of while)
$count = 0
do {
    Write-Output "Count: $count"
    $count++
} until ($count -ge 5)
```

### Loop Control
```powershell
# Break - exit loop
for ($i = 0; $i -lt 10; $i++) {
    if ($i -eq 5) {
        break
    }
    Write-Output $i  # 0, 1, 2, 3, 4
}

# Continue - skip current iteration
for ($i = 0; $i -lt 10; $i++) {
    if ($i % 2 -eq 0) {
        continue
    }
    Write-Output $i  # 1, 3, 5, 7, 9
}
```

## Arrays and Hash Tables

### Arrays
```powershell
# Creating arrays
$array1 = @()                           # Empty array
$array2 = @(1, 2, 3, 4, 5)             # Integer array
$array3 = "A", "B", "C"                 # String array
$array4 = 1..10                         # Range

# Accessing elements
$colors = @("Red", "Green", "Blue")
$colors[0]                              # "Red"
$colors[-1]                             # "Blue" (last element)
$colors[0..2]                           # First three elements

# Array properties
$colors.Length                          # 3
$colors.Count                           # 3

# Adding elements
$colors += "Yellow"
$colors = $colors + "Purple"

# Array methods
$numbers = @(5, 2, 8, 1, 9)
$numbers -contains 5                    # True
$numbers.IndexOf(8)                     # 2
[Array]::Reverse($numbers)
[Array]::Sort($numbers)

# Multidimensional arrays
$matrix = @(
    @(1, 2, 3),
    @(4, 5, 6),
    @(7, 8, 9)
)
$matrix[1][2]                           # 6

# Array manipulation
$filtered = $numbers | Where-Object { $_ -gt 3 }
$doubled = $numbers | ForEach-Object { $_ * 2 }
```

### Hash Tables
```powershell
# Creating hash tables
$empty = @{}
$person = @{
    Name = "Alice"
    Age  = 30
    City = "Seattle"
}

# Accessing values
$person["Name"]                         # "Alice"
$person.Age                             # 30

# Adding/updating entries
$person["Email"] = "alice@example.com"
$person.Phone = "555-1234"
$person["Age"] = 31                     # Update

# Removing entries
$person.Remove("Phone")

# Hash table properties
$person.Count                           # Number of entries
$person.Keys                            # All keys
$person.Values                          # All values

# Checking for keys
$person.ContainsKey("Name")             # True
$person.ContainsKey("Phone")            # False

# Iterating
foreach ($key in $person.Keys) {
    Write-Output "$key = $($person[$key])"
}

# Ordered hash tables (maintains insertion order)
$ordered = [ordered]@{
    First  = 1
    Second = 2
    Third  = 3
}
```

## Basic Functions

### Function Declaration
```powershell
# Simple function
function Get-Greeting {
    Write-Output "Hello, World!"
}
Get-Greeting

# Function with parameters
function Get-CustomGreeting {
    param($Name)
    Write-Output "Hello, $Name!"
}
Get-CustomGreeting -Name "Alice"

# Function with multiple parameters
function Add-Numbers {
    param(
        $Number1,
        $Number2
    )
    return $Number1 + $Number2
}
$result = Add-Numbers -Number1 10 -Number2 20

# Function with default parameter values
function Get-Welcome {
    param(
        $Name = "Guest"
    )
    Write-Output "Welcome, $Name!"
}
Get-Welcome              # "Welcome, Guest!"
Get-Welcome -Name "Bob"  # "Welcome, Bob!"

# Function with typed parameters
function Multiply {
    param(
        [int]$X,
        [int]$Y
    )
    return $X * $Y
}
```

### Return Values
```powershell
# Explicit return
function Get-Sum {
    param($a, $b)
    return $a + $b
}

# Implicit return (outputs anything not captured)
function Get-Numbers {
    1
    2
    3
}
$nums = Get-Numbers  # $nums = @(1, 2, 3)

# Return multiple values
function Get-Stats {
    param($Numbers)
    $min = ($Numbers | Measure-Object -Minimum).Minimum
    $max = ($Numbers | Measure-Object -Maximum).Maximum
    return $min, $max
}
$minimum, $maximum = Get-Stats -Numbers @(1, 5, 3, 9, 2)
```

## Working with Files and Folders

### Navigation
```powershell
# Get current location
Get-Location
pwd

# Change directory
Set-Location -Path C:\Users
cd C:\Users

# Go to parent directory
Set-Location ..
cd ..

# Go to home directory
Set-Location ~
cd ~
```

### Listing Files
```powershell
# List files and folders
Get-ChildItem
Get-ChildItem -Path C:\Temp
ls
dir

# List only files
Get-ChildItem -File

# List only directories
Get-ChildItem -Directory

# Recursive listing
Get-ChildItem -Recurse

# Filter by extension
Get-ChildItem -Filter *.txt
Get-ChildItem -Path C:\Logs -Filter *.log

# Include hidden files
Get-ChildItem -Force
```

### Creating Files and Folders
```powershell
# Create new directory
New-Item -Path C:\Temp\NewFolder -ItemType Directory
mkdir C:\Temp\NewFolder

# Create new file
New-Item -Path C:\Temp\test.txt -ItemType File

# Create file with content
"Hello World" | Out-File -FilePath C:\Temp\greeting.txt
Set-Content -Path C:\Temp\data.txt -Value "Initial content"
```

### Reading Files
```powershell
# Read entire file
Get-Content -Path C:\Temp\test.txt
cat C:\Temp\test.txt

# Read first N lines
Get-Content -Path C:\Temp\test.txt -TotalCount 10

# Read last N lines
Get-Content -Path C:\Temp\test.txt -Tail 10

# Read file as single string
Get-Content -Path C:\Temp\test.txt -Raw
```

### Writing to Files
```powershell
# Overwrite file
"New content" | Out-File -FilePath C:\Temp\test.txt
Set-Content -Path C:\Temp\test.txt -Value "New content"

# Append to file
"Additional line" | Out-File -FilePath C:\Temp\test.txt -Append
Add-Content -Path C:\Temp\test.txt -Value "Additional line"

# Write multiple lines
$lines = @("Line 1", "Line 2", "Line 3")
$lines | Out-File -FilePath C:\Temp\test.txt
```

### Copying, Moving, and Deleting
```powershell
# Copy file
Copy-Item -Path C:\Temp\test.txt -Destination C:\Backup\test.txt
cp C:\Temp\test.txt C:\Backup\

# Copy folder (recursive)
Copy-Item -Path C:\Temp\Folder1 -Destination C:\Backup\Folder1 -Recurse

# Move file
Move-Item -Path C:\Temp\test.txt -Destination C:\Archive\test.txt
mv C:\Temp\test.txt C:\Archive\

# Rename file
Rename-Item -Path C:\Temp\old.txt -NewName new.txt

# Delete file
Remove-Item -Path C:\Temp\test.txt
rm C:\Temp\test.txt

# Delete folder (recursive)
Remove-Item -Path C:\Temp\OldFolder -Recurse -Force

# Delete with confirmation
Remove-Item -Path C:\Temp\test.txt -Confirm
```

### File and Folder Properties
```powershell
# Get file/folder info
$file = Get-Item -Path C:\Temp\test.txt
$file.Name
$file.Length              # File size in bytes
$file.LastWriteTime
$file.CreationTime
$file.Extension

# Check if exists
Test-Path -Path C:\Temp\test.txt     # True/False

# Get file size
(Get-Item C:\Temp\test.txt).Length

# Get folder size
$size = (Get-ChildItem -Path C:\Temp -Recurse | Measure-Object -Property Length -Sum).Sum
```

## Pipeline Basics

### Understanding the Pipeline
```powershell
# Pipeline passes objects, not text
Get-Process | Where-Object CPU -gt 10 | Select-Object Name, CPU

# Each cmdlet receives objects from previous cmdlet
Get-ChildItem | Where-Object Length -gt 1MB | Sort-Object Length -Descending

# Pipeline variable $_
Get-Process | Where-Object { $_.WorkingSet -gt 100MB }
```

### Common Pipeline Commands
```powershell
# Where-Object - Filter objects
Get-Service | Where-Object Status -eq "Running"
Get-Process | Where-Object { $_.CPU -gt 100 }

# Select-Object - Select properties
Get-Process | Select-Object Name, CPU, ID
Get-Process | Select-Object -First 5
Get-Process | Select-Object -Last 3

# Sort-Object - Sort objects
Get-Process | Sort-Object CPU -Descending
Get-ChildItem | Sort-Object Length

# ForEach-Object - Perform action on each object
Get-Process | ForEach-Object { $_.Name.ToUpper() }
1..10 | ForEach-Object { $_ * 2 }

# Measure-Object - Calculate statistics
Get-ChildItem | Measure-Object -Property Length -Sum -Average
1..100 | Measure-Object -Sum -Average -Maximum -Minimum

# Group-Object - Group objects
Get-Process | Group-Object -Property ProcessName
Get-Service | Group-Object -Property Status
```

### Output Commands
```powershell
# Write-Output - Standard output
Write-Output "Hello, World!"

# Write-Host - Direct to console (doesn't go through pipeline)
Write-Host "Status: Complete" -ForegroundColor Green

# Write-Error - Error message
Write-Error "Something went wrong!"

# Write-Warning - Warning message
Write-Warning "This is a warning"

# Write-Verbose - Verbose output (shown with -Verbose)
Write-Verbose "Detailed information" -Verbose

# Format output
Get-Process | Format-Table Name, CPU, Id
Get-Process | Format-List Name, CPU, WorkingSet
Get-Process | Format-Wide Name -Column 3
```

## Getting Help

### Get-Help
```powershell
# Get help for a cmdlet
Get-Help Get-Process

# Get detailed help
Get-Help Get-Process -Detailed

# Get full help (with examples)
Get-Help Get-Process -Full

# Get examples only
Get-Help Get-Process -Examples

# Get online help
Get-Help Get-Process -Online

# Update help files
Update-Help

# Search help
Get-Help *process*
Get-Help about_*
```

### Get-Command
```powershell
# Find commands
Get-Command

# Find commands by name
Get-Command *Process*

# Find commands by verb
Get-Command -Verb Get

# Find commands by noun
Get-Command -Noun Service

# Find commands in a module
Get-Command -Module Microsoft.PowerShell.Management

# Get command syntax
Get-Command Get-Process -Syntax
```

### Get-Member
```powershell
# See properties and methods of an object
Get-Process | Get-Member
Get-Date | Get-Member

# See only properties
Get-Process | Get-Member -MemberType Property

# See only methods
Get-Process | Get-Member -MemberType Method

# Example usage
$date = Get-Date
$date | Get-Member
$date.AddDays(7)
$date.ToString("yyyy-MM-dd")
```

## Best Practices (Do's)

✅ **Use full cmdlet names in scripts (not aliases)**
```powershell
# Good - clear and explicit
Get-ChildItem -Path C:\Temp

# Avoid in scripts - unclear
ls C:\Temp
```

✅ **Use parameter names for clarity**
```powershell
# Good - clear intent
Get-Process -Name "notepad"

# Less clear
Get-Process notepad
```

✅ **Use approved verbs for functions**
```powershell
# Good - follows convention
function Get-UserInfo { }
function Set-Configuration { }
function Remove-OldFiles { }

# Bad - non-standard verbs
function Fetch-UserInfo { }
function Configure-Settings { }
function Delete-OldFiles { }
```

✅ **Use descriptive variable names**
```powershell
# Good
$userName = "Alice"
$maximumRetries = 3

# Bad
$u = "Alice"
$x = 3
```

✅ **Use proper comparison operators**
```powershell
# Good - PowerShell style
if ($value -eq 10) { }
if ($name -like "A*") { }

# Avoid - not PowerShell style
if ($value = 10) { }     # Assignment, not comparison!
if ($value == 10) { }    # Wrong operator
```

✅ **Use Write-Output for function returns**
```powershell
# Good
function Get-Result {
    Write-Output "Result"
}

# Also good
function Get-Result {
    return "Result"
}
```

✅ **Use Test-Path before file operations**
```powershell
# Good
if (Test-Path -Path C:\Temp\file.txt) {
    Remove-Item -Path C:\Temp\file.txt
}

# Risky - may fail if file doesn't exist
Remove-Item -Path C:\Temp\file.txt
```

## Common Mistakes (Don'ts)

❌ **Don't use assignment (=) instead of comparison (-eq)**
```powershell
# Wrong - assigns 10 to $x
if ($x = 10) { }

# Correct - compares $x to 10
if ($x -eq 10) { }
```

❌ **Don't forget $ for variables**
```powershell
# Wrong
name = "Alice"

# Correct
$name = "Alice"
```

❌ **Don't use Write-Host for output in functions**
```powershell
# Bad - doesn't pass through pipeline
function Get-Data {
    Write-Host "Data"
}

# Good - can be piped
function Get-Data {
    Write-Output "Data"
}
```

❌ **Don't ignore errors**
```powershell
# Risky
Get-Content -Path C:\NonExistent.txt

# Better - handle potential errors
if (Test-Path -Path C:\NonExistent.txt) {
    Get-Content -Path C:\NonExistent.txt
}
```

❌ **Don't use positional parameters without clarity**
```powershell
# Unclear
Get-ChildItem C:\Temp *.txt $true

# Clear
Get-ChildItem -Path C:\Temp -Filter *.txt -Recurse
```

❌ **Don't mix quote types inappropriately**
```powershell
# Wrong - variable not expanded
$name = 'Alice'
Write-Output 'Hello, $name'  # Output: Hello, $name

# Correct
$name = "Alice"
Write-Output "Hello, $name"  # Output: Hello, Alice
```

❌ **Don't forget -Recurse when copying folders**
```powershell
# Incomplete - only copies folder structure
Copy-Item -Path C:\Source -Destination C:\Dest

# Complete - copies all contents
Copy-Item -Path C:\Source -Destination C:\Dest -Recurse
```

❌ **Don't use aliases in production scripts**
```powershell
# Bad - unclear, may not work on all systems
ls | ? { $_.Length -gt 1MB } | select Name

# Good - clear and portable
Get-ChildItem | Where-Object { $_.Length -gt 1MB } | Select-Object Name
```
