**Objective:**  
Package the PowerShell Command Center (PSCC) as a local MCP (Multi-Command Platform) connector, so that tools like Claude and VSCode can install the connector and invoke all PSCC capabilities via MCP commands, including custom script execution, thread management, and output handling.

---

### 1. **Connector Overview & Goals**
- **Purpose:** Enable seamless, programmatic access to all PSCC features (multi-threaded PowerShell execution, thread account management, custom script execution, output retrieval) through MCP commands.
- **Target Platforms:** Local installation for Windows 10/Server 2012 R2+ environments, with support for integration in VSCode and LLM-based tools (e.g., Claude).
- **User Experience:** Users should be able to install the MCP connector, configure PSCC, and invoke any PSCC-supported operation (including custom scripts) via standardized MCP commands.

---

### 2. **Connector Packaging Requirements**
- **Installer:** Provide a script or package (e.g., pip, npm, or MSI) that:
  - Installs PSCC and all required dependencies (PowerShell 5+, required modules, .NET, etc.).
  - Registers the MCP connector with the local MCP registry.
  - Optionally, exposes a CLI and/or REST API for local and remote invocation.
- **Configuration:** Allow users to specify:
  - PSCC root directory and script locations.
  - Thread account config files or automated provisioning options.
  - Target environment (Azure, Exchange Online, SharePoint Online, etc.).
  - Authentication credentials (with secure storage).
- **Permissions:** Ensure the connector can launch PowerShell sessions with the necessary privileges and manage thread accounts as described in the documentation.

---

### 3. **Connector Command Mapping**
- **Expose all PSCC “Execution Methods” as MCP commands.**  
  For each method, map parameters (input files, thread count, account prefix, etc.) to MCP command arguments.
- **Support for Custom Scripts:**  
  - Allow users to specify a custom .ps1 script and required input files.
  - Pass all necessary variables and context from MCP to the script, as PSCC does.
- **Thread Management:**  
  - Provide MCP commands for automated and manual thread account provisioning, including config file management.
  - Expose thread status, progress, and logs via MCP queries.
- **Output Handling:**  
  - Retrieve and consolidate per-thread output files.
  - Provide MCP commands to fetch, download, or stream consolidated results.

---

### 4. **Technical Implementation Details**
- **PowerShell Integration:**  
  - Use PowerShell SDK or subprocess invocation to launch PSCC scripts and modules.
  - Handle execution policy (e.g., set to Unrestricted if needed).
  - Ensure .ps1 files are unblocked and accessible.
- **Authentication:**  
  - Support both interactive and config-file-based authentication.
  - Allow for secure storage of credentials and thread account passwords.
- **Concurrency & Throttling:**  
  - Respect service-specific concurrency limits (e.g., Exchange Online max 9 threads by default).
  - Allow users to specify thread count per execution.
- **Error Handling:**  
  - Capture and relay all PSCC error messages and logs to MCP.
  - Provide clear feedback for failed executions or misconfigurations.

---

### 5. **Documentation & Examples**
- **Provide sample MCP commands for:**
  - Installing and configuring the connector.
  - Running a built-in PSCC execution method (e.g., Exchange Online mailbox stats).
  - Provisioning thread accounts (automated and manual).
  - Running a custom script with input files.
  - Fetching and consolidating output.
- **Include troubleshooting tips for common issues (e.g., authentication failures, throttling limits, missing modules).**

---

### 6. **Security & Compliance**
- **Credential Management:**  
  - Ensure all credentials are stored securely and never logged in plaintext.
  - Advise users to use complex passwords for thread accounts and exempt them from MFA if required (as per PSCC limitations).
- **Audit Logging:**  
  - Log all MCP command invocations and PSCC executions for traceability.

---

### 7. **Extensibility**
- **Allow for future expansion:**  
  - Design the connector so new PSCC modules or execution methods can be mapped to MCP commands without major refactoring.
  - Support for additional cloud services as PSCC evolves.

---

### 8. **References**
- All technical details, prerequisites, and workflows should be implemented as described at the bottom of this readme. 
---

