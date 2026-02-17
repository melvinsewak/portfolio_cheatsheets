# PowerShell Advanced Cheatsheet

## Advanced Functions with Parameter Sets and Validation

### Complex Parameter Validation
```powershell
function Set-UserConfiguration {
    [CmdletBinding(SupportsShouldProcess = $true)]
    param(
        # Custom validation using script block
        [ValidateScript({
            if (Test-Path $_) {
                $true
            }
            else {
                throw "Path '$_' does not exist"
            }
        })]
        [string]$ConfigPath,
        
        # Multiple validations on same parameter
        [ValidateNotNullOrEmpty()]
        [ValidateLength(3, 50)]
        [ValidatePattern("^[A-Za-z0-9_]+$")]
        [string]$UserName,
        
        # Custom validation class
        [ValidateSet("Low", "Medium", "High", "Critical")]
        [string]$Priority = "Medium",
        
        # Validate count
        [ValidateCount(1, 10)]
        [string[]]$Tags,
        
        # Transformation attribute
        [Parameter()]
        [string]
        [ValidateScript({ $_ -is [string] })]
        $TransformedValue
    )
    
    if ($PSCmdlet.ShouldProcess($UserName, "Set configuration")) {
        Write-Verbose "Setting configuration for user: $UserName"
        # Implementation
    }
}
```

### Advanced Parameter Sets
```powershell
function Get-UserInformation {
    [CmdletBinding(DefaultParameterSetName = "ByUsername")]
    param(
        [Parameter(
            ParameterSetName = "ByUsername",
            Mandatory = $true,
            Position = 0,
            ValueFromPipeline = $true,
            ValueFromPipelineByPropertyName = $true
        )]
        [ValidateNotNullOrEmpty()]
        [string]$UserName,
        
        [Parameter(
            ParameterSetName = "ById",
            Mandatory = $true
        )]
        [ValidateRange(1, [int]::MaxValue)]
        [int]$UserId,
        
        [Parameter(
            ParameterSetName = "ByEmail",
            Mandatory = $true
        )]
        [ValidatePattern("^[\w\.-]+@[\w\.-]+\.\w+$")]
        [string]$Email,
        
        [Parameter(ParameterSetName = "ByUsername")]
        [Parameter(ParameterSetName = "ById")]
        [Parameter(ParameterSetName = "ByEmail")]
        [switch]$IncludeDetails,
        
        [Parameter(
            ParameterSetName = "All",
            Mandatory = $true
        )]
        [switch]$All
    )
    
    begin {
        Write-Verbose "Parameter Set: $($PSCmdlet.ParameterSetName)"
    }
    
    process {
        switch ($PSCmdlet.ParameterSetName) {
            "ByUsername" {
                Write-Output "Getting user by username: $UserName"
                if ($IncludeDetails) {
                    # Get detailed information
                }
            }
            "ById" {
                Write-Output "Getting user by ID: $UserId"
            }
            "ByEmail" {
                Write-Output "Getting user by email: $Email"
            }
            "All" {
                Write-Output "Getting all users"
            }
        }
    }
}

# Usage examples
Get-UserInformation -UserName "alice"
Get-UserInformation -UserId 123
Get-UserInformation -Email "alice@example.com"
Get-UserInformation -All
```

### Dynamic Parameters
```powershell
function Get-DynamicParameter {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory = $true)]
        [ValidateSet("Server", "Desktop", "Laptop")]
        [string]$ComputerType
    )
    
    DynamicParam {
        # Create parameter dictionary
        $paramDictionary = New-Object System.Management.Automation.RuntimeDefinedParameterDictionary
        
        # Define dynamic parameter based on ComputerType
        if ($ComputerType -eq "Server") {
            $attributes = New-Object System.Management.Automation.ParameterAttribute
            $attributes.Mandatory = $true
            
            $validateSet = New-Object System.Management.Automation.ValidateSetAttribute("Windows", "Linux")
            
            $attributeCollection = New-Object System.Collections.ObjectModel.Collection[System.Attribute]
            $attributeCollection.Add($attributes)
            $attributeCollection.Add($validateSet)
            
            $serverOS = New-Object System.Management.Automation.RuntimeDefinedParameter(
                "OperatingSystem",
                [string],
                $attributeCollection
            )
            
            $paramDictionary.Add("OperatingSystem", $serverOS)
        }
        
        return $paramDictionary
    }
    
    begin {
        # Access dynamic parameter
        if ($PSBoundParameters.ContainsKey("OperatingSystem")) {
            $os = $PSBoundParameters["OperatingSystem"]
            Write-Output "OS: $os"
        }
    }
    
    process {
        Write-Output "Computer Type: $ComputerType"
    }
}
```

## Script Modules and Manifest Files

