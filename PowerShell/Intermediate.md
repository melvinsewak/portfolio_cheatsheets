# PowerShell Intermediate Cheatsheet

## Advanced Functions and Parameters

### Advanced Function Structure
```powershell
function Get-UserData {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory = $true, Position = 0)]
        [string]$UserName,
        
        [Parameter(Mandatory = $false)]
        [int]$Age = 0,
        
        [Parameter()]
        [switch]$IncludeDetails
    )
    
    begin {
        Write-Verbose "Starting Get-UserData function"
    }
    
    process {
        Write-Output "Processing user: $UserName"
        if ($IncludeDetails) {
            Write-Output "Age: $Age"
        }
    }
    
    end {
        Write-Verbose "Completed Get-UserData function"
    }
}

# Usage
Get-UserData -UserName "Alice" -Age 30 -IncludeDetails -Verbose
```

### Parameter Attributes
```powershell
function Set-Configuration {
    [CmdletBinding()]
    param(
        # Mandatory parameter
        [Parameter(Mandatory = $true)]
        [string]$ConfigName,
        
        # Parameter with validation
        [ValidateNotNullOrEmpty()]
        [string]$Value,
        
        # Validate against a set of values
        [ValidateSet("Development", "Staging", "Production")]
        [string]$Environment,
        
        # Validate range
        [ValidateRange(1, 100)]
        [int]$Timeout,
        
        # Validate pattern (regex)
        [ValidatePattern("^\d{3}-\d{2}-\d{4}$")]
        [string]$SSN,
        
        # Validate script
        [ValidateScript({ $_ -ge 0 -and $_ -le 100 })]
        [int]$Percentage,
        
        # Validate length
        [ValidateLength(1, 50)]
        [string]$Name,
        
        # Switch parameter
        [switch]$Force,
        
        # Accept pipeline input
        [Parameter(ValueFromPipeline = $true)]
        [string]$InputObject
    )
    
    process {
        Write-Output "Config: $ConfigName = $Value"
        Write-Output "Environment: $Environment"
    }
}
```

### Parameter Sets
```powershell
function Get-Data {
    [CmdletBinding(DefaultParameterSetName = "ByName")]
    param(
        [Parameter(ParameterSetName = "ByName", Mandatory = $true)]
        [string]$Name,
        
        [Parameter(ParameterSetName = "ById", Mandatory = $true)]
        [int]$Id,
        
        [Parameter(ParameterSetName = "All")]
        [switch]$All
    )
    
    switch ($PSCmdlet.ParameterSetName) {
        "ByName" { Write-Output "Getting data by name: $Name" }
        "ById"   { Write-Output "Getting data by ID: $Id" }
        "All"    { Write-Output "Getting all data" }
    }
}

# Usage
Get-Data -Name "Alice"
Get-Data -Id 123
Get-Data -All
```

### Advanced Parameter Features
```powershell
function Invoke-Process {
    [CmdletBinding(SupportsShouldProcess = $true, ConfirmImpact = "High")]
    param(
        [Parameter(Mandatory = $true)]
        [string]$ProcessName
    )
    
    if ($PSCmdlet.ShouldProcess($ProcessName, "Stop process")) {
        Stop-Process -Name $ProcessName
    }
}

# WhatIf and Confirm support
Invoke-Process -ProcessName "notepad" -WhatIf
Invoke-Process -ProcessName "notepad" -Confirm
```

## Pipeline and Filtering

### Where-Object (Filtering)
```powershell
# Basic filtering
Get-Process | Where-Object CPU -gt 100
Get-Process | Where-Object { $_.CPU -gt 100 }

# Multiple conditions
Get-Process | Where-Object { $_.CPU -gt 100 -and $_.WorkingSet -gt 100MB }

# Complex filtering
Get-Service | Where-Object { 
    $_.Status -eq "Running" -and $_.StartType -eq "Automatic" 
}

# Filter with -like
Get-Process | Where-Object Name -like "power*"

# Filter with -match (regex)
Get-ChildItem | Where-Object Name -match "\d{4}"

# Comparison operators in Where-Object
Get-ChildItem | Where-Object Length -gt 1MB
Get-Process | Where-Object StartTime -gt (Get-Date).AddHours(-1)
```