**Validation Audit**
- Completed tool needs full end to end validation and produciton polish. 


---
PowerShell Command Center 

PowerShell Command Center (PSCC) is a versatile tool that allows for the rapid, multi-threaded execution of PowerShell commands against Azure Cloud-based services, with support for extension to other services. While most PowerShell scripts take days to develop and countless hours to run at scale, PSCC provides the user with an intuitive graphical user interface that makes them productive in minutes, and it leverages a multi-threading engine that drops execution time by an order of magnitude. It comes prebuilt to support the execution of common tasks, but it provides a template for extension that can be used to accomplish anything with the same powerful capabilities. For experienced users, PSCC provides a level of agility and a scale of execution that can best be described as a kind of scripting super power, allowing them to overcome any number of obstacles for customers. 

How It Works 

PSCC works by allowing a user to execute pre-defined scripts made available in the UI called “Execution Methods”. When the user selects an Execution Method and qualifies the tool with all the necessary parameters and selects the “Start” button, PSCC creates separate PowerShell sessions that connect to the target service called “Threads”. The number of threads that are available are dependent on the number of accounts created and permissioned accordingly in Azure AD for use. Some services have throttling limits that allow for a few as 10 concurrent PowerShell sessions to be active at once, while other allow for 80 or more concurrent connections. Most throttling policies are enforced on a per-user basis, so, by providing a single, unique account per thread, its possible to perform activities against the cloud services in bulk with parallel threads using unique accounts. Typically, for an activity that takes a single query 5 hours to run, it will take PSCC with 5 threads only 1 hour to run (or 30 mins for 10 threads); there is generally a directly correlation between the number of active threads and the time it takes to execute. 

For example, if you are querying Exchange Online, the default throttling policy allows for a maximum concurrency of 9 sessions, so you’ll want to create 9 separate accounts (in the circumstance of this activity). If you need to asses the overall mailbox statistics for the entire organization, you would execute the Exchange Online -> Reporting -> Mailbox Statistics execution method, select the “thread account prefix” of the accounts, or use an account config file and select “9” threads. Once you select “Start”, PSCC will call the module designed to execute the Mailbox Statistics execution method and 9 PowerShell windows will pop up. Each window (aka thread) indicates the progress of the activity and which account is being used to execute against it. Each window will have a different account executing the query, which minimizes disruption from throttling. The PowerShell window will remain open until every thread is complete. Once they are all complete, a consolidated report is generated with the output from all threads. 

Account and Software Prerequisites and Recommendations 

PowerShell Command Center General Requirements 

Hardware 

Spec 

Hosted 

VM Name 

CPUs 

RAM 

IOPs 

Max Threads 

Minimum 

Azure 

D2s_v3 

2 

8GB 

3200 

15 

Recommended 

Azure 

D4s_v3 

4 

16GB 

6400 

30 

Preferred 

Azure 

DS4_v2 

4 

14GB 

12800 

40+ 

Operating System 

Windows 10 OR 

Windows Server 2012 R2 or later 

Software 

Minimum: PowerShell Version 3 

Recommended: PowerShell Version 5. To download the latest version of the Windows Management Framework, go to aka.ms/wmf 

Recommendations 

If you or the customer submit a service request to MSOnline Support, you can request for the Exchange Remote PowerShell Throttling Policy to be relaxed for up to 80 in support of migrations. In doing so, the following values will be modified: 

Default Organization Throttling Policy 

Updated Organization Throttling Policy 

Parameters 

Values 

Parameters 

Values 

ExchangeMaxCmdlets 

                200  

ExchangeMaxCmdlets 

                     400  

PowerShellCutoffBalance 

    3,000,000  

PowerShellCutoffBalance 

         6,000,000  

PowerShellMaxBurst 

        900,000  

PowerShellMaxBurst 

         1,800,000  

PowerShellMaxCmdlets 

                400  

PowerShellMaxCmdlets 

                     800  

PowerShellMaxConcurrency 

                  20  

PowerShellMaxConcurrency 

                       80  