### Creating a Script Module
```powershell
# MyModule.psm1

# Private functions (not exported)
function Get-InternalData {
    # Internal implementation
    return "Internal data"
}

# Public functions
function Get-ModuleData {
    <#
    .SYNOPSIS
    Gets module data
    
    .DESCRIPTION
    Retrieves data from the module with optional filtering
    
    .PARAMETER Name
    The name to filter by
    
    .PARAMETER IncludeDetails
    Include detailed information
    
    .EXAMPLE
    Get-ModuleData -Name "Test"
    
    .EXAMPLE
    Get-ModuleData -Name "Test" -IncludeDetails
    #>
    [CmdletBinding()]
    param(
        [Parameter(Mandatory = $true)]
        [string]$Name,
        
        [switch]$IncludeDetails
    )
    
    $data = Get-InternalData
    
    if ($IncludeDetails) {
        # Return detailed data
        [PSCustomObject]@{
            Name    = $Name
            Data    = $data
            Details = "Additional details"
        }
    }
    else {
        [PSCustomObject]@{
            Name = $Name
            Data = $data
        }
    }
}

function Set-ModuleConfiguration {
    [CmdletBinding(SupportsShouldProcess = $true)]
    param(
        [Parameter(Mandatory = $true)]
        [string]$ConfigName,
        
        [Parameter(Mandatory = $true)]
        [string]$Value
    )
    
    if ($PSCmdlet.ShouldProcess($ConfigName, "Set configuration")) {
        # Set configuration
        Write-Verbose "Setting $ConfigName = $Value"
    }
}

# Module variables
$ModuleVersion = "1.0.0"

# Export members
Export-ModuleMember -Function Get-ModuleData, Set-ModuleConfiguration -Variable ModuleVersion
```

### Creating Module Manifest
```powershell
# Create manifest file
New-ModuleManifest -Path ".\MyModule.psd1" `
    -ModuleVersion "1.0.0" `
    -Author "Your Name" `
    -Description "My PowerShell Module" `
    -PowerShellVersion "7.0" `
    -FunctionsToExport @("Get-ModuleData", "Set-ModuleConfiguration") `
    -VariablesToExport @("ModuleVersion") `
    -Tags @("Automation", "Utility") `
    -ProjectUri "https://github.com/username/mymodule" `
    -LicenseUri "https://github.com/username/mymodule/blob/main/LICENSE" `
    -RequiredModules @()

# Manifest file structure (MyModule.psd1)
@{
    ModuleVersion     = '1.0.0'
    GUID              = 'a1234567-89ab-cdef-0123-456789abcdef'
    Author            = 'Your Name'
    CompanyName       = 'Your Company'
    Copyright         = '(c) 2024. All rights reserved.'
    Description       = 'My PowerShell Module'
    PowerShellVersion = '7.0'
    
    # Script module file
    RootModule        = 'MyModule.psm1'
    
    # Functions to export
    FunctionsToExport = @('Get-ModuleData', 'Set-ModuleConfiguration')
    
    # Variables to export
    VariablesToExport = @('ModuleVersion')
    
    # Cmdlets to export
    CmdletsToExport   = @()
    
    # Aliases to export
    AliasesToExport   = @()
    
    # Module dependencies
    RequiredModules   = @()
    
    # Private data
    PrivateData       = @{
        PSData = @{
            Tags         = @('Automation', 'Utility')
            LicenseUri   = 'https://github.com/username/mymodule/blob/main/LICENSE'
            ProjectUri   = 'https://github.com/username/mymodule'
            ReleaseNotes = 'Initial release'
        }
    }
}
```

### Module Best Practices
```powershell
# Folder structure
# MyModule/
#   ├── MyModule.psd1 (manifest)
#   ├── MyModule.psm1 (module file)
#   ├── Public/
#   │   ├── Get-Something.ps1
#   │   └── Set-Something.ps1
#   ├── Private/
#   │   └── Get-InternalHelper.ps1
#   ├── Tests/
#   │   └── MyModule.Tests.ps1
#   └── README.md

# MyModule.psm1 with dot-sourcing
# Get public and private function files
$Public = @(Get-ChildItem -Path $PSScriptRoot\Public\*.ps1 -ErrorAction SilentlyContinue)
$Private = @(Get-ChildItem -Path $PSScriptRoot\Private\*.ps1 -ErrorAction SilentlyContinue)

# Dot source the files
foreach ($import in @($Public + $Private)) {
    try {
        . $import.FullName
    }
    catch {
        Write-Error "Failed to import function $($import.FullName): $_"
    }
}

# Export public functions
Export-ModuleMember -Function $Public.BaseName
```

## Classes and Object-Oriented PowerShell