### Select-Object (Property Selection)
```powershell
# Select specific properties
Get-Process | Select-Object Name, CPU, Id

# Select first N objects
Get-Process | Select-Object -First 10

# Select last N objects
Get-Process | Select-Object -Last 5

# Skip and select
Get-Process | Select-Object -Skip 5 -First 10

# Select unique objects
Get-Process | Select-Object ProcessName -Unique

# Create calculated properties
Get-Process | Select-Object Name, 
    @{Name="CPUTime"; Expression={$_.CPU}},
    @{Name="MemoryMB"; Expression={$_.WorkingSet / 1MB}}

# Rename properties
Get-Process | Select-Object @{Name="ProcessName"; Expression={$_.Name}}, CPU

# Expand property
Get-Process | Select-Object -ExpandProperty Name
```

### ForEach-Object (Pipeline Processing)
```powershell
# Process each object in pipeline
Get-ChildItem | ForEach-Object { $_.FullName }

# Using $_ (current object)
1..10 | ForEach-Object { $_ * 2 }

# Multiple statements
Get-Process | ForEach-Object {
    $name = $_.Name
    $cpu = $_.CPU
    Write-Output "$name uses $cpu CPU"
}

# Begin, Process, End blocks
Get-ChildItem | ForEach-Object -Begin {
    $total = 0
} -Process {
    $total += $_.Length
} -End {
    Write-Output "Total size: $total bytes"
}

# Method invocation
Get-Service | ForEach-Object { $_.Stop() }
"hello", "world" | ForEach-Object { $_.ToUpper() }
```

### Advanced Pipeline Techniques
```powershell
# Chain multiple pipeline commands
Get-Process | 
    Where-Object CPU -gt 10 | 
    Sort-Object CPU -Descending | 
    Select-Object -First 5 | 
    Format-Table Name, CPU

# Group and measure
Get-Process | 
    Group-Object -Property ProcessName | 
    Select-Object Name, Count | 
    Sort-Object Count -Descending

# Tee-Object (split pipeline)
Get-Process | 
    Tee-Object -FilePath C:\Temp\processes.txt | 
    Where-Object CPU -gt 10

# Out-GridView (interactive filtering)
Get-Process | Out-GridView -PassThru | Stop-Process

# Compare-Object
$old = Get-Content C:\Temp\old.txt
$new = Get-Content C:\Temp\new.txt
Compare-Object $old $new
```

## Objects and Properties

### Working with Objects
```powershell
# Get object properties
$process = Get-Process -Name "powershell" | Select-Object -First 1
$process.Name
$process.CPU
$process.Id

# Get all properties
$process | Get-Member -MemberType Property

# Get methods
$process | Get-Member -MemberType Method

# Call methods
$date = Get-Date
$date.AddDays(7)
$date.ToString("yyyy-MM-dd")
```

### Creating Custom Objects
```powershell
# PSCustomObject
$person = [PSCustomObject]@{
    Name = "Alice"
    Age  = 30
    City = "Seattle"
}

# Access properties
$person.Name
$person.Age

# Add properties
$person | Add-Member -MemberType NoteProperty -Name "Email" -Value "alice@example.com"

# Create multiple objects
$users = @(
    [PSCustomObject]@{Name="Alice"; Age=30},
    [PSCustomObject]@{Name="Bob"; Age=25},
    [PSCustomObject]@{Name="Charlie"; Age=35}
)

# Process collection
$users | Where-Object Age -gt 28
$users | Select-Object Name
```