PowerShellMaxDestructiveCmdlets 

                120  

PowerShellMaxDestructiveCmdlets 

                     240  

PowerShellMaxOperations 

                400  

PowerShellMaxOperations 

                     800  

PowerShellMaxTenantConcurrency 

                100  

PowerShellMaxTenantConcurrency 

                     400  

PowerShellRechargeRate 

    2,160,000  

PowerShellRechargeRate 

         5,040,000  

 

Note: This affects all PowerShell related request to the target tenant. The recommended number of accounts/concurrent threads are outlined per-service is based on the default throttling policies for those services. 

Limitations 

Currently PSCC does not support Modern Authentication and Multi-Factor Auth. For customers that have MFA enabled in their environment, these thread accounts must be exempted from policy. It is strongly advised that the passwords used for the thread accounts are very complex. 

Depending on the activity you are executing and the enforced throttling policies, some activities may fail if you execute them with too many concurrent threads. It’s important to understand the tenant’s limitations prior to execution, otherwise you may risk retrieving incomplete data sets. 

 

SharePoint Online (SPO) 

Accounts 

20 to 40 accounts assigned: 

SharePoint Online Administrator MSOLRole  

Directory Readers or User Account Administrator 

Software 

The SharePoint Online Client Components SDK (CSOM - aka.ms/SPOCSOM) and SharePoint Online Management Shell (aka.ms/SPOMS) must be installed on the workstation 

CSOM queries can be execute with as many as 40 concurrent threads, the primary limiting factor is the workstation. For ideal throughput, 20 concurrent threads are recommended. 

SharePoint Online queries can be executed with as many as 10 concurrent threads. For ideal throughput, 10 concurrent threads are recommended 

Recommendations 

Since the SPO Management Shell and CSOM queries both require SharePoint Online Administrator rights, request the customer provision 20 accounts and use those 20 accounts for all of the SPO related tasks. 

Exchange Online (ExO) 

Accounts 

Create 10 accounts that have: 

Exchange Service Administrator RBAC 

Directory Readers or User Account Administrator 

Software 

The workstation must meet the General Requirements of PowerShell Command Center. Exchange Online uses Remote PowerShell and does not required any additional components. 

Microsoft Online Service (MSOL) 

Accounts 

Create 10 accounts that have: 

User Account Administrator 

Software 

The MSOnline Module must be installed on the workstation. This requires PowerShell version 3.0 or greater in order to run Install-Module and install the MSOnline Module from the PowerShell Gallery. 

Getting Started 

Executing PowerShell Command Center 

To being with, download PowerShell Command Center to the local drive of your workstation, open PowerShell and navigate to the file location.  

A blue and white screen with white text

Description automatically generated 

Note: You many need to set the execution policy on the local workstation to Unrestricted (Set-ExecutionPolicy -ExecutionPolicy Unrestricted). If it fails to launch, you may also need to “Unblock” the .ps1 files by right clicking the file, getting properties and selecting “Unblock”. 

Requirement Assessment 

At launch of PowerShell Command Center (PSCC), an initial check on the state of your PowerShell configuration will take place. This process validates the supported version of PowerShell and detects the modules that are currently installed on your local workstacd..tion. If a specific module is marked as ‘Not Detected’, you may or may not need to act; you can ignore the alert if you do not plan to the associated features/services. 

A screenshot of a computer

Description automatically generated 

First Login 

PSCC needs to first connect to a target environment before it will allow you to do anything. This requires that you first have an Office 365 tenant and secondly, that you have access to the tenant. The activities you are trying to perform will determine the level of access you need 

Select the ‘Office 365 Admin Account’ button. 

Enter the credentials of the account. 

A screenshot of a computer

Description automatically generated 

Note: You can limit the access of the account as much as you would like, so long it has the ability to use the MSOL cmdlets. Functions like “Manage Thread Accounts” would be unavailable in this case, however. 

Once you are successfully authenticated, you’ll notice the tenant name is auto-populated based off the tenant you are connected to and the status window is updated to indicate which account you are logged in with. 