### Basic Class Definition
```powershell
class Person {
    # Properties
    [string]$FirstName
    [string]$LastName
    [int]$Age
    
    # Constructor
    Person([string]$firstName, [string]$lastName, [int]$age) {
        $this.FirstName = $firstName
        $this.LastName = $lastName
        $this.Age = $age
    }
    
    # Default constructor
    Person() {
        $this.FirstName = ""
        $this.LastName = ""
        $this.Age = 0
    }
    
    # Method
    [string] GetFullName() {
        return "$($this.FirstName) $($this.LastName)"
    }
    
    # Method with parameters
    [void] HaveBirthday() {
        $this.Age++
    }
    
    # Static method
    static [Person] CreateDefault() {
        return [Person]::new("John", "Doe", 30)
    }
}

# Create instances
$person1 = [Person]::new("Alice", "Smith", 30)
$person2 = [Person]::new()
$person2.FirstName = "Bob"
$person2.LastName = "Jones"

# Call methods
$fullName = $person1.GetFullName()
$person1.HaveBirthday()

# Static method
$defaultPerson = [Person]::CreateDefault()
```

### Inheritance
```powershell
class Employee : Person {
    [string]$EmployeeId
    [string]$Department
    [decimal]$Salary
    
    # Constructor calling base constructor
    Employee([string]$firstName, [string]$lastName, [int]$age, 
             [string]$employeeId, [string]$department, [decimal]$salary) 
        : base($firstName, $lastName, $age) {
        $this.EmployeeId = $employeeId
        $this.Department = $department
        $this.Salary = $salary
    }
    
    # Override method
    [string] GetFullName() {
        return "$($this.FirstName) $($this.LastName) (ID: $($this.EmployeeId))"
    }
    
    # Additional methods
    [void] GiveRaise([decimal]$percentage) {
        $this.Salary *= (1 + $percentage / 100)
    }
    
    [string] ToString() {
        return "$($this.GetFullName()) - $($this.Department)"
    }
}

# Create employee
$employee = [Employee]::new("Alice", "Smith", 30, "EMP001", "IT", 75000)
$employee.GiveRaise(10)
```

### Properties and Validation
```powershell
class BankAccount {
    # Property with validation
    [ValidateRange(0, [double]::MaxValue)]
    [double]$Balance
    
    [ValidateNotNullOrEmpty()]
    [string]$AccountNumber
    
    # Hidden property
    hidden [string]$InternalId
    
    # Static property
    static [string]$BankName = "PowerShell Bank"
    
    # Constructor
    BankAccount([string]$accountNumber) {
        $this.AccountNumber = $accountNumber
        $this.Balance = 0
        $this.InternalId = [guid]::NewGuid().ToString()
    }
    
    # Method with validation
    [void] Deposit([double]$amount) {
        if ($amount -le 0) {
            throw "Deposit amount must be positive"
        }
        $this.Balance += $amount
    }
    
    [bool] Withdraw([double]$amount) {
        if ($amount -le 0) {
            throw "Withdrawal amount must be positive"
        }
        if ($amount -gt $this.Balance) {
            Write-Warning "Insufficient funds"
            return $false
        }
        $this.Balance -= $amount
        return $true
    }
    
    [string] GetAccountInfo() {
        return "Account: $($this.AccountNumber), Balance: $([math]::Round($this.Balance, 2))"
    }
}

# Usage
$account = [BankAccount]::new("ACC-12345")
$account.Deposit(1000)
$account.Withdraw(250)
Write-Output $account.GetAccountInfo()
```

### Enums
```powershell
# Define enum
enum Status {
    Pending = 0
    InProgress = 1
    Completed = 2
    Failed = 3
}

enum Priority {
    Low
    Medium
    High
    Critical
}

# Use enum in class
class Task {
    [string]$Name
    [Status]$Status
    [Priority]$Priority
    [DateTime]$CreatedDate
    
    Task([string]$name, [Priority]$priority) {
        $this.Name = $name
        $this.Status = [Status]::Pending
        $this.Priority = $priority
        $this.CreatedDate = Get-Date
    }
    
    [void] Start() {
        $this.Status = [Status]::InProgress
    }
    
    [void] Complete() {
        $this.Status = [Status]::Completed
    }
    
    [string] ToString() {
        return "$($this.Name) - Status: $($this.Status), Priority: $($this.Priority)"
    }
}

# Usage
$task = [Task]::new("Deploy Application", [Priority]::High)
$task.Start()
$task.Complete()
```

## Advanced Error Handling and Debugging