### Object Manipulation
```powershell
# Select and expand
$processes = Get-Process | Select-Object Name, Id, @{
    Name = "MemoryMB"
    Expression = { [math]::Round($_.WorkingSet / 1MB, 2) }
}

# Sort objects
$processes | Sort-Object MemoryMB -Descending

# Group objects
Get-Service | Group-Object Status

# Filter and transform
Get-ChildItem | 
    Where-Object Extension -eq ".txt" | 
    Select-Object Name, @{
        Name = "SizeKB"
        Expression = { [math]::Round($_.Length / 1KB, 2) }
    }
```

### Nested Objects
```powershell
# Create nested structure
$company = [PSCustomObject]@{
    Name = "Acme Corp"
    Employees = @(
        [PSCustomObject]@{Name="Alice"; Role="Developer"},
        [PSCustomObject]@{Name="Bob"; Role="Manager"}
    )
    Location = [PSCustomObject]@{
        City = "Seattle"
        State = "WA"
    }
}

# Access nested properties
$company.Name
$company.Employees[0].Name
$company.Location.City

# Modify nested objects
$company.Employees += [PSCustomObject]@{Name="Charlie"; Role="Designer"}
```

## Working with Modules

### Finding and Installing Modules
```powershell
# Find available modules
Get-Module -ListAvailable

# Find modules in PowerShell Gallery
Find-Module -Name "Az"
Find-Module -Tag "Azure"

# Install module from gallery
Install-Module -Name "Az" -Scope CurrentUser

# Install specific version
Install-Module -Name "Az" -RequiredVersion "10.0.0"

# Update module
Update-Module -Name "Az"

# Uninstall module
Uninstall-Module -Name "Az"
```

### Importing and Using Modules
```powershell
# Import module
Import-Module -Name "Az"

# Import with prefix (avoid naming conflicts)
Import-Module -Name "Microsoft.PowerShell.Management" -Prefix "Mgmt"

# List imported modules
Get-Module

# Remove module from session
Remove-Module -Name "Az"

# Get commands from module
Get-Command -Module "Az"

# Get module information
Get-Module -Name "Az" -ListAvailable | Select-Object Name, Version, Path
```

### Creating Simple Modules
```powershell
# Create a .psm1 file: MyModule.psm1
# MyModule.psm1 content:

function Get-Greeting {
    param($Name = "Guest")
    return "Hello, $Name!"
}

function Get-Farewell {
    param($Name = "Guest")
    return "Goodbye, $Name!"
}

# Export functions
Export-ModuleMember -Function Get-Greeting, Get-Farewell

# Use the module
Import-Module .\MyModule.psm1
Get-Greeting -Name "Alice"
```

### Module Paths
```powershell
# View module paths
$env:PSModulePath -split ';'

# Add custom module path
$env:PSModulePath += ";C:\MyModules"

# User module path
$home\Documents\PowerShell\Modules

# System module path
C:\Program Files\PowerShell\Modules
```

## Error Handling

### Try/Catch/Finally
```powershell
# Basic try/catch
try {
    Get-Content -Path "C:\NonExistent.txt" -ErrorAction Stop
}
catch {
    Write-Output "Error occurred: $($_.Exception.Message)"
}

# Catch specific exceptions
try {
    $result = 10 / 0
}
catch [System.DivideByZeroException] {
    Write-Output "Cannot divide by zero!"
}
catch {
    Write-Output "An unexpected error occurred"
}

# Try/Catch/Finally
try {
    $file = [System.IO.File]::OpenRead("C:\Temp\data.txt")
    # Process file
}
catch {
    Write-Error "Failed to open file: $($_.Exception.Message)"
}
finally {
    if ($file) {
        $file.Close()
        Write-Output "File closed"
    }
}
```

### Error Action Preference
```powershell
# Stop on error
Get-Content -Path "C:\NonExistent.txt" -ErrorAction Stop

# Continue on error (default)
Get-Content -Path "C:\NonExistent.txt" -ErrorAction Continue

# Silently continue
Get-Content -Path "C:\NonExistent.txt" -ErrorAction SilentlyContinue

# Inquire (prompt user)
Get-Content -Path "C:\NonExistent.txt" -ErrorAction Inquire

# Set global error action preference
$ErrorActionPreference = "Stop"
```