A screenshot of a computer

Description automatically generated 

Once you have successfully authenticated, you must ensure that you have the thread accounts setup correctly if you haven’t do so yet. 

Thread Account Setup 

PSCC supports the creation of thread accounts in two ways: 

Automated, tool-based provisioning 

Manual provisioning 

Automated Provisioning 

Thread Manager 

Automated, tool-based provisioning can be accomplished through the Thread Account Manager, once a user has successfully authenticated. In order to leverage this account provisioning method, the account must be assigned “Company Administrator” rights in Azure Active Directory (Azure AD). Once authenticated,  

Select “Manage Thread Accounts. 

A screenshot of a computer

Description automatically generated 

If this is your first-time creating Thread Accounts, the pop-up dialog will be empty. 

Select “Create” 

A screenshot of a computer

Description automatically generated 

Next, the “Create Thread” dialog box will appear and prompt you to enter information in to the form. 

Complete the form. 

Thread Account Prefix: This will become the display name of the account in Azure AD. The account will automatically have “MCC-“ appended to the beginning and incremented with “01” through X number of thread accounts you choose to create.  

In this case MCC-PSCCGlobalAdmin01@MSMADLAB1.onmicrosoft.com through MCC-PSCCGlobalAdmin40@MSMADLAB1.onmicrosoft.com will be created. 

Password: The password that all thread accounts will be created with 

Alt E-mail Address: The Alternate Email Address assigned to the account. This is to help any customer administrator identify the owner of the account 

Phone Number: This is set as the phone number of the account, in the case of an emergency 

Usage Location: The Usage Location of the account 

Thread Count: The number of Thread Accounts you wish to have created 

Select “Create”. 

Once you select “Create”, there will be a minor delay as the accounts are created. 

Note: When creating accounts through the thread manager, they will be created as unlicensed Company Administrators. 

A screenshot of a computer

Description automatically generated 

Once the Thread Accounts are created, the accounts will be visible and manageable though the Thread Account Manager. If you need to Modify or delete these accounts, you can do so through the user interface. 

Select “Okay” 

A screenshot of a computer

Description automatically generated 

Standalone Script 

Not all customers are comfortable with allowing automated tools to create accounts in their environment(s). If required, the following script will provision a set number of cloud-based accounts with the requisite account naming structure. In the sample script below, 10 accounts are created with the UPN of MCC-MSFTAdmin01@contoso.onmicrosoft.com through MCC-MSFTAdmin10@contoso.onmicrosoft.com. Each account is assigned the password “Sup3rP@55w0rd!” and SharePoint Service Administrator rights. This script should be modified to meet your needs. 

$numToCreate = 10 

$Name = "PSCCGlobalAdmin" 

$TenantLong = "msmadlab1.onmicrosoft.com" 

$AccountOwner = "paulkot@microsoft.com" 

$UsageLocation  = "US" 

$NewPassword = "Sup3rP@55w0rd!" 

$AdminType = "SharePoint Service Administrator" #run get-msolrole to determine name if other than SPO Admin 

$AdminRole    = (Get-MsolRole | Where-Object {$_.Name -eq $AdminType}).ObjectID  

 

for($i = 1;$i -le $numToCreate;$i ++){ 

    $UserUPNPrefix  = "MCC-$name{0:D2}" -f $i 

    $UPN  = $UserUPNPrefix + "@" + $TenantLong 

    $FirstName      = $UserUPNPrefix 

    $LastName       = "Office365" 

    $DisplayName    = $LastName + ', ' + $FirstName 

     

    Write-Host "---------Creating Account $i------------" 

    Write-host "UPN        : " $UPN 

    Write-host "DisplayName: " $DisplayName 

    Write-host "Title      : " $Name    

    write-host "" 

    New-MsolUser -UserPrincipalName $UPN -DisplayName $DisplayName -LastName $LastName -FirstName $FirstName -AlternateEmailAddresses $AccountOwner -Title "MCC-$Name" -UsageLocation $NewUsageLocation -ForceChangePassword $false -PasswordNeverExpires $true -Password $NewPassword | Out-Null 

    Add-MsolRoleMember -RoleObjectId $AdminRole -RoleMemberEmailAddress $UPN 

    } 

  