### Advanced Try/Catch Patterns
```powershell
function Invoke-SafeOperation {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory = $true)]
        [scriptblock]$Operation,
        
        [int]$MaxRetries = 3,
        [int]$RetryDelaySeconds = 2
    )
    
    $attempt = 0
    $success = $false
    $lastError = $null
    
    while (-not $success -and $attempt -lt $MaxRetries) {
        $attempt++
        try {
            Write-Verbose "Attempt $attempt of $MaxRetries"
            $result = & $Operation
            $success = $true
            return $result
        }
        catch [System.Net.WebException] {
            $lastError = $_
            Write-Warning "Network error on attempt $attempt : $($_.Exception.Message)"
            if ($attempt -lt $MaxRetries) {
                Start-Sleep -Seconds $RetryDelaySeconds
            }
        }
        catch [System.IO.IOException] {
            $lastError = $_
            Write-Warning "IO error on attempt $attempt : $($_.Exception.Message)"
            if ($attempt -lt $MaxRetries) {
                Start-Sleep -Seconds $RetryDelaySeconds
            }
        }
        catch {
            # Non-retriable error
            Write-Error "Fatal error: $($_.Exception.Message)"
            throw
        }
    }
    
    if (-not $success) {
        throw "Operation failed after $MaxRetries attempts. Last error: $($lastError.Exception.Message)"
    }
}

# Usage
Invoke-SafeOperation -Operation {
    Invoke-WebRequest -Uri "https://api.example.com/data"
} -MaxRetries 5 -RetryDelaySeconds 3
```

### Custom Exception Classes
```powershell
class CustomException : System.Exception {
    [string]$ErrorCode
    [DateTime]$Timestamp
    
    CustomException([string]$message, [string]$errorCode) : base($message) {
        $this.ErrorCode = $errorCode
        $this.Timestamp = Get-Date
    }
}

class ValidationException : CustomException {
    [string]$FieldName
    
    ValidationException([string]$message, [string]$fieldName) : base($message, "VALIDATION_ERROR") {
        $this.FieldName = $fieldName
    }
}

# Usage
function Test-Input {
    param([string]$Value)
    
    try {
        if ([string]::IsNullOrWhiteSpace($Value)) {
            throw [ValidationException]::new("Value cannot be empty", "Value")
        }
        # Process value
    }
    catch [ValidationException] {
        Write-Error "Validation failed for field '$($_.FieldName)': $($_.Message)"
    }
}
```

### Debugging Techniques
```powershell
# Set breakpoint in script
Set-PSBreakpoint -Script "C:\Scripts\MyScript.ps1" -Line 25

# Conditional breakpoint
Set-PSBreakpoint -Script "C:\Scripts\MyScript.ps1" -Line 25 -Action {
    if ($variable -gt 100) {
        break
    }
}

# Variable breakpoint
Set-PSBreakpoint -Variable "importantVar" -Mode Write

# Command breakpoint
Set-PSBreakpoint -Command "Invoke-WebRequest"

# List breakpoints
Get-PSBreakpoint

# Remove breakpoints
Remove-PSBreakpoint -Id 1
Get-PSBreakpoint | Remove-PSBreakpoint

# Enable/Disable breakpoints
Disable-PSBreakpoint -Id 1
Enable-PSBreakpoint -Id 1

# Step debugging
$debugPreference = "Continue"  # Enable debug messages
$VerbosePreference = "Continue"  # Enable verbose messages

# Trace script execution
Set-PSDebug -Trace 1  # Trace lines before execution
Set-PSDebug -Trace 2  # Trace lines and variable assignments
Set-PSDebug -Off      # Turn off tracing

# Strict mode
Set-StrictMode -Version Latest  # Catch common errors
```

### Logging Framework
```powershell
class Logger {
    [string]$LogPath
    [string]$LogLevel
    
    Logger([string]$logPath, [string]$logLevel) {
        $this.LogPath = $logPath
        $this.LogLevel = $logLevel
    }
    
    hidden [void] WriteLog([string]$level, [string]$message) {
        $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
        $logEntry = "[$timestamp] [$level] $message"
        
        # Console output
        switch ($level) {
            "ERROR" { Write-Host $logEntry -ForegroundColor Red }
            "WARN"  { Write-Host $logEntry -ForegroundColor Yellow }
            "INFO"  { Write-Host $logEntry -ForegroundColor Green }
            default { Write-Host $logEntry }
        }
        
        # File output
        Add-Content -Path $this.LogPath -Value $logEntry
    }
    
    [void] Error([string]$message) {
        $this.WriteLog("ERROR", $message)
    }
    
    [void] Warning([string]$message) {
        if ($this.LogLevel -in @("WARN", "INFO", "DEBUG")) {
            $this.WriteLog("WARN", $message)
        }
    }
    
    [void] Info([string]$message) {
        if ($this.LogLevel -in @("INFO", "DEBUG")) {
            $this.WriteLog("INFO", $message)
        }
    }
    
    [void] Debug([string]$message) {
        if ($this.LogLevel -eq "DEBUG") {
            $this.WriteLog("DEBUG", $message)
        }
    }
}

# Usage
$logger = [Logger]::new("C:\Logs\application.log", "INFO")
$logger.Info("Application started")
$logger.Warning("Low disk space")
$logger.Error("Failed to connect to database")
```

## Performance Optimization