### Error Variables
```powershell
# Automatic error variable
try {
    Get-Content -Path "C:\NonExistent.txt" -ErrorAction Stop
}
catch {
    Write-Output "Error message: $($_.Exception.Message)"
    Write-Output "Error type: $($_.Exception.GetType().FullName)"
    Write-Output "Stack trace: $($_.ScriptStackTrace)"
}

# Error record properties
$Error[0]                          # Most recent error
$Error[0].Exception.Message
$Error[0].CategoryInfo
$Error[0].FullyQualifiedErrorId

# Clear error variable
$Error.Clear()
```

### Custom Error Handling
```powershell
function Get-FileContent {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory = $true)]
        [string]$Path
    )
    
    try {
        if (-not (Test-Path -Path $Path)) {
            throw "File not found: $Path"
        }
        
        $content = Get-Content -Path $Path -ErrorAction Stop
        return $content
    }
    catch [System.IO.IOException] {
        Write-Error "IO Error reading file: $($_.Exception.Message)"
    }
    catch {
        Write-Error "Unexpected error: $($_.Exception.Message)"
        throw
    }
}
```

### Throw and Terminating Errors
```powershell
# Throw simple error
if ($value -lt 0) {
    throw "Value cannot be negative"
}

# Throw with error record
$errorRecord = [System.Management.Automation.ErrorRecord]::new(
    [System.Exception]::new("Custom error"),
    "CustomError",
    [System.Management.Automation.ErrorCategory]::InvalidOperation,
    $value
)
throw $errorRecord

# Write-Error (non-terminating)
Write-Error "This is a non-terminating error"

# Convert to terminating error
Write-Error "This will stop" -ErrorAction Stop
```

## Regular Expressions

### Basic Pattern Matching
```powershell
# -match operator
"PowerShell" -match "Shell"              # True
"test123" -match "\d+"                   # True
"abc" -match "^\w{3}$"                   # True

# Capture matches
"test123" -match "(\w+)(\d+)"
$Matches[0]                              # "test123" (full match)
$Matches[1]                              # "test" (first group)
$Matches[2]                              # "123" (second group)

# Case-insensitive (default)
"PowerShell" -match "powershell"         # True

# Case-sensitive
"PowerShell" -cmatch "powershell"        # False
```

### Common Regex Patterns
```powershell
# Email validation
$email = "user@example.com"
$email -match "^[\w\.-]+@[\w\.-]+\.\w+$"

# Phone number
$phone = "555-123-4567"
$phone -match "^\d{3}-\d{3}-\d{4}$"

# IP address
$ip = "192.168.1.1"
$ip -match "^(\d{1,3}\.){3}\d{1,3}$"

# Date (YYYY-MM-DD)
$date = "2024-01-15"
$date -match "^\d{4}-\d{2}-\d{2}$"

# URL
$url = "https://www.example.com"
$url -match "^https?://(www\.)?[\w\.-]+\.\w+$"

# Extract numbers from string
"Total: 123 items" -match "\d+"
$Matches[0]                              # "123"
```

### Select-String (grep equivalent)
```powershell
# Search in file
Select-String -Path "C:\Temp\log.txt" -Pattern "error"

# Search in multiple files
Select-String -Path "C:\Logs\*.log" -Pattern "error"

# Case-sensitive search
Select-String -Path "C:\Temp\log.txt" -Pattern "ERROR" -CaseSensitive

# Get context lines
Select-String -Path "C:\Temp\log.txt" -Pattern "error" -Context 2, 2

# Return only matches
Select-String -Path "C:\Temp\log.txt" -Pattern "\d{3}-\d{2}-\d{4}" | 
    ForEach-Object { $_.Matches.Value }

# Search through pipeline
Get-Content "C:\Temp\log.txt" | Select-String -Pattern "error"
```

