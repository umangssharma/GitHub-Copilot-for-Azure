# Prerequisites

Complete these checks in order. Do not start a Prometheus investigation until every required check passes.

## Requirements

| Requirement | Needed for | Verification |
|---|---|---|
| Azure CLI | Authentication and extension host | `az --version` |
| ORAS | Downloading the extension wheel from ACR | `oras version` |
| `aiinfraoptimization` extension | Workload-intelligence commands | `az extension show --name aiinfraoptimization` |
| Azure sign-in | Azure Monitor managed Prometheus access | `az account show` |
| `Monitoring Data Reader` | Querying the Azure Monitor workspace | Confirm role assignment when access fails |

ORAS is not needed after the extension is installed, but still check it when the extension is missing or must be replaced.

## 1. Detect the environment

Determine the operating system and available package manager without changing the machine:

```powershell
$IsWindows
$IsLinux
$IsMacOS
Get-Command winget, brew, snap, az, oras -ErrorAction SilentlyContinue
```

On POSIX shells:

```bash
uname -s
command -v brew || true
command -v snap || true
command -v az || true
command -v oras || true
```

Do not assume that Windows implies WinGet, or that Linux implies a particular distribution or package manager.

## 2. Check Azure CLI and ORAS

Run each command independently so one missing executable does not hide the other:

```powershell
az --version
oras version
```

If both are available, continue to extension verification.

If either is missing, present one installation plan listing only the missing components, the package manager, and the commands that will run. Obtain explicit approval before executing it.

### Windows with WinGet

```powershell
winget install --exact --id Microsoft.AzureCLI
winget install --exact --id ORASProject.ORAS
```

Run only the command for each missing component. Do not reinstall a component whose verification already succeeded.

### macOS with Homebrew

```bash
brew install azure-cli
brew install oras
```

Run only the command for each missing component.

### Linux and WSL

- If ORAS is missing and `snap` is available, offer:

  ```bash
  sudo snap install oras --classic
  ```

- Azure CLI installation is distribution-specific. Do not guess a command or pipe a downloaded script into a privileged shell. Direct the user to [Install the Azure CLI on Linux](https://learn.microsoft.com/cli/azure/install-azure-cli-linux), ask them to select their distribution, and stop until `az --version` succeeds.
- If `snap` is unavailable, direct the user to the official [ORAS installation guide](https://oras.land/docs/installation) and stop until `oras version` succeeds.

### Unsupported or missing package manager

Provide the matching official instructions and stop:

- [Install Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Install ORAS](https://oras.land/docs/installation)

Do not improvise download URLs, versions, archive extraction, `sudo` operations, or PATH changes.

### Verify after installation

Run:

```powershell
az --version
oras version
```

On Windows, Azure CLI and ORAS installation may require a new terminal for PATH changes to take effect. If either command remains unavailable, tell the user to open a new terminal or reload VS Code and stop. Do not report installation success until both commands execute.

## 3. Verify Azure authentication

The signed-in identity needs `Monitoring Data Reader` on the Azure Monitor managed service for Prometheus workspace. Confirm role propagation when access was granted recently.

Run:

```powershell
az account show --output table
```

If no account is active, run `az login`. If the current subscription is not the intended one, ask for confirmation before running:

```powershell
az account set --subscription "<subscription-id-or-name>"
```

Never expose access tokens or place them in command arguments, logs, or chat.

## 4. Check the extension

Run:

```powershell
az extension show --name aiinfraoptimization --output table
```

If the extension is installed, record its version and verify its command:

```powershell
az aiinfraoptimization workload-intelligence --help
```

If a different version is installed, report the installed and documented versions. Ask whether to keep the working version or replace it; never replace it automatically.

## 5. Install the extension when missing or approved for replacement

Installation changes the local Azure CLI environment. Obtain the user's approval before installing or replacing the extension.

The repository quickstart currently publishes version `0.1.0` through ACR. Confirm that `oras version` succeeds, then run:

```powershell
oras version

$extensionVersion = "0.1.0"
$downloadDirectory = Join-Path $HOME "aiinfraoptimization-$extensionVersion"
New-Item -ItemType Directory -Path $downloadDirectory -Force | Out-Null

oras pull --output $downloadDirectory "infraiqlab.azurecr.io/cli/aiinfraoptimization-whl:$extensionVersion"

$wheelPath = Join-Path $downloadDirectory "aiinfraoptimization-$extensionVersion-py3-none-any.whl"
az extension add --yes --allow-preview true --source $wheelPath
```

On a POSIX shell:

```bash
extension_version="0.1.0"
download_directory="$HOME/aiinfraoptimization-$extension_version"
mkdir -p "$download_directory"

oras pull --output "$download_directory" \
  "infraiqlab.azurecr.io/cli/aiinfraoptimization-whl:$extension_version"

az extension add --yes --allow-preview true \
  --source "$download_directory/aiinfraoptimization-$extension_version-py3-none-any.whl"
```

Verify installation:

```powershell
az extension show --name aiinfraoptimization --output table
az aiinfraoptimization workload-intelligence --help
```

Do not proceed if the pull fails, the expected wheel is absent, `az extension add` fails, or the workload-intelligence help command is unavailable. Report the failing prerequisite and preserve the command error.
