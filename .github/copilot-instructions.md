# GitHub Copilot Instructions for PowerShell Command Center MCP

## Project Overview

This repository contains the **PowerShell Command Center (PSCC) MCP Connector**, which packages the PowerShell Command Center as a local Multi-Command Platform (MCP) connector. This enables tools like Claude, VSCode, and other LLM-based applications to programmatically access all PSCC capabilities through standardized MCP commands.

### Key Features
- Multi-threaded PowerShell execution for Microsoft 365 services
- Thread account management (automated and manual provisioning)
- Custom script execution with input file support
- Output consolidation and retrieval
- Integration with Exchange Online, SharePoint Online, Azure AD, and more

## Architecture & Technology Stack

- **Primary Language**: PowerShell (5.1+)
- **Integration**: MCP (Multi-Command Platform) protocol
- **Target Platforms**: Windows 10/Server 2012 R2+
- **Cloud Services**: Microsoft 365, Azure, Exchange Online, SharePoint Online
- **Authentication**: OAuth, basic auth, config file-based credentials

## Development Guidelines

### Code Style & Conventions

#### PowerShell
- Use approved PowerShell verbs (Get-, Set-, New-, Remove-, etc.)
- Follow PowerShell naming conventions with PascalCase for functions
- Use proper error handling with try/catch blocks
- Include comment-based help for all functions
- Prefix custom functions with `MCC-` (Multi-Command Center)

#### Security Best Practices
- **Never store credentials in plaintext**
- Use secure credential storage mechanisms
- Implement proper input validation and sanitization
- Log authentication attempts but never log passwords
- Support MFA exemption for thread accounts when required
- Use complex passwords for thread accounts

#### MCP Integration
- Map all PSCC execution methods to MCP commands
- Provide clear parameter documentation for each MCP command
- Handle asynchronous operations properly
- Implement proper error handling and status reporting
- Support both local and remote invocation patterns

### File Organization

```
/
├── .github/
│   ├── copilot-instructions.md (this file)
│   └── workflows/
├── src/
│   ├── connectors/          # MCP connector implementations
│   ├── pscc/               # Core PSCC PowerShell modules
│   ├── scripts/            # Sample and utility scripts
│   └── config/             # Configuration templates
├── tests/
│   ├── unit/               # Unit tests
│   ├── integration/        # Integration tests
│   └── fixtures/           # Test data and fixtures
├── docs/
│   ├── installation.md    # Installation guide
│   ├── configuration.md   # Configuration guide
│   └── examples/          # Usage examples
└── README.md
```

### Testing Approach

#### PowerShell Testing
- Use Pester framework for PowerShell module testing
- Mock external service calls (Exchange Online, Azure AD, etc.)
- Test both success and failure scenarios
- Include performance tests for multi-threaded operations

#### MCP Testing
- Test MCP command mapping and parameter validation
- Verify proper error handling and status reporting
- Test authentication flows
- Validate output formatting and consolidation

### Common Patterns

#### Thread Management
```powershell
# Example thread account provisioning pattern
function New-ThreadAccount {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory = $true)]
        [string]$TenantName,
        
        [Parameter(Mandatory = $true)]
        [string]$AccountPrefix,
        
        [Parameter()]
        [int]$ThreadCount = 10
    )
    
    # Implementation follows PSCC patterns
}
```

#### Error Handling
```powershell
try {
    # PSCC operation
    $result = Invoke-PSCCOperation @params
    Write-Output $result
}
catch {
    Write-Error "Operation failed: $($_.Exception.Message)"
    # Log error details for debugging
    Write-Debug $_.Exception.StackTrace
}
```

#### Configuration Management
```powershell
# Use secure configuration patterns
$configPath = Join-Path $env:USERPROFILE ".pscc\config.json"
$config = Get-Content $configPath | ConvertFrom-Json
```

### Documentation Standards

#### Function Documentation
Always include comment-based help:
```powershell
<#
.SYNOPSIS
    Brief description of the function.

.DESCRIPTION
    Detailed description of what the function does.

.PARAMETER ParameterName
    Description of the parameter.

.EXAMPLE
    Example of how to use the function.

.NOTES
    Additional information about the function.
#>
```

#### README Updates
When adding new features:
- Update the main README.md with new capabilities
- Include usage examples
- Document any new configuration requirements
- Add troubleshooting tips for common issues

### Integration Considerations

#### Microsoft 365 Services
- Handle throttling limits appropriately
- Implement retry logic for transient failures
- Support both modern and legacy authentication
- Consider regional data residency requirements

#### MCP Protocol
- Follow MCP specification for command structure
- Implement proper capability discovery
- Support both synchronous and asynchronous operations
- Provide clear status and progress reporting

### Troubleshooting Common Issues

#### Authentication Failures
- Verify PowerShell execution policy is set correctly
- Check that .ps1 files are unblocked
- Validate credentials and permissions
- Test connectivity to target services

#### Module Dependencies
- Ensure all required PowerShell modules are installed
- Handle version compatibility issues
- Provide clear error messages for missing dependencies

#### Threading Issues
- Monitor thread pool usage
- Implement proper resource cleanup
- Handle concurrent access to shared resources
- Validate thread account availability

## Development Workflow

1. **Setup**: Install PowerShell 5.1+, required modules, and dependencies
2. **Configuration**: Set execution policy and unblock files as needed
3. **Testing**: Run unit tests before committing changes
4. **Documentation**: Update relevant documentation for new features
5. **Security Review**: Ensure no credentials are exposed in code or logs

## Useful Commands

```powershell
# Set execution policy for development
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser

# Unblock PowerShell files
Get-ChildItem -Path . -Recurse -Filter "*.ps1" | Unblock-File

# Install required modules (example)
Install-Module -Name ExchangeOnlineManagement -Force
Install-Module -Name Microsoft.Graph -Force

# Run Pester tests
Invoke-Pester -Path ./tests/
```

## Resources

- PowerShell Command Center documentation in README.md
- Microsoft Graph PowerShell SDK documentation
- Exchange Online PowerShell reference
- MCP specification and examples

---

This file should be updated as the project evolves to ensure Copilot has the most current context for generating helpful code suggestions.