### Regex Replace
```powershell
# -replace operator
"Hello World" -replace "World", "PowerShell"     # "Hello PowerShell"

# Remove digits
"test123abc456" -replace "\d+", ""               # "testabc"

# Replace with pattern groups
"John Doe" -replace "(\w+) (\w+)", '$2, $1'      # "Doe, John"

# Multiple replacements
$text = "Hello World"
$text -replace "Hello", "Hi" -replace "World", "PowerShell"

# Regex.Replace method
[regex]::Replace("test123", "\d+", "456")        # "test456"
```

### Advanced Regex
```powershell
# Named capture groups
$text = "Name: Alice, Age: 30"
if ($text -match "Name: (?<name>\w+), Age: (?<age>\d+)") {
    $Matches.name                        # "Alice"
    $Matches.age                         # "30"
}

# Lookahead
"password123" -match "\w+(?=\d+)"        # Matches "password" before digits

# Lookbehind
"$123.45" -match "(?<=\$)\d+\.\d+"       # Matches "123.45" after $

# Non-greedy matching
"<div>Content</div>" -match "<div>(.*?)</div>"
$Matches[1]                              # "Content"

# Split with regex
"one,two;three:four" -split "[,;:]"      # @("one", "two", "three", "four")
```

## Working with Data Formats

### CSV Files
```powershell
# Import CSV
$data = Import-Csv -Path "C:\Temp\data.csv"

# Access data
$data[0].Name
$data | Where-Object Age -gt 25

# Export to CSV
$users = @(
    [PSCustomObject]@{Name="Alice"; Age=30; City="Seattle"},
    [PSCustomObject]@{Name="Bob"; Age=25; City="Portland"}
)
$users | Export-Csv -Path "C:\Temp\users.csv" -NoTypeInformation

# Append to CSV
$newUser = [PSCustomObject]@{Name="Charlie"; Age=35; City="Denver"}
$newUser | Export-Csv -Path "C:\Temp\users.csv" -Append -NoTypeInformation

# Custom delimiter
Import-Csv -Path "C:\Temp\data.txt" -Delimiter "`t"

# Select specific columns
Import-Csv -Path "C:\Temp\data.csv" | Select-Object Name, Age
```

### JSON Files
```powershell
# Import JSON
$json = Get-Content -Path "C:\Temp\data.json" -Raw | ConvertFrom-Json

# Access properties
$json.Name
$json.Users[0].Email

# Export to JSON
$data = [PSCustomObject]@{
    Name = "Project"
    Users = @(
        [PSCustomObject]@{Name="Alice"; Email="alice@example.com"},
        [PSCustomObject]@{Name="Bob"; Email="bob@example.com"}
    )
}
$data | ConvertTo-Json -Depth 10 | Out-File -FilePath "C:\Temp\output.json"

# Pretty print JSON
$data | ConvertTo-Json -Depth 10 | Out-File -FilePath "C:\Temp\output.json"

# Compress JSON (no formatting)
$data | ConvertTo-Json -Compress
```

### XML Files
```powershell
# Import XML
[xml]$xml = Get-Content -Path "C:\Temp\data.xml"

# Access elements
$xml.root.name
$xml.root.users.user[0].name

# Create XML
$xml = [xml]@"
<root>
    <user>
        <name>Alice</name>
        <age>30</age>
    </user>
</root>
"@

# Modify XML
$newUser = $xml.CreateElement("user")
$newUser.InnerText = "Bob"
$xml.root.AppendChild($newUser)

# Save XML
$xml.Save("C:\Temp\output.xml")

# XPath queries
$xml.SelectNodes("//user[@age>25]")

# Select-Xml
Select-Xml -Path "C:\Temp\data.xml" -XPath "//user"
```

## Remote Management (PSRemoting)

### Enable and Configure PSRemoting
```powershell
# Enable PSRemoting (run as Administrator)
Enable-PSRemoting -Force

# Check WinRM service
Get-Service WinRM

# Configure trusted hosts (for workgroup environments)
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "Server01,Server02"

# View trusted hosts
Get-Item WSMan:\localhost\Client\TrustedHosts
```

### One-to-One Remoting
```powershell
# Enter remote session
Enter-PSSession -ComputerName Server01
Enter-PSSession -ComputerName Server01 -Credential (Get-Credential)

