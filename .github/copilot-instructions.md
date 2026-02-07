# Copilot Instructions for MyBuildModules

## Project Overview
MyBuildModules is a PowerShell module library for Windows systems automation. The project focuses on building reusable, production-ready modules for common Windows system administration tasks.

## Architecture & Structure

### Module Organization
- **PSM1 Files**: Core module implementation (functions, logic)
- **PSD1 Files**: Module manifests defining exports, dependencies, PowerShell versions
- **Module Root**: Create a top-level folder per module under `/Modules`
- **Naming**: Use `PascalCase` for module names; PowerShell convention (e.g., `MyUtilityModule`)

### Typical Module Structure
```
/Modules/
  ModuleName/
    ModuleName.psd1       # Manifest with metadata
    ModuleName.psm1       # Main module file
    /Public               # Exported functions
      Function1.ps1
      Function2.ps1
    /Private              # Internal helpers (not exported)
      Helper1.ps1
    /Tests                # Pester tests
      ModuleName.Tests.ps1
    README.md             # Module-specific documentation
```

## Development Conventions

### Function Design
- **Naming**: Use `Verb-Noun` format following PowerShell cmdlet naming guidelines (e.g., `Get-SystemInfo`, `Set-Configuration`)
- **Parameters**: Use `[Parameter()]` attributes; mark mandatory params with `[Parameter(Mandatory=$true)]`
- **Output**: Return objects (not strings) for pipeline-able behavior; use `[PSCustomObject]` for structured data
- **Help**: Include PowerShell Help comments (`<#...#>`) with `.SYNOPSIS`, `.DESCRIPTION`, `.PARAMETER`, `.EXAMPLE` sections

### Error Handling
- Use `$ErrorActionPreference = 'Stop'` at module start for fail-fast behavior
- Throw terminating errors with descriptive messages: `throw "Specific error description"`
- Use Write-Error for non-terminating errors when appropriate

### Code Style
- **Indentation**: 4 spaces (PowerShell standard)
- **Variables**: Use `$PascalCase` for module/script scope; `$camelCase` for local
- **Splatting**: Use hashtable splatting for complex command calls: `@{ Param1 = 'value'; Param2 = 'value' } | % { cmd @_ }`
- **PowerShell Version**: Minimum PowerShell 5.1 (supports `$PSScriptRoot`, classes, and modern error handling)

## Target Domains & Module Scope

Modules in MyBuildModules cover critical Windows system administration areas:

### Active Directory (AD)
- Domain user and group management
- Computer object lifecycle (create, move, remove)
- Group membership and delegation
- Use `Get-ADUser`, `Get-ADGroup`, `Get-ADComputer` from ActiveDirectory module
- Require domain-joined systems; document elevation needs

### Group Policy Objects (GPO)
- GPO creation, modification, and linking
- GP registry preferences
- Replication and consistency checks
- Use GroupPolicy module commands (`New-GPO`, `Set-GPInheritance`, etc.)

### File Systems & Storage
- Share management, permissions, and auditing
- Volume and mount point operations
- ACL management with security contexts
- Use `Get-Acl`, `Set-Acl` for permission manipulation

### System Configuration
- Network settings, DNS, DHCP
- Services and scheduled tasks
- Windows registry operations
- Windows Features installation/removal

## Build & Test Workflow

### Testing
- Use **Pester** for unit tests (included with PowerShell 5.0+)
- Run tests before committing: `Invoke-Pester /Modules/ModuleName/Tests/`
- Test public functions thoroughly; test error conditions and edge cases
- Mock external dependencies (file system, registry, network) in tests

### Module Import
- Test module loads cleanly: `Import-Module -Name ./Modules/ModuleName/ -Verbose`
- Verify exported functions available after import
- Check for naming conflicts with existing commands: `Get-Command FunctionName`

### Build Commands (Planned)
- **Build**: Compile manifests, run linting
- **Test**: Execute Pester test suite
- **Publish**: PowerShell Gallery upload (future)

## Integration Points

### Internal Naming Conventions
- **Machine Names**: All internal resources follow pattern `w01*` (e.g., `w01-ad01`, `w01-fs02`)
- Module functions targeting machines should validate this naming convention when operating on local or remote systems
- Document machine role expectations (domain controller, file server, etc.) in module help

### Windows API/Registry Access
- Use `[System.Security.AccessControl.RegistryRights]` for registry operations
- Use WMI (`Get-WmiObject`) for system information; prefer `Get-CimInstance` (WMI v2) for newer systems
- Respect UAC; require elevation where needed with explicit documentation

### External Dependencies
- Minimize external module dependencies
- Document required PowerShell version (e.g., 5.1+, 7.0+)
- If using external modules, vendor them or document installation: `Install-Module DependencyName -Scope CurrentUser`

### Configuration
- Use JSON or PSD1 for configuration files
- Store configs in `$env:PROGRAMDATA` for system-wide; `$env:APPDATA` for user-specific
- Validate config schemas on load

## Common Patterns

### Logging
```powershell
function Write-ModuleLog {
    param([string]$Message, [string]$Level = 'Info')
    $timestamp = Get-Date -Format 'yyyy-MM-dd HH:mm:ss'
    Write-Host "[$timestamp] [$Level] $Message"
}
```

### Configuration Loading
```powershell
$configPath = Join-Path $PSScriptRoot 'config.json'
$config = @{}
if (Test-Path $configPath) {
    $config = Get-Content $configPath | ConvertFrom-Json
}
```

### Pipeline Support
Return objects that work in pipelines:
```powershell
$result = [PSCustomObject]@{
    ComputerName = $env:COMPUTERNAME
    Status = 'OK'
    Timestamp = Get-Date
}
Write-Output $result
```

## Key Files to Reference
- **LICENSE**: MIT License (check for any constraints before distribution)
- **README.md**: Project-level documentation (expand as modules are added)

## Future Enhancements
- Add Pester test framework setup and templates
- Create module template for new module development
- Add PSScriptAnalyzer rules for code quality
- Establish security scanning for sensitive data (credentials in logs, etc.)

## Questions for New Module Development
When creating a new module, clarify:
1. Which target domain(s)? (AD, GPO, File Systems, System Configuration)
2. Windows version requirements (Windows Server 2016+)?
3. Elevation requirements and access scope?
4. Local vs. remote execution capabilities?