Once the accounts are created, they will be manageable though the Thread Account Manager (if the logged in accounts has adequate permissions). 

Manual Provisioning 

For customers that require very specific naming structures, have already provisioned X number of accounts for use or have security structures in place that require each of the Thread Accounts to have unique passwords, it may be difficult to work with the customer to re-create the Thread Accounts to the liking of the Thread Account Manager. In this case, the accounts can be created however necessary, but a special config file must be created. 

In the root of the PSCC folder (where the PSCC365.ps1 exists), create a file name <TenaneName>.Accounts.Config (i.e. MSMadLab1.Accounts.Config). 

Open the config file with you editor of choice. 

Enter the header values of “AccountUPN” and “AccountPassword”. 

Save. 

For example, the MSMadLab1.Accounts.Config file should look like this in Notepad: 

AccountUPN,AccountPassword 

UniqueAcct01@MSMadLab1.onmicrosoft.com,Un1qu3PWD01 

UniqueAcct02@MSMadLab1.onmicrosoft.com,Un1qu3PWD02 

UniqueAcct03@MSMadLab1.onmicrosoft.com,Un1qu3PWD03 

UniqueAcct04@MSMadLab1.onmicrosoft.com,Un1qu3PWD04 

UniqueAcct05@MSMadLab1.onmicrosoft.com,Un1qu3PWD05 

UniqueAcct06@MSMadLab1.onmicrosoft.com,Un1qu3PWD06 

UniqueAcct07@MSMadLab1.onmicrosoft.com,Un1qu3PWD07 

UniqueAcct08@MSMadLab1.onmicrosoft.com,Un1qu3PWD08 

UniqueAcct09@MSMadLab1.onmicrosoft.com,Un1qu3PWD09 

UniqueAcct10@MSMadLab1.onmicrosoft.com,Un1qu3PWD10 

If this was done correctly, the next time you select the “Thread Account Prefix” dropdown in PSCC, you should see and option that says “Use Config File <LoggedInTenantName>.Accounts.Config.  

Note: If you do not see your config file in the Thread Account Prefix dropdown, check the file name and ensure it starts with the correct tenant name. If it does not match, it will not show up. 

A screenshot of a computer

Description automatically generated 

Execution 

Specific execution approaches can be found in later sections that outline what exactly each activity in PSCC does, but the general execution of PSCC is consistent across all workloads. Depending on the activity you are trying to accomplish, you may or may not be required to select an input file.  

Using Input Files 

For example, if you are looking to capture the size and details of every mailbox in Exchange Online, you would need to seed the activity with a source file derived from the Get-Mailbox cmdlet first. When you have your seed fille: 

Connect to the target Tenant 

Select “Import File” 

Once the file is imported, a verification “ItemCount” dialog appears to confirm the number of mailboxes you have in the source file 

 

A screenshot of a computer

Description automatically generated 

General Execution 

For scripts that do and do not need seed/input files, the preliminar steps to execute activites are the same: 

Connect to the targeted tenant 

Select “Thread Account Prefix” dropdown box 

If you are configuring PSCC to run with “MCC-“ Thread Accounts created by one of the automated provisioning processes, you’ll need to enter the “Thread Account Password” as well. If you are using a config file, the file itself should contain the associated account passwords. 

 

A screenshot of a computer

Description automatically generated 

Once you have selected the desired Thread Account Prefix, PSCC should analyze the account prefix/file to determine the number of accounts available to use for execution 

Select “Number of Threads” and choose the desired number. 

Select “Start” 

Note: Depending on the activity you are executing and the enforced throttling policies, some activities may fail if you execute them with too many concurrent threads. It is important to understand the tenant’s limitations prior to execution, otherwise you may risk retrieving incomplete data sets. 

 

A screenshot of a computer

Description automatically generated 

 

Understanding Active Threads 