# Run commands on remote computer
Get-Process
Get-Service

# Exit remote session
Exit-PSSession

# Invoke command remotely
Invoke-Command -ComputerName Server01 -ScriptBlock {
    Get-Process | Where-Object CPU -gt 10
}

# Pass arguments to remote script
Invoke-Command -ComputerName Server01 -ScriptBlock {
    param($Name)
    Get-Process -Name $Name
} -ArgumentList "powershell"
```

### One-to-Many Remoting
```powershell
# Run command on multiple computers
Invoke-Command -ComputerName Server01, Server02, Server03 -ScriptBlock {
    Get-Service -Name "Spooler"
}

# Store results
$results = Invoke-Command -ComputerName Server01, Server02 -ScriptBlock {
    Get-Process | Select-Object Name, CPU
}

# Filter results by computer
$results | Where-Object PSComputerName -eq "Server01"

# Run script file remotely
Invoke-Command -ComputerName Server01 -FilePath "C:\Scripts\Maintenance.ps1"
```

### Persistent Sessions
```powershell
# Create session
$session = New-PSSession -ComputerName Server01

# Use session multiple times
Invoke-Command -Session $session -ScriptBlock { Get-Process }
Invoke-Command -Session $session -ScriptBlock { Get-Service }

# Import remote module
Import-PSSession -Session $session -Module ActiveDirectory

# Copy files to remote computer
Copy-Item -Path "C:\Local\file.txt" -Destination "C:\Remote\" -ToSession $session

# Copy files from remote computer
Copy-Item -Path "C:\Remote\file.txt" -Destination "C:\Local\" -FromSession $session

# Close session
Remove-PSSession -Session $session

# Get all sessions
Get-PSSession
```

## Background Jobs

### Starting Jobs
```powershell
# Start background job
$job = Start-Job -ScriptBlock {
    Get-Process | Where-Object CPU -gt 10
}

# Job with name
Start-Job -Name "ProcessCheck" -ScriptBlock {
    Get-Process
}

# Job with arguments
Start-Job -ScriptBlock {
    param($Name)
    Get-Process -Name $Name
} -ArgumentList "powershell"

# Run script as job
Start-Job -FilePath "C:\Scripts\LongRunning.ps1"
```

### Managing Jobs
```powershell
# List all jobs
Get-Job

# Get specific job
Get-Job -Name "ProcessCheck"
Get-Job -Id 1

# Check job state
$job.State                         # Running, Completed, Failed

# Wait for job to complete
Wait-Job -Id 1
Wait-Job -Job $job

# Stop job
Stop-Job -Id 1

# Remove job
Remove-Job -Id 1

# Remove all completed jobs
Get-Job | Where-Object State -eq "Completed" | Remove-Job
```

### Retrieving Job Results
```powershell
# Receive job results
$results = Receive-Job -Id 1

# Keep results for multiple retrievals
Receive-Job -Id 1 -Keep

# Wait and receive
$results = Wait-Job -Id 1 | Receive-Job

# Automatic job result retrieval
Start-Job -ScriptBlock { Get-Process } | Wait-Job | Receive-Job
```

### Remote Jobs
```powershell
# Start job on remote computer
$job = Invoke-Command -ComputerName Server01 -ScriptBlock {
    Get-EventLog -LogName System -Newest 100
} -AsJob

# Multiple remote jobs
$job = Invoke-Command -ComputerName Server01, Server02, Server03 -ScriptBlock {
    Get-Service
} -AsJob

# Check and retrieve remote job results
Get-Job
Receive-Job -Job $job
```

## Best Practices (Do's)

✅ **Use [CmdletBinding()] for advanced functions**
```powershell
# Good - enables common parameters
function Get-Data {
    [CmdletBinding()]
    param($Name)
    # Function body
}