### Measuring Performance
```powershell
# Measure-Command
$duration = Measure-Command {
    Get-Process | Where-Object CPU -gt 10
}
Write-Output "Execution time: $($duration.TotalMilliseconds) ms"

# Compare methods
$method1 = Measure-Command {
    $array = @()
    for ($i = 0; $i -lt 10000; $i++) {
        $array += $i
    }
}

$method2 = Measure-Command {
    $list = [System.Collections.Generic.List[int]]::new()
    for ($i = 0; $i -lt 10000; $i++) {
        $list.Add($i)
    }
}

Write-Output "Method 1 (Array +=): $($method1.TotalMilliseconds) ms"
Write-Output "Method 2 (List.Add): $($method2.TotalMilliseconds) ms"
```

### Optimization Techniques
```powershell
# Use .NET types for better performance
# Slow - PowerShell array
$array = @()
for ($i = 0; $i -lt 10000; $i++) {
    $array += $i
}

# Fast - .NET List
$list = [System.Collections.Generic.List[int]]::new()
for ($i = 0; $i -lt 10000; $i++) {
    $list.Add($i)
}

# Use ArrayList
$arrayList = [System.Collections.ArrayList]::new()
for ($i = 0; $i -lt 10000; $i++) {
    [void]$arrayList.Add($i)  # [void] prevents output
}

# Avoid pipeline when not needed
# Slow
$result = Get-ChildItem -Path C:\Temp | Where-Object Length -gt 1MB

# Faster
$result = foreach ($item in Get-ChildItem -Path C:\Temp) {
    if ($item.Length -gt 1MB) {
        $item
    }
}

# Pre-compile regex
$pattern = [regex]::new("\d{3}-\d{2}-\d{4}", [System.Text.RegularExpressions.RegexOptions]::Compiled)
$strings = 1..1000 | ForEach-Object { "SSN: 123-45-$_" }
foreach ($str in $strings) {
    $pattern.IsMatch($str)
}

# Use StringBuilder for string concatenation
$sb = [System.Text.StringBuilder]::new()
for ($i = 0; $i -lt 1000; $i++) {
    [void]$sb.Append("Line $i`n")
}
$result = $sb.ToString()
```

### Memory Management
```powershell
# Clear variables when done
$largeData = Get-LargeDataSet
# Process data
Remove-Variable -Name largeData

# Force garbage collection (use sparingly)
[System.GC]::Collect()
[System.GC]::WaitForPendingFinalizers()

# Use streaming for large files
# Bad - loads entire file into memory
$content = Get-Content -Path "C:\LargeFile.txt"

# Good - process line by line
Get-Content -Path "C:\LargeFile.txt" | ForEach-Object {
    # Process each line
}

# Even better - use .NET StreamReader
$reader = [System.IO.StreamReader]::new("C:\LargeFile.txt")
try {
    while ($null -ne ($line = $reader.ReadLine())) {
        # Process line
    }
}
finally {
    $reader.Close()
}
```

### Parallel Processing
```powershell
# ForEach-Object -Parallel (PowerShell 7+)
$servers = 1..100 | ForEach-Object { "Server$_" }

$results = $servers | ForEach-Object -Parallel {
    $serverName = $_
    Test-Connection -ComputerName $serverName -Count 1 -Quiet
} -ThrottleLimit 10

# Parallel with shared state (PowerShell 7+)
$sharedData = [hashtable]::Synchronized(@{})

1..10 | ForEach-Object -Parallel {
    $num = $_
    $hash = $using:sharedData
    $hash[$num] = $num * 2
} -ThrottleLimit 5

# Runspaces for advanced parallel processing
$runspacePool = [runspacefactory]::CreateRunspacePool(1, 5)
$runspacePool.Open()

$jobs = 1..10 | ForEach-Object {
    $ps = [powershell]::Create().AddScript({
        param($number)
        Start-Sleep -Seconds 1
        return $number * 2
    }).AddArgument($_)
    
    $ps.RunspacePool = $runspacePool
    
    [PSCustomObject]@{
        PowerShell = $ps
        Handle     = $ps.BeginInvoke()
    }
}

# Collect results
$results = $jobs | ForEach-Object {
    $_.PowerShell.EndInvoke($_.Handle)
    $_.PowerShell.Dispose()
}

$runspacePool.Close()
$runspacePool.Dispose()
```

## Desired State Configuration (DSC) Basics

### Simple DSC Configuration
```powershell
# Define configuration
Configuration WebServerConfig {
    param(
        [string[]]$ComputerName = "localhost"
    )
    
    Import-DscResource -ModuleName PSDesiredStateConfiguration
    
    Node $ComputerName {
        # Ensure Windows feature is installed
        WindowsFeature IIS {
            Ensure = "Present"
            Name   = "Web-Server"
        }
        
        # Ensure directory exists
        File WebContent {
            Ensure          = "Present"
            Type            = "Directory"
            DestinationPath = "C:\inetpub\wwwroot\MyApp"
            DependsOn       = "[WindowsFeature]IIS"
        }
        
        # Ensure file exists with specific content
        File IndexHtml {
            Ensure          = "Present"
            Type            = "File"
            DestinationPath = "C:\inetpub\wwwroot\MyApp\index.html"
            Contents        = "<html><body><h1>Hello from DSC</h1></body></html>"
            DependsOn       = "[File]WebContent"
        }
        
        # Ensure service is running
        Service IISService {
            Name        = "W3SVC"
            State       = "Running"
            StartupType = "Automatic"
            DependsOn   = "[WindowsFeature]IIS"
        }
    }
}

