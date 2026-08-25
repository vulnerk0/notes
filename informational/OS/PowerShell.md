# CMDLets
these are the commands

>[!note] 
> The Windows terminal is case insensitive.


##### Help Menu (man | -h)
```powershell
Get-Help Get-ChildItem
```

##### Update Help Menu
```powershell
Update-Help
```

##### Get Current Location (pwd)
```powershell
Get-Location
```

##### List Directory Contents (ls)
```powershell
Get-ChildItem
```
or to list a different directory from the current one
```powershell
Get-ChildItem 'C:\'
```

##### Move to Directory (cd)
```powershell
Set-Location C:\Users\Administrator\secretsauce
```

##### Output file contents (cat)
```powershell
Get-Content secretsauce.txt
```

### Get-Command

This cmdlet will help us find the commands we are looking for.

get all commands currently loaded into powershell
```powershell
Get-Command
```

get all commands by specifying a verb (first word)
```powershell
Get-Command -verb get # Get all commands that has the verb "get" like Get-ChildItem
Get-Command -verb get* # Get all commands which verb starts with "get" like Get-ChildItem
Get-Command get* # Get all commands that start with "get" like GetHelp.exe
```

get all commands by specifying a noun (second word)
```powershell
Get-Command -noun Windows # You get the idea
Get-Command -noun Windows* 
Get-Command Windows*
```

##### Get Commands From a Module
```powershell
Get-Command -Module PowerSploit
```

##### Get Commands History (history)
```powershell
Get-History

Id CommandLine 
-- ----------- 
1 Get-Command 
2 clear 
3 get-command -verb set
```

to rerun a command we use `r` with the command number. Run the `clear` command
```powershell
r 2
```

once the powershell session is closed, the commands will disappear from future powershell sessions. To get the last 4096 commands run on the system, we check the \_history.txt file.
```powershell
Get-Content C:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

##### Clear Screen (clear)
```powershell
Clear-Host
clear
cls
```

# Hotkeys

|**HotKey**|**Description**|
|---|---|
|`CTRL+R`|It makes for a searchable history. We can start typing after, and it will show us results that match previous commands.|
|`CTRL+L`|Quick screen clear.|
|`CTRL+ALT+Shift+?`|This will print the entire list of keyboard shortcuts PowerShell will recognize.|
|`Escape`|When typing into the CLI, if you wish to clear the entire line, instead of holding backspace, you can just hit `escape`, which will erase the line.|
|`↑`|Scroll up through our previous history.|
|`↓`|Scroll down through our previous history.|
|`F7`|Brings up a TUI with a scrollable interactive history from our session.|

# Tab Completion

We can type a word like "get" and press tab for auto-completion, pressing `TAB` moves forward, and pressing `SHIFT+TAB` moves backward.

# Aliases

Aliases are shortened cmdlets (commands), we can view all aliases using `Get-Alias` or `gal`
```powershell
Get-Alias

Alias % -> ForEach-Object 
Alias ? -> Where-Object
Alias ac -> Add-Content 
Alias asnp -> Add-PSSnapin 
Alias cat -> Get-Content
```

We can also set an alias for a cmdlet like `Get-Help`
```powershell
Set-Alias -Name gh -Value Get-Help
```


# Working with Modules

A PowerShell module is a self-contained, reusable package that groups related cmdlets, functions, variables, aliases, and providers to manage specific tasks or system areas.

get all the loaded powershell modules
```powershell
Get-Module
```

list available modules to be imported
```powershell
Get-Module -ListAvailable
```

import available modules
```powershell
Import-Module MODULE
```

view the default module path
```powershell
$env:PSModulePath
```


# Execution Policy

The execution policy in powershell is NOT a security measure, it is a safeguard for IT departments.

Check execution policy
```powershell
Get-ExecutionPolicy
```
- Restricted: you can't import modules
- Undefined: you CAN import modules

Set execution policy
```powershell
Set-ExecutionPolicy undefined # or restricted
```

##### Stealthily change the execution policy
We can change the execution policy at the `Process` level, which should make us more stealthy.
```powershell
Set-ExecutionPolicy -scope Process # once prompted for the value, type "bypass"
```

check the EP
```powershell
Get-ExecutionPolicy -list 
```

this [blog post](https://www.netspi.com/blog/technical/network-penetration-testing/15-ways-to-bypass-the-powershell-execution-policy/) covers more ways to bypass the EP.

# Users and Groups

##### Get a list of local groups
```powershell
Get-LocalGroup
```

##### Get a list of local accounts
```powershell
Get-LocalUser
```

##### Create a User
```powershell
New-LocalUser -Name "abdulfattah_elmaksiki" -NoPassword
```