# Now supports -Verbose, -Debug, etc.
Get-Data -Name "Test" -Verbose
```

✅ **Use parameter validation**
```powershell
# Good - validates input
function Set-Age {
    param(
        [ValidateRange(0, 120)]
        [int]$Age
    )
    Write-Output "Age set to $Age"
}
```

✅ **Use Try/Catch for error handling**
```powershell
# Good - handles errors gracefully
try {
    $content = Get-Content -Path $Path -ErrorAction Stop
    # Process content
}
catch {
    Write-Error "Failed to read file: $($_.Exception.Message)"
}
```

✅ **Use -ErrorAction Stop for critical operations**
```powershell
# Good - ensures errors are catchable
try {
    Get-Content -Path $Path -ErrorAction Stop
}
catch {
    # Handle error
}
```

✅ **Use proper output commands**
```powershell
# Good - Write-Output for pipeline
function Get-Result {
    Write-Output $result
}

# Good - Write-Verbose for details
Write-Verbose "Processing file $fileName" -Verbose
```

✅ **Use calculated properties in Select-Object**
```powershell
# Good - clear and readable
Get-Process | Select-Object Name, @{
    Name = "MemoryMB"
    Expression = { [math]::Round($_.WorkingSet / 1MB, 2) }
}
```

✅ **Use proper regex escaping**
```powershell
# Good - escape special characters
$pattern = [regex]::Escape("file.txt")
"file.txt" -match $pattern
```

✅ **Close sessions and clean up jobs**
```powershell
# Good - clean up resources
$session = New-PSSession -ComputerName Server01
try {
    Invoke-Command -Session $session -ScriptBlock { }
}
finally {
    Remove-PSSession -Session $session
}
```

## Common Mistakes (Don'ts)

❌ **Don't forget -ErrorAction Stop in Try/Catch**
```powershell
# Bad - error not caught
try {
    Get-Content -Path "C:\NonExistent.txt"
}
catch {
    # This won't execute!
}

# Good
try {
    Get-Content -Path "C:\NonExistent.txt" -ErrorAction Stop
}
catch {
    Write-Error "File not found"
}
```

❌ **Don't use Write-Host for function output**
```powershell
# Bad - can't be piped
function Get-Data {
    Write-Host "Result"
}

# Good - can be piped
function Get-Data {
    Write-Output "Result"
}
```

❌ **Don't forget to specify depth in ConvertTo-Json**
```powershell
# Bad - may truncate nested objects
$data | ConvertTo-Json

# Good - specifies depth
$data | ConvertTo-Json -Depth 10
```

❌ **Don't leave jobs running**
```powershell
# Bad - jobs accumulate
Start-Job -ScriptBlock { Get-Process }

# Good - clean up
$job = Start-Job -ScriptBlock { Get-Process }
$job | Wait-Job | Receive-Job
Remove-Job -Job $job
```

❌ **Don't ignore parameter validation**
```powershell
# Bad - no validation
function Set-Percentage {
    param($Value)
    # What if $Value is 200?
}

# Good - validates range
function Set-Percentage {
    param(
        [ValidateRange(0, 100)]
        [int]$Value
    )
}
```

❌ **Don't forget -NoTypeInformation when exporting CSV**
```powershell
# Bad - includes type information header
Export-Csv -Path "data.csv"

# Good - clean CSV
Export-Csv -Path "data.csv" -NoTypeInformation
```

❌ **Don't use positional parameters in remote commands**
```powershell
# Bad - unclear
Invoke-Command -ComputerName Server01 -ScriptBlock { Get-Process "powershell" }

# Good - explicit
Invoke-Command -ComputerName Server01 -ScriptBlock { 
    Get-Process -Name "powershell" 
}
```

❌ **Don't forget to remove PSSessions**
```powershell
# Bad - sessions left open
$session = New-PSSession -ComputerName Server01
Invoke-Command -Session $session -ScriptBlock { }

# Good - clean up
$session = New-PSSession -ComputerName Server01
try {
    Invoke-Command -Session $session -ScriptBlock { }
}
finally {
    Remove-PSSession -Session $session
}
```