# Compile configuration to MOF
WebServerConfig -ComputerName "Server01" -OutputPath "C:\DSC\WebServerConfig"

# Apply configuration
Start-DscConfiguration -Path "C:\DSC\WebServerConfig" -Wait -Verbose

# Test configuration
Test-DscConfiguration -Path "C:\DSC\WebServerConfig"

# Get current configuration
Get-DscConfiguration
```

### Custom DSC Resources
```powershell
# Create custom DSC resource class
[DscResource()]
class MyCustomResource {
    [DscProperty(Key)]
    [string]$Name
    
    [DscProperty(Mandatory)]
    [string]$Value
    
    [DscProperty()]
    [Ensure]$Ensure = [Ensure]::Present
    
    # Get method - returns current state
    [MyCustomResource] Get() {
        $current = [MyCustomResource]::new()
        $current.Name = $this.Name
        
        # Check if resource exists and get its value
        if (Test-Path -Path "C:\Config\$($this.Name).txt") {
            $current.Value = Get-Content -Path "C:\Config\$($this.Name).txt" -Raw
            $current.Ensure = [Ensure]::Present
        }
        else {
            $current.Ensure = [Ensure]::Absent
        }
        
        return $current
    }
    
    # Test method - returns true if in desired state
    [bool] Test() {
        $current = $this.Get()
        
        if ($this.Ensure -eq [Ensure]::Present) {
            return ($current.Ensure -eq [Ensure]::Present -and $current.Value -eq $this.Value)
        }
        else {
            return ($current.Ensure -eq [Ensure]::Absent)
        }
    }
    
    # Set method - brings resource to desired state
    [void] Set() {
        if ($this.Ensure -eq [Ensure]::Present) {
            $this.Value | Out-File -FilePath "C:\Config\$($this.Name).txt" -Force
        }
        else {
            Remove-Item -Path "C:\Config\$($this.Name).txt" -Force -ErrorAction SilentlyContinue
        }
    }
}
```

## Working with .NET Classes

### Common .NET Classes
```powershell
# System.IO.File
[System.IO.File]::Exists("C:\Temp\file.txt")
[System.IO.File]::ReadAllText("C:\Temp\file.txt")
[System.IO.File]::WriteAllText("C:\Temp\file.txt", "Content")
[System.IO.File]::Copy("C:\Source.txt", "C:\Dest.txt")
[System.IO.File]::Delete("C:\Temp\file.txt")

# System.IO.Path
[System.IO.Path]::GetFileName("C:\Temp\file.txt")           # "file.txt"
[System.IO.Path]::GetDirectoryName("C:\Temp\file.txt")      # "C:\Temp"
[System.IO.Path]::GetExtension("C:\Temp\file.txt")          # ".txt"
[System.IO.Path]::GetFileNameWithoutExtension("C:\Temp\file.txt")  # "file"
[System.IO.Path]::Combine("C:\Temp", "SubDir", "file.txt")  # "C:\Temp\SubDir\file.txt"

# System.Math
[Math]::Round(3.14159, 2)              # 3.14
[Math]::Ceiling(3.2)                   # 4
[Math]::Floor(3.8)                     # 3
[Math]::Abs(-10)                       # 10
[Math]::Max(10, 20)                    # 20
[Math]::Min(10, 20)                    # 10
[Math]::Pow(2, 8)                      # 256
[Math]::Sqrt(16)                       # 4

# System.DateTime
$now = [DateTime]::Now
$utcNow = [DateTime]::UtcNow
$today = [DateTime]::Today
$parsed = [DateTime]::Parse("2024-01-15")
$tryParse = [DateTime]::TryParse("2024-01-15", [ref]$parsed)

# System.TimeSpan
$span = New-TimeSpan -Days 1 -Hours 2 -Minutes 30
$span = [TimeSpan]::FromMinutes(90)
$span.TotalHours                       # 1.5

# System.Guid
$guid = [Guid]::NewGuid()
$parsed = [Guid]::Parse("a1234567-89ab-cdef-0123-456789abcdef")

# System.Convert
[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("Hello"))
[Convert]::FromBase64String("SGVsbG8=")
[Convert]::ToInt32("FF", 16)           # 255 (hex to decimal)

# System.Text.Encoding
$bytes = [System.Text.Encoding]::UTF8.GetBytes("Hello")
$text = [System.Text.Encoding]::UTF8.GetString($bytes)

