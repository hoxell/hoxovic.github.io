---
title: "Setting up vim in PowerShell"
layout: single
published: true
---

One of the features I really miss in PowerShell is a command line editor. The open source community and Microsoft have done an awesome job with VS Code, but while working in the terminal, opening VS Code would disrupt the workflow. Enter _WSL_ and _vim_.

In PowerShell, you can run

```shell
$ bash
```

to start an interactive bash session. Obviously, this requires you to have set up WSL and installed a suitable Linux distro. Now you can simply run vim from bash as you would in Linux. However, that requires an extra step as compared to working on Linux or macOS.

Instead, let's create a PowerShell function that can be invoked like so:

```shell
$ vim <path_to_file>
```

## Finding the location of the PowerShell profile
In the [official documentation](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles?view=powershell-7) you can read about PowerShell profiles. Basically, you can use the profiles to customize your environment.

The PowerShell console supports the following profiles (but other hosts can support other profiles):

| Description                | Path                                                             |
| :------------------------- | :--------------------------------------------------------------- |
| All Users, All hosts       | $PSHOME\Profile.ps1                                              |
| All Users, Current Host    | $PSHOME\Microsoft.PowerShell_profile.ps1                         |
| Current User, All Hosts    | $Home\[My ]Documents\PowerShell\Profile.ps1                      |
| Current user, Current Host | $Home\[My ]Documents\PowerShell\Microsoft.PowerShell_profile.ps1 |

<br/>
You can use `$PROFILE.<profile_name>` to get the path to the corresponding profile (e.g. `$PROFILE.AllUsersAllHosts`). 

## Add the function to your profile
Decide which profile you want to use and open it for editing. I went with _Current User, All Hosts_.

```shell
$ code $PROFILE.CurrentUserAllHosts
```

Add the following snippet:

```powershell
function vim ($FileName)
{
    $FileName = $FileName.Replace("\", "/").Replace(" ", "\ ")
    bash -c "vim $FileName"
}
```

The first line is needed because of different conventions of how to represent a path in Windows and Linux. The second line simply invokes bash in "command mode", meaning that bash will execute the command string and exit. When you quit vim, you'll be taken right back to where you were in PowerShell.

## Source the profile
To make the function available in the active session, you'll need to source the profile. Adapt the following to the profile you chose in the previous step:

```shell
$ .$PROFILE.CurrentUserAllHosts
```
