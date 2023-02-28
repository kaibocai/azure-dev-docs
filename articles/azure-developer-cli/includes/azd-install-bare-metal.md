---
author: alexwolfmsft
ms.service: azure-dev-cli
ms.topic: include
ms.date: 01/11/2023
ms.author: alexwolf
---

## Pre-requisites

Before you get started using `azd`, ensure you have:

- Installed:
  - [Git](https://git-scm.com/)
  - [GitHub CLI v2.3+](https://github.com/cli/cli) **(only required for `azd pipeline config` when using GitHub Actions)**
- An Azure account with an active subscription. [Create one for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).


## Install `azd`

### [Windows](#tab/windows)

Run the following script:

```azdeveloper
powershell -ex AllSigned -c "Invoke-RestMethod 'https://aka.ms/install-azd.ps1' | Invoke-Expression"
```

### [Linux/MacOS](#tab/linuxmac)

```azdeveloper
curl -fsSL https://aka.ms/install-azd.sh | bash 
```

---

## Install using MSI directly

For an advanced approach, you can install using the MSI directly. Download the MSI directly from [GitHub Releases](https://github.com/Azure/azure-dev/releases) and install the MSI via the following command:

```cmd
msiexec.exe /i <msi-path> <optional parameters>
```

When installing using the MSI directly (instead of the [install script](#install-azd)) the MSI behavior can be modified by providing the following parameters to `msiexec.exe`.

| Parameters | Value |
| -------- | ----- |
| `ALLUSERS` | `2`: Default. Install for current user (no privilege elevation required). <br/> `1`: Install for _all_ users (may require privilege elevation). |
| `INSTALLDIR` | Installation path. <br/> `"%LOCALAPPDATA%\Programs\Azure Dev CLI"`: Default. <br/> `"%PROGRAMFILES%\Azure Dev CLI"`: Default all users. |

For example, to install for all users in `c:\all-users\azd`, you can run a command similar to:

```cmd
msiexec.exe /i <msi-path> ALLUSERS=1 INSTALLDIR=c:\all-users\azd
```

> [!NOTE]
> The install script doesn't support installing versions of `azd` on Windows that **predate** the MSI. To manually update older versions of the Azure Developer CLI without MSI, see the [Install versions predating MSI section](#install-versions-predating-msi). 

## Install using package management tools

A package manager assists developers and administrators with installing, updating, configuring, and removing software packages in a reliable way. You can install the Azure Developer CLI using the following popular package management tools:

* Brew
* Windows Package Manager

### [Brew](#tab/brew)

```bash
brew tap azure/azd && brew install azd
```

### [Winget](#tab/winget)

```bash
winget install microsoft.azd
```

---

## Uninstall `azd`

To uninstall the `azd`:

### [Windows](#tab/windows)

Once you've installed the MSI versions, using the uninstall script to remove `azd` will leave some items behind on the machine. **Instead, for version 0.5.0-beta.1 and later:** 

1. Search for **Add or remove programs** in Windows.

1. Locate **Azure Dev CLI** and select the three dots to expand the options menu.

1. Select **Uninstall**.

**For versions before 0.5.0-beta.1**, use the following uninstall script:

```azdeveloper
powershell -ex AllSigned -c "Invoke-RestMethod 'https://aka.ms/uninstall-azd.ps1' | Invoke-Expression"
```

### [Linux/MacOS](#tab/linuxmac)

```azdeveloper
curl -fsSL https://aka.ms/uninstall-azd.sh | bash 
```

---

## Uninstall using a package manager

If you installed the Azure Developer CLI using a package manager, you can also uninstall it using the following commands:

### [Brew](#tab/brew)

```bash
brew uninstall azd
```

### [Winget](#tab/winget)

```bash
winget uninstall microsoft.azd
```

---

## Install versions predating MSI

As of version 0.5.0-beta.1, the PowerShell install script for Azure Developer CLI (`install-azd.ps1`) uses the published MSI file instead of installing from the .zip file. There is no change for users who want to use the script to install or upgrade. 

Since the install script doesn't support installing versions of `azd` on Windows that **predate** the MSI, you'll need to manually install older versions. 

1. Download the appropriate .zip file from the [Azure Developer CLI GitHub releases](https://github.com/Azure/azure-dev/releases). 

1. Extract the .zip file.
1. Rename the `azd-windows-amd64.exe` to `azd.exe`
1. Place `azd.exe` in the appropriate location.