# System.Net.WebClient
$client = [System.Net.WebClient]::new()
$client.DownloadFile("https://example.com/file.zip", "C:\Temp\file.zip")
$content = $client.DownloadString("https://api.example.com/data")
$client.Dispose()
```

### Creating .NET Objects
```powershell
# New-Object
$list = New-Object System.Collections.Generic.List[string]
$dict = New-Object 'System.Collections.Generic.Dictionary[string,int]'

# Type accelerator
$list = [System.Collections.Generic.List[string]]::new()
$dict = [System.Collections.Generic.Dictionary[string,int]]::new()

# Working with generic collections
$list = [System.Collections.Generic.List[int]]::new()
$list.Add(1)
$list.Add(2)
$list.Add(3)
$list.Remove(2)
$list.Contains(1)                      # True
$list.Count                            # 2

# Dictionary
$dict = [System.Collections.Generic.Dictionary[string,int]]::new()
$dict.Add("One", 1)
$dict.Add("Two", 2)
$dict["Three"] = 3
$dict.ContainsKey("One")               # True
$dict.Keys
$dict.Values

# Stack
$stack = [System.Collections.Generic.Stack[string]]::new()
$stack.Push("First")
$stack.Push("Second")
$top = $stack.Pop()                    # "Second"

# Queue
$queue = [System.Collections.Generic.Queue[string]]::new()
$queue.Enqueue("First")
$queue.Enqueue("Second")
$first = $queue.Dequeue()              # "First"
```

## Security Best Practices

### Credential Management
```powershell
# Never store credentials in plain text
# Bad
$username = "admin"
$password = "P@ssw0rd"

# Better - prompt for credentials
$credential = Get-Credential

# Use SecureString
$securePassword = ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential("username", $securePassword)

# Export encrypted credentials (user-specific encryption)
$credential | Export-Clixml -Path "C:\Secure\cred.xml"
$credential = Import-Clixml -Path "C:\Secure\cred.xml"

# Decrypt SecureString (when necessary)
$bstr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($securePassword)
$plainPassword = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($bstr)
[System.Runtime.InteropServices.Marshal]::ZeroFreeBSTR($bstr)

# Use Windows Credential Manager (Windows only)
Install-Module -Name CredentialManager
New-StoredCredential -Target "MyApp" -UserName "admin" -Password "P@ssw0rd" -Type Generic
$cred = Get-StoredCredential -Target "MyApp"
```

### Execution Policy
```powershell
# View current execution policy
Get-ExecutionPolicy
Get-ExecutionPolicy -List

# Set execution policy
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Bypass execution policy for single script
PowerShell.exe -ExecutionPolicy Bypass -File .\script.ps1

# Policies:
# - Restricted: No scripts allowed
# - AllSigned: Only signed scripts
# - RemoteSigned: Local scripts OK, downloaded must be signed
# - Unrestricted: All scripts allowed (prompts for downloaded)
# - Bypass: Nothing blocked, no warnings
```

### Code Signing
```powershell
# Get code signing certificate
$cert = Get-ChildItem -Path Cert:\CurrentUser\My -CodeSigningCert

# Sign script
Set-AuthenticodeSignature -FilePath ".\MyScript.ps1" -Certificate $cert

# Verify signature
Get-AuthenticodeSignature -FilePath ".\MyScript.ps1"

# Create self-signed certificate (for testing only)
$cert = New-SelfSignedCertificate -Subject "CN=PowerShell Code Signing" `
    -Type CodeSigningCert `
    -CertStoreLocation Cert:\CurrentUser\My
```

### Input Validation
```powershell
function Invoke-SafeCommand {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [ValidatePattern("^[A-Za-z0-9_-]+$")]
        [string]$Name,
        
        [Parameter(Mandatory = $true)]
        [ValidateScript({
            Test-Path -Path $_ -PathType Leaf
        })]
        [string]$Path
    )
    
    # Sanitize input
    $safeName = $Name -replace '[^A-Za-z0-9_-]', ''
    
    # Validate path is in allowed directory
    $allowedPath = "C:\Safe\Directory"
    $resolvedPath = Resolve-Path -Path $Path
    
    if (-not $resolvedPath.Path.StartsWith($allowedPath)) {
        throw "Path must be within $allowedPath"
    }
    
    # Process safely
}

# Avoid command injection
# Bad
$command = "Get-Process -Name $userInput"
Invoke-Expression $command

# Good
$command = {
    param($Name)
    Get-Process -Name $Name
}
& $command -Name $userInput
```

### Logging and Auditing
```powershell
# Enable script block logging
# Registry: HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" `
    -Name "EnableScriptBlockLogging" -Value 1 -PropertyType DWord

# Enable module logging
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging" `
    -Name "EnableModuleLogging" -Value 1 -PropertyType DWord

# Enable transcription
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription" `
    -Name "EnableTranscripting" -Value 1 -PropertyType DWord