Upon selecting “Start”, PSCC will generate X number of PowerShell sessions, depending on the “Number of Threads” you selected. In the example above, we selected “10” threads, so 10 PS sessions were created for Exchange Online. 

A stack of blue and white papers

Description automatically generated 

Thread Status Information Windows 

Where an activity is started, there are quite a few things that go on behind the scenes, however the important things to know are output in the header of each of the threads. The information that is output is unique to the script, but if any issues come up, it important to know what you’re looking. 

Execution Method: The script in the /<PSCC Root>/Modules folder that is being executed 

Migration Account: The account that is being passed to that specific session 

In the title bar of the Status Window includes additional information that is specific to that session/thread. In short, the title bar is labeled as: 

<Thread Number> | <Tenant Name> | <Executed Script> | Line <Current/Total> | <Percent Complete> 

A group of blue and yellow text boxes

Description automatically generated 

At-a-glance Thread Progress Viewer 

This window is a summary of the overall progress of the activity you are executing. It includes: 

The tenant you are connected to 

The script that is executing 

The progress on each thread along with an estimated amount of time remaining on each thread 

Note: The progress on each thread is an estimated amount of time that is based off how long it has taken the script to complete the current number of queries over the total number of queries it needs to do. Many activities are linear in terms of execution time; however some activities may take more time to execute on one user v. another and may be less accurate when querying all Personal Site Items for a specific user v. retrieving the Suffixes of All MSOLusers. 

A screenshot of a computer

Description automatically generated 

Understanding the Output 

For each PowerShell Session/Thread that is initiated, the log of the output on a per-session basis is generated in the /<PSCC Root>/<TenantName>/Threads/ location with the naming structure of <ExecutionMethod>.Results.<ThreadNumber>.<DateTime>.csv 

A screenshot of a computer screen

Description automatically generated 

When all of the active threads have completed their activity and are in a “Completed” state, a file consolidation process will take place. All files related to a specific activity will be consolidated and output to the /<PSCC Root>/<TenantName>/ location with the naming structure of <TenantName>.<ExecutionMethod>.Consolidated.<DateTime>.csv.  

A screen shot of a computer

Description automatically generated 

Advanced Settings and Custom Scripts 

For advanced users, PSCC allows then  to augment the current capabilities of PSCC with their own custom scripts using a template script that enables the user to quickly get started and execute activities at scale. To trigger the execution of a custom script: 

Select the “Settings” tab 

Check the “Override Method” checkbox 

Select your customer script 

For custom scripts that require input files, follow the steps outlined in “Using Input Files” and “General Execution”. The actual execution of a customer script would follow the same workflow as any built-in module. 

Note: The Override Method search for .ps1 scripts in the root of the PSCC folder. 

A screenshot of a computer

Description automatically generated 

Creating Custom Scripts 

Creating custom scripts for user with PSCC is rather straightforward if you have prior experience creating script in general and it shouldn’t require any in-depth knowledge of how PSCC actually work. 

A sample file that can be modified for creating custom scripts can be found <location>/Custom-Template-GetRecipientExample.ps1 

The content of the template should not be modified unless otherwise stated. Numerous functions and variables are pre-defined and passed from the PSCC UI to the custom template which allows it to function like any of the built-in modules. Within the template, the only section that needs modification is located within the region: 

 

#======================================================================================= 

#region CUSTOM SCRIPT SECTION - YOU ARE FREE TO MODIFY  

#======================================================================================= 

 

To further elaborate, when the script is run, 

Define which services you need to connect to, in this case Exchange Online and MS Online are unremarked since we are using cmdlets from those modules. 

$UsersToCapture is the variable that contains the full input of the CSV, so the next section can be read as “Foreach line item in the CSV that was imported, do this action” 

$Error.Clear() clears any error already stored in the PowerShell session 

$ReportProgress = ReportStatus is what generates the status information displayed in the Windows UI Header 