# Manual transcription
Start-Transcript -Path "C:\Logs\session.txt"
# Commands executed here are logged
Stop-Transcript

# View security events
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 100
```

## Best Practices (Do's)

✅ **Use classes for complex data structures**
```powershell
# Good - organized and reusable
class Configuration {
    [string]$Name
    [hashtable]$Settings
    
    [void] Validate() {
        # Validation logic
    }
}
```

✅ **Implement proper error handling with retry logic**
```powershell
# Good - resilient to transient failures
function Invoke-WithRetry {
    param($ScriptBlock, $MaxAttempts = 3)
    
    for ($i = 1; $i -le $MaxAttempts; $i++) {
        try {
            return & $ScriptBlock
        }
        catch {
            if ($i -eq $MaxAttempts) { throw }
            Start-Sleep -Seconds (2 * $i)
        }
    }
}
```

✅ **Use .NET types for performance-critical operations**
```powershell
# Good - much faster for large collections
$list = [System.Collections.Generic.List[int]]::new()
for ($i = 0; $i -lt 100000; $i++) {
    $list.Add($i)
}
```

✅ **Create comprehensive module manifests**
```powershell
# Good - provides metadata and dependencies
New-ModuleManifest -Path ".\MyModule.psd1" `
    -RootModule "MyModule.psm1" `
    -ModuleVersion "1.0.0" `
    -Description "Comprehensive module" `
    -FunctionsToExport @("Get-Data", "Set-Data")
```

✅ **Use SecureString for sensitive data**
```powershell
# Good - protects credentials
$cred = Get-Credential
$cred | Export-Clixml -Path ".\cred.xml"
```

✅ **Implement proper logging**
```powershell
# Good - traceable operations
Start-Transcript -Path "C:\Logs\script-$(Get-Date -Format 'yyyyMMdd-HHmmss').txt"
try {
    # Script logic
}
finally {
    Stop-Transcript
}
```

✅ **Use parallel processing for independent operations**
```powershell
# Good - faster for I/O bound operations
$servers | ForEach-Object -Parallel {
    Test-Connection -ComputerName $_ -Count 1
} -ThrottleLimit 10
```

## Common Mistakes (Don'ts)

❌ **Don't use array += for large collections**
```powershell
# Bad - very slow
$array = @()
for ($i = 0; $i -lt 100000; $i++) {
    $array += $i
}

# Good - use List
$list = [System.Collections.Generic.List[int]]::new()
for ($i = 0; $i -lt 100000; $i++) {
    $list.Add($i)
}
```

❌ **Don't store credentials in plain text**
```powershell
# Bad - security risk
$password = "P@ssw0rd"

# Good - use SecureString
$securePassword = Read-Host -AsSecureString "Enter password"
```

❌ **Don't use Invoke-Expression with user input**
```powershell
# Bad - code injection vulnerability
Invoke-Expression "Get-Process -Name $userInput"

# Good - use proper parameter passing
Get-Process -Name $userInput
```

❌ **Don't ignore module dependencies**
```powershell
# Bad - may fail on other systems
function Get-AzureData {
    Get-AzVM  # What if Az module isn't installed?
}

# Good - check for dependencies
function Get-AzureData {
    if (-not (Get-Module -ListAvailable -Name Az)) {
        throw "Az module is required"
    }
    Import-Module Az
    Get-AzVM
}
```

❌ **Don't skip input validation**
```powershell
# Bad - vulnerable to injection
function Remove-UserFile {
    param($FileName)
    Remove-Item -Path "C:\Users\$FileName"
}

# Good - validate input
function Remove-UserFile {
    param(
        [ValidatePattern("^[A-Za-z0-9_-]+\.(txt|log)$")]
        [string]$FileName
    )
    Remove-Item -Path "C:\Users\$FileName"
}
```

❌ **Don't forget to dispose of resources**
```powershell
# Bad - resource leak
$stream = [System.IO.FileStream]::new("C:\file.txt", "Open")
# Process stream

# Good - proper cleanup
$stream = [System.IO.FileStream]::new("C:\file.txt", "Open")
try {
    # Process stream
}
finally {
    $stream.Dispose()
}
```

❌ **Don't use Write-Host for pipeline output**
```powershell
# Bad - breaks pipeline
function Get-Result {
    Write-Host "Result"
}

# Good - proper output
function Get-Result {
    Write-Output "Result"
}
```

❌ **Don't hardcode paths without checking**
```powershell
# Bad - assumes path exists
Copy-Item -Path "C:\Source\file.txt" -Destination "C:\Dest\"

# Good - validate first
if (Test-Path "C:\Source\file.txt") {
    if (-not (Test-Path "C:\Dest")) {
        New-Item -Path "C:\Dest" -ItemType Directory
    }
    Copy-Item -Path "C:\Source\file.txt" -Destination "C:\Dest\"
}
```