The content from “Do Work Below Here” to “Do Work Above Here” should contain all your custom code. Since, in the example I’m pulling all recipients in Exchange Online to retrieve their UPNs in Azure AD. I’m calling the Get-Recipient cmdlet for each identity in the source file, and capturing the actual ObjectID. From there, I use that ObjectID to resolve the MSOLUser and store the output as $UserData which has been initialized as a New PSObject.  

After $UserData has the approperiate columns created and is populated with data, it is output to a CSV which is predefined in the variable $OutputFilePath 

After it has been output, the $ReportIncriment is incremented by 1 and the next user is retrieved from $UsersToCapture 

 

#======================================================================================= 

#region CUSTOM SCRIPT SECTION - YOU ARE FREE TO MODIFY  

#======================================================================================= 

 

#▼▼▼▼▼Remove Commenting for Services to Connect to ▼▼▼▼▼#  

 

MCC-ConnectMSOL 			#Connect to MSOnline 

MCC-ConnectEXO			#Connect to Exchange Online  

#MCC-ConnectCSOM                   		#Connect to Client Side Object Model (CSOM for SPO) 

#MCC-ConnectSPO                     		#Connect to SharePoint Online 

#▲▲▲▲▲Remove Commenting for Services to Connect to ▲▲▲▲▲# 

 

#---------------Initialization of Foreach and Other Required Functions------------------# 

 

$UsersToCapture | foreach {        		#For each users in the imported file 

$Error.Clear()                 		#Clear any current errors 

$ReportProgress = ReportStatus	#Report current status to window bar and thread progress manager 

  

#▼▼▼▼▼▼▼▼▼Do Work Below Here ▼▼▼▼▼▼▼▼▼▼#  

#-------------------------------Input and CMDLETs---------------------------------# 

 

$Identity             	= $_.Identity 

$Recipient            	= "" 

$MSOLUser             	= "" 

$Recipient            	= Get-Recipient -Identity $Identity | select Identity, CustomAttribute8, CustomAttribute9, ExternalEmailAddress, PrimarySmtpAddress, WindowsLiveID, ExchangeGuid, Id, Guid, ExternalDirectoryObjectId, BlockCredential 

if($Error) {$MSOLUser  	= Get-MsolUser -UserPrincipalName $Identity} 

Else{$MSOLUser 	= Get-MsolUser -ObjectId $Recipient.ExternalDirectoryObjectId} 

 

#------------------------------Generate Output Here-------------------------------# 

 

$UserData = New-Object   PSObject 

$UserData | Add-Member NoteProperty Identity             		$Identity 

$UserData | Add-Member NoteProperty UserPrincipalName    	$MSOLUser.UserPrincipalName 

$UserData | Add-Member NoteProperty IsLicensed           	$MSOLUser.IsLicensed 

$UserData | Add-Member NoteProperty BlockCredential     	$MSOLUser.BlockCredential 

$UserData | Add-Member NoteProperty CustomAttribute8    	$Recipient.CustomAttribute8 

$UserData | Add-Member NoteProperty CustomAttribute9     	$Recipient.CustomAttribute9 

$UserData | Add-Member NoteProperty ExternalEmailAddress	$Recipient.ExternalEmailAddress 

$UserData | Add-Member NoteProperty PrimarySmtpAddress   	$Recipient.PrimarySmtpAddress 

$UserData | Add-Member NoteProperty WindowsLiveID       	$Recipient.WindowsLiveID 

$UserData | Add-Member NoteProperty ExchangeGuid         	$Recipient.ExchangeGuid 

$UserData | Add-Member NoteProperty Id                   		$Recipient.Id 

$UserData | Add-Member NoteProperty Guid                 		$Recipient.Guid 

$UserData | Add-Member NoteProperty Error               		$Error[0].Exception.Message 

$UserData | Export-Csv                                   			$OutputFilePath -NoTypeInformation -Append 

 

#▲▲▲▲▲▲▲▲▲Do Work Above Here ▲▲▲▲▲▲▲▲▲▲#  

$ReportIncriment++             #Increment report status 

} 

     

#endregion 

#======================================================================================= 

  

Services and Modules Reference 

 
