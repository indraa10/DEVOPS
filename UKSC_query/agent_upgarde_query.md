# Question : azure devops is doing build and deployment for dynamics 365 application , where agent is microsoft hosted windows server 2019 , how to ensure system will continue to work by upgrading tool versions

## chatgpt

To ensure your **Azure DevOps pipelines** that build and deploy a **[Dynamics 365](w)** application continue to function reliably on **Microsoft-hosted Windows Server 2019 agents**, you must proactively manage **tool version upgrades**, particularly for **Power Platform Build Tools**, **PowerShell modules**, and any **third-party dependencies**. Here’s a comprehensive strategy:

---

### ✅ 1. **Understand the Microsoft-Hosted Agent Image Lifecycle**

Microsoft-hosted agents (like `windows-latest` or `windows-2019`) are periodically **updated**. These updates may:

* Upgrade or deprecate pre-installed tools.
* Break existing scripts if versions change unexpectedly.

👉 **Reference:** [GitHub Agent Specification](w) (applies similarly to Azure DevOps agents):
[https://github.com/actions/runner-images](https://github.com/actions/runner-images)

---

### ✅ 2. **Pin Tool Versions in Your Pipeline**

Avoid relying on the "latest" version of tools. Instead:

* **Power Platform CLI (`pac`)**: Explicitly install and pin the version.
* **PowerShell Modules**: Use specific versions (e.g., `Install-Module Microsoft.PowerApps.Administration.PowerShell -RequiredVersion 2.0.120`).
* **Node.js, .NET SDK, etc.**: Use version selectors in the pipeline YAML (e.g., `useDotNet` task).

Example:

```yaml
- task: UseDotNet@2
  inputs:
    version: '6.0.x'
```

---

### ✅ 3. **Maintain a Tool Installation Script**

Create a `tools-install.ps1` script and include it in your pipeline to **install and lock** tools and modules your build depends on.

Example:

```powershell
Install-Module -Name Microsoft.PowerApps.Administration.PowerShell -RequiredVersion 2.0.120 -Scope CurrentUser -Force
Install-Module -Name Microsoft.PowerApps.PowerShell -RequiredVersion 2.0.120 -Scope CurrentUser -Force
```

Add to pipeline:

```yaml
- powershell: ./scripts/tools-install.ps1
  displayName: 'Install Required Tools'
```

---

### ✅ 4. **Use Self-Hosted Agent for Long-Term Stability (Optional)**

If you want **full control over versions** and stability:

* Use a **self-hosted agent** with pinned tool versions.
* Maintain it using scripts or configuration management tools (like DSC, Ansible, or Azure Automation).

---

### ✅ 5. **Automate Tool Version Testing**

* Maintain a **sandbox pipeline** that runs weekly to validate builds against the latest Microsoft-hosted image.
* Use this pipeline to detect upcoming breaking changes early.

---

### ✅ 6. **Stay Informed on Image Updates**

Subscribe or monitor:

* [Azure DevOps Agent Image Releases](w): [https://github.com/actions/runner-images/releases](https://github.com/actions/runner-images/releases)
* [Power Platform CLI release notes](w): [https://learn.microsoft.com/en-us/power-platform/developer/cli/introduction](https://learn.microsoft.com/en-us/power-platform/developer/cli/introduction)

---

### ✅ 7. **Monitor and Audit Tool Usage**

Add a diagnostic step in your pipeline to print tool versions:

```yaml
- powershell: |
    pac --version
    Get-InstalledModule
    $PSVersionTable
  displayName: 'Diagnostic - Tool Versions'
```

---

### ✅ Summary Checklist

| Action                           | Status   |
| -------------------------------- | -------- |
| Pin all tool versions            | ✅        |
| Use installation script          | ✅        |
| Create test pipelines            | ✅        |
| Monitor hosted agent changes     | ✅        |
| Optionally use self-hosted agent | Optional |

---

Would you like a YAML pipeline template for managing version-pinned tools for Dynamics 365 CI/CD?

Here is a **YAML pipeline template** for **Dynamics 365 build and deployment using Azure DevOps**, with tool versions explicitly pinned and best practices applied for Microsoft-hosted agents:

---

### 📄 **azure-pipelines.yml**

```yaml
trigger:
  branches:
    include:
      - main
      - release
      - develop

pool:
  vmImage: 'windows-2019'

variables:
  # Set tool versions
  PowerPlatformToolVersion: '2.0.120'
  DotNetVersion: '6.0.x'
  PacCliVersion: '1.32.4'

stages:
- stage: ToolSetup
  displayName: 'Install Required Tools'
  jobs:
  - job: InstallTools
    steps:

    - task: UseDotNet@2
      inputs:
        version: $(DotNetVersion)
      displayName: 'Install .NET SDK $(DotNetVersion)'

    - powershell: |
        Install-Module -Name Microsoft.PowerApps.Administration.PowerShell -RequiredVersion $(PowerPlatformToolVersion) -Scope CurrentUser -Force -AllowClobber
        Install-Module -Name Microsoft.PowerApps.PowerShell -RequiredVersion $(PowerPlatformToolVersion) -Scope CurrentUser -Force -AllowClobber
      displayName: 'Install Power Platform PowerShell Modules'

    - powershell: |
        Invoke-WebRequest -Uri "https://aka.ms/paccli" -OutFile ".\pac.msi"
        Start-Process msiexec.exe -ArgumentList '/i pac.msi /quiet /norestart' -Wait
        pac install latest --version $(PacCliVersion)
        pac --version
      displayName: 'Install Power Platform CLI (pac)'

    - powershell: |
        Write-Host "Installed Tools:"
        pac --version
        Get-InstalledModule
        $PSVersionTable
      displayName: 'Diagnostic - Verify Installed Tool Versions'

- stage: Build
  displayName: 'Build D365 Solution'
  dependsOn: ToolSetup
  jobs:
  - job: BuildSolution
    steps:
    - checkout: self

    - powershell: |
        # Replace with actual unpack command if source controlled unpacked
        pac solution list
      displayName: 'List Solutions (Sample PAC Command)'

    # Add your actual build steps here (pack solution, export, etc.)

- stage: Deploy
  displayName: 'Deploy to Environment'
  dependsOn: Build
  jobs:
  - deployment: DeployToDev
    environment: 'Dev'
    strategy:
      runOnce:
        deploy:
          steps:
          - powershell: |
              pac auth create --url https://your-org.crm.dynamics.com --name DevConnection
              pac solution import --path ./out/YourSolution.zip --force
            displayName: 'Deploy to Dev Environment'
```

---

### 📦 **What this template does:**

* **Tool Setup Stage**:

  * Installs .NET SDK (`6.0.x`)
  * Installs specific version of Power Platform PowerShell modules (`2.0.120`)
  * Installs a pinned version of `pac CLI` (`1.32.4`)

* **Build Stage**:

  * Placeholder to pack/export/unpack Dynamics 365 solutions.

* **Deploy Stage**:

  * Sample PAC CLI command to deploy the solution.

---

### ✅ Optional Enhancements:

* Use Azure Key Vault for storing credentials (e.g., for `pac auth create`)
* Use [Service Connections](w) for secure auth instead of interactive login
* Implement environment-specific variables and approvals for UAT/Prod stages

---

Would you like me to convert this pipeline into a **modular multi-stage template setup** or prepare a **PowerShell script to go with it**?
Great! Here's a **modular, multi-stage Azure DevOps pipeline setup** for **Dynamics 365 CI/CD** that includes:

1. A **main pipeline YAML**
2. Separate **template YAMLs** for:

   * Tool installation
   * Build
   * Deploy
3. Optional PowerShell script for tool installation

---

## 📁 Folder Structure

```
/azure-pipelines.yml                  → Main entry pipeline
/templates/
  └ install-tools.yml                 → Tool installation stage
  └ build-solution.yml                → Build stage
  └ deploy-solution.yml               → Deploy stage
/scripts/
  └ install-tools.ps1                 → Optional PS script to install tools
```

---

## 📄 `azure-pipelines.yml` (Main Entry)

```yaml
trigger:
  branches:
    include:
      - main
      - release
      - develop

pool:
  vmImage: 'windows-2019'

variables:
  PowerPlatformToolVersion: '2.0.120'
  DotNetVersion: '6.0.x'
  PacCliVersion: '1.32.4'

stages:
- template: templates/install-tools.yml
- template: templates/build-solution.yml
- template: templates/deploy-solution.yml
```

---

## 📄 `templates/install-tools.yml`

```yaml
parameters:
  PowerPlatformToolVersion: '2.0.120'
  DotNetVersion: '6.0.x'
  PacCliVersion: '1.32.4'

stages:
- stage: ToolSetup
  displayName: 'Install Required Tools'
  jobs:
  - job: InstallTools
    steps:
    - task: UseDotNet@2
      inputs:
        version: ${{ parameters.DotNetVersion }}
      displayName: 'Install .NET SDK'

    - powershell: |
        ./scripts/install-tools.ps1 -ToolVersion "${{ parameters.PowerPlatformToolVersion }}" -PacVersion "${{ parameters.PacCliVersion }}"
      displayName: 'Install Power Platform Tools and pac CLI'

    - powershell: |
        pac --version
        Get-InstalledModule
        $PSVersionTable
      displayName: 'Verify Installed Tools'
```

---

## 📄 `scripts/install-tools.ps1`

```powershell
param (
  [string]$ToolVersion = "2.0.120",
  [string]$PacVersion = "1.32.4"
)

Install-Module -Name Microsoft.PowerApps.Administration.PowerShell -RequiredVersion $ToolVersion -Scope CurrentUser -Force -AllowClobber
Install-Module -Name Microsoft.PowerApps.PowerShell -RequiredVersion $ToolVersion -Scope CurrentUser -Force -AllowClobber

Invoke-WebRequest -Uri "https://aka.ms/paccli" -OutFile ".\pac.msi"
Start-Process msiexec.exe -ArgumentList "/i pac.msi /quiet /norestart" -Wait

pac install latest --version $PacVersion
```

---

## 📄 `templates/build-solution.yml`

```yaml
stages:
- stage: Build
  displayName: 'Build D365 Solution'
  dependsOn: ToolSetup
  jobs:
  - job: BuildSolution
    steps:
    - checkout: self

    - powershell: |
        pac solution list
        # Add commands to export or pack solution as needed
      displayName: 'Build and List Solutions'
```

---

## 📄 `templates/deploy-solution.yml`

```yaml
stages:
- stage: Deploy
  displayName: 'Deploy to Environment'
  dependsOn: Build
  jobs:
  - deployment: DeployToDev
    environment: 'Dev'
    strategy:
      runOnce:
        deploy:
          steps:
          - powershell: |
              pac auth create --url https://your-org.crm.dynamics.com --name DevConnection
              pac solution import --path ./out/YourSolution.zip --force
            displayName: 'Deploy to Dev'
```

---

## ✅ Summary

| Component             | Description                      |
| --------------------- | -------------------------------- |
| `azure-pipelines.yml` | Main entry to run modular stages |
| `install-tools.yml`   | Tool setup with version control  |
| `build-solution.yml`  | Sample stage to build/export     |
| `deploy-solution.yml` | Simple deployment using PAC      |
| `install-tools.ps1`   | Easy-to-update tool installer    |

---

Would you like me to bundle this into a downloadable ZIP or generate a Draw\.io architecture diagram for the pipeline flow?

## deepseek

# Ensuring Continuity in Azure DevOps for Dynamics 365 with Microsoft-Hosted Agents

To maintain your Dynamics 365 application builds and deployments on Microsoft-hosted Windows Server 2019 agents while keeping tool versions up-to-date, follow these strategies:

## 1. Stay Informed About Agent Updates
- **Monitor Microsoft's agent image updates**: Check the [Microsoft-hosted agents documentation](https://github.com/actions/virtual-environments) regularly
- **Review release notes**: Azure DevOps publishes updates about tool version changes

## 2. Implement Version Pinning
For critical tools in your pipeline:
```yaml
steps:
- task: UseDotNet@2
  inputs:
    version: '6.x' # Pin specific version
    includePreview: false
```

## 3. Create a Tool Version Matrix
Test with different versions in parallel:
```yaml
jobs:
- job: TestWithDifferentVersions
  strategy:
    matrix:
      net5:
        dotnetVersion: '5.0.x'
      net6:
        dotnetVersion: '6.0.x'
```

## 4. Implement Upgrade Validation Process
1. **Create a test pipeline** that mirrors production but runs on new agent images first
2. **Add validation steps** that check for compatibility issues
3. **Gradual rollout** to production pipelines after validation

## 5. Dynamics 365 Specific Considerations
- **Power Platform CLI**: Pin to specific versions
  ```yaml
  - task: PowerPlatformToolInstaller@2
    inputs:
      DefaultVersion: false
      Version: '1.14.6' # Specific version
  ```
- **Solution Packager**: Ensure version compatibility with your Dynamics solutions

## 6. Dependency Management
- **Maintain a versions manifest file** in your repo listing all tool versions
- **Use package management** for NuGet, npm, etc. to lock versions

## 7. Monitoring and Alerts
- Set up pipeline alerts for build failures
- Monitor for deprecation notices from Microsoft

## 8. Contingency Plan
- **Create a self-hosted agent option** as fallback
- **Document rollback procedures** for critical tools

## Example Upgrade Strategy for Dynamics 365 Pipeline:
```yaml
# Sample pipeline with version control
trigger:
- main

pool:
  vmImage: 'windows-2019' # Pinned image

steps:
# Tool installations with version pinning
- task: PowerPlatformToolInstaller@2
  inputs:
    DefaultVersion: false
    Version: '1.14.6'

- task: UseDotNet@2
  inputs:
    version: '6.x'
    
# Solution packager step
- task: PowerPlatformExportSolution@2
  inputs:
    authenticationType: 'PowerPlatformSPN'
    PowerPlatformSPN: '$(ServiceConnection)'
    SolutionName: '$(SolutionName)'
    SolutionOutputFile: '$(Build.ArtifactStagingDirectory)\$(SolutionName).zip'
```

By implementing these practices, you can ensure your Dynamics 365 CI/CD pipeline remains stable while keeping tools up-to-date.

## grok 

To ensure that your Dynamics 365 application build and deployment processes continue to work smoothly on a Microsoft-hosted Windows Server 2019 agent in Azure DevOps, you need to manage tool versions effectively, especially given the specific requirements for Dynamics 365 Finance and Operations (D365FO) build automation. Below is a comprehensive guide to upgrading tool versions while maintaining system functionality:

### 1. Understand the Tooling Requirements for Dynamics 365
Dynamics 365 Finance and Operations build automation relies on specific tools and NuGet packages, such as the X++ compiler (`xppc.exe`), MSBuild, and Dynamics 365-specific NuGet packages (e.g., `Microsoft.Dynamics.AX.Platform.CompilerPackage`). These tools are critical for compiling X++ code and creating deployable packages. Microsoft-hosted Windows Server 2019 agents come pre-installed with a set of tools, but you must ensure compatibility with your D365FO application version.

- **Check Installed Software**: Review the software pre-installed on the Windows Server 2019 image by checking the [Microsoft-hosted agents documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=yaml#software). This includes Visual Studio 2019, MSBuild, and NuGet. Ensure these align with your D365FO version requirements.[](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops)
- **NuGet Packages**: Dynamics 365 build pipelines use NuGet packages from the Microsoft Dynamics 365 Lifecycle Services (LCS) Shared Asset Library. These include:
  - `Microsoft.Dynamics.AX.Platform.CompilerPackage` (X++ compiler and tools)
  - `Microsoft.Dynamics.AX.Platform.DevALM.BuildXpp` (Platform Build Reference)
  - `Microsoft.Dynamics.AX.Application.DevALM.BuildXpp` (Application Build Reference)
  - `Microsoft.Dynamics.AX.ApplicationSuite.DevALM.BuildXpp` (Application Suite Build Reference)
  
  Download the correct versions matching your D365FO environment’s Platform Update from LCS.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/dev-tools/hosted-build-automation)[](https://www.encorebusiness.com/blog/build-automation-using-azure-pipeline-microsoft-hosted-agent/)

### 2. Verify Compatibility with Windows Server 2019
Microsoft-hosted Windows Server 2019 agents are being phased out, with support extended until December 31, 2025. After this, you’ll need to transition to Windows Server 2022 or 2025. To ensure continuity:

- **Check Deprecation Notices**: Monitor the [Azure DevOps Blog](https://devblogs.microsoft.com/devops/) for updates on Windows Server 2019 deprecation and new image availability (e.g., Windows Server 2025, available since June 16, 2025).[](https://devblogs.microsoft.com/devops/upcoming-updates-for-azure-pipelines-agents-images/)
- **Test with Windows Server 2022/2025**: Start testing your pipelines with the `windows-2022` or `windows-latest` (currently Windows Server 2022, soon to be Windows Server 2025) VM image to ensure compatibility before the 2019 image is retired. Update your pipeline YAML or classic pipeline to use `vmImage: 'windows-latest'` or `vmImage: 'windows-2022'`.[](https://devblogs.microsoft.com/devops/upcoming-updates-for-azure-pipelines-agents-images/)

### 3. Update NuGet Packages
When upgrading your D365FO environment to a new Platform Update (e.g., 10.0.32 or later), update the NuGet packages in your Azure DevOps pipeline to match:

- **Download Updated Packages**: From LCS, download the latest NuGet packages corresponding to your environment’s version. For example, if upgrading to Platform Update 56, ensure packages like `Microsoft.Dynamics.AX.Platform.CompilerPackage` are version `7.0.7521.60` or newer.[](https://denistrunin.com/d365fo-buildrelease/)
- **Update `packages.config`**: Update the `packages.config` file in your repository to reflect the new package versions. For example:
  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <packages>
    <package id="Microsoft.Dynamics.AX.Platform.CompilerPackage" version="7.0.7521.60" targetFramework="net40" />
    <package id="Microsoft.Dynamics.AX.Platform.DevALM.BuildXpp" version="7.0.7521.60" targetFramework="net40" />
    <package id="Microsoft.Dynamics.AX.Application.DevALM.BuildXpp" version="10.0.2177.37" targetFramework="net40" />
    <package id="Microsoft.Dynamics.AX.ApplicationSuite.DevALM.BuildXpp" version="10.0.2177.37" targetFramework="net40" />
  </packages>
  ```
  Ensure version numbers match your environment.[](https://denistrunin.com/d365fo-buildrelease/)
- **Upload to Azure Artifacts**: Push the updated NuGet packages to your Azure DevOps Artifacts feed using `nuget.exe push` commands, ensuring the pipeline references the correct feed. Use a service account with appropriate permissions for authentication.[](https://denistrunin.com/d365fo-buildrelease/)

### 4. Update Pipeline Configuration
Modify your Azure DevOps pipeline to use the updated tools and packages:

- **Install NuGet Tool**: Ensure the pipeline includes a NuGet Tool Installer task to use the latest NuGet version (e.g., `5.x` or higher). Example YAML:
  ```yaml
  - task: NuGetToolInstaller@1
    inputs:
      versionSpec: '5.x'
  ```
- **Restore NuGet Packages**: Add a task to restore the NuGet packages specified in `packages.config`. Example:
  ```yaml
  - task: NuGetCommand@2
    inputs:
      command: 'restore'
      restoreSolution: '$(Build.SourcesDirectory)\Projects\YourSolution.sln'
      feedsToUse: 'config'
      nugetConfigPath: '$(Build.SourcesDirectory)\Projects\nuget.config'
  ```
- **Update MSBuild Arguments**: Ensure the MSBuild task references the updated NuGet package paths and metadata directories. Example:
  ```yaml
  - task: MSBuild@1
    inputs:
      solution: '$(Build.SourcesDirectory)\Projects\YourSolution.sln'
      msbuildArguments: '/p:BuildTasksDirectory="$(Pipeline.Workspace)\NuGets\Microsoft.Dynamics.AX.Platform.CompilerPackage\DevAlm" /p:MetadataDirectory="$(Build.SourcesDirectory)\Metadata" /p:OutputDirectory="$(Build.BinariesDirectory)"'
  ```
  Verify paths like `$(Pipeline.Workspace)\NuGets` point to the updated packages.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/dev-tools/hosted-build-automation)
- **Set Visual Studio Version**: In the pipeline, set the Visual Studio version to 2022 (available on `windows-2022`) for compatibility with newer D365FO versions. Example:
  ```yaml
  - task: VSBuild@1
    inputs:
      solution: '$(Build.SourcesDirectory)\Projects\YourSolution.sln'
      vsVersion: '17.0' # Visual Studio 2022
  ```
  [](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/dev-tools/hosted-build-automation)

### 5. Test the Pipeline
Before deploying changes to production, test the pipeline to ensure the updated tools work correctly:

- **Run a Test Pipeline**: Create a test pipeline or use a separate branch to validate the build and deployment process with the updated NuGet packages and tools.
- **Verify Deployable Package**: Ensure the pipeline generates a valid deployable package (ZIP file) and upload it to LCS for testing in a sandbox environment.[](https://www.encorebusiness.com/blog/build-automation-using-azure-pipeline-microsoft-hosted-agent/)
- **Check Logs**: Review pipeline logs for errors related to tool versions, missing dependencies, or compatibility issues.

### 6. Handle Agent Image Upgrades
Since Microsoft-hosted agents are updated regularly, ensure your pipeline is resilient to changes:

- **Monitor Agent Software Updates**: Check the [GitHub Actions Runner Images](https://github.com/actions/runner-images) for updates to the Windows Server 2019 image, as it lists installed software versions.[](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops)
- **Pin Tool Versions**: If specific tool versions are required, use the NuGet Tool Installer or script tasks to install exact versions during the pipeline run to avoid breaking changes from agent updates.
- **Transition to Newer Images**: Test your pipeline with `windows-2022` or `windows-latest` to prepare for the eventual retirement of Windows Server 2019. Update the `vmImage` in your YAML pipeline:
  ```yaml
  pool:
    vmImage: 'windows-2022'
  ```

### 7. Manage Azure Artifacts Storage
Dynamics 365 NuGet packages can be large, and Azure Artifacts has a 2 GB free storage limit for free Azure DevOps organizations. To avoid issues:

- **Clean Up Old Packages**: Regularly delete outdated or unused NuGet packages from your Azure Artifacts feed to stay within the free tier limit.[](https://www.encorebusiness.com/blog/build-automation-using-azure-pipeline-microsoft-hosted-agent/)
- **Monitor Usage**: Check Azure Artifacts usage in your Azure DevOps project settings and consider purchasing add-ons if you exceed the limit.

### 8. Automate Version Updates
To streamline future upgrades:

- **Use Variables**: Store NuGet package versions in pipeline variables (e.g., `NuGetVersion`) to make updates easier without hardcoding versions in `packages.config`. Example:
  ```yaml
  variables:
    NuGetVersion: '7.0.7521.60'
  ```
- **Script Version Checks**: Add a PowerShell task to validate that the NuGet package versions match the D365FO environment version before running the build.[](https://learn.microsoft.com/en-us/power-platform/alm/devops-build-tool-tasks)

### 9. Handle Firewall and Authentication
If your organization has firewalls or specific authentication requirements:

- **Configure Firewall Rules**: Ensure the Microsoft-hosted agent’s IP ranges are allowed through your firewall to access LCS and Azure Artifacts. Check the [Azure Pipelines documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=yaml#agent-ip-ranges) for IP ranges.[](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops)
- **Service Connections**: Use a service account with no MFA and an application registration in Azure Active Directory for LCS authentication. Update the service connection in Azure DevOps with the latest client ID and secret if needed.[](https://blog.ateliware.com/step-by-step-to-deploy-d365fo-with-azure-devops/)

### 10. Plan for Future Deprecations
Microsoft is transitioning to newer agent images (e.g., Windows Server 2025). To future-proof your pipeline:

- **Adopt YAML Pipelines**: Use YAML pipelines over classic pipelines for better control and compatibility with future Azure DevOps updates.[](https://learn.microsoft.com/en-us/azure/devops/release-notes/features-timeline)
- **Consider Self-Hosted Agents**: If Microsoft-hosted agents don’t meet your needs (e.g., due to firewall restrictions or specific tool versions), set up self-hosted agents on Windows Server 2022 or later with the exact tool versions required.[](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops)[](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/windows-agent?view=azure-devops)
- **Monitor Roadmap**: Stay updated with the [Azure DevOps Roadmap](https://learn.microsoft.com/en-us/azure/devops/release-notes/roadmap) for upcoming changes to agents and tools.[](https://learn.microsoft.com/en-us/azure/devops/release-notes/features-timeline)

### 11. Backup and Rollback Plan
Before upgrading tools or switching agent images:

- **Backup Configurations**: Save copies of your `nuget.config`, `packages.config`, and pipeline YAML files in your repository.
- **Test in Pre-Production**: Run a dry run of the updated pipeline in a pre-production environment to catch issues.[](https://learn.microsoft.com/en-us/azure/devops/server/upgrade/get-started?view=azure-devops-2022)
- **Rollback Strategy**: If an upgrade fails, revert to the previous `packages.config` and NuGet package versions, and test with the `windows-2019` image.

### Example YAML Pipeline Snippet
Here’s an example YAML pipeline snippet for a D365FO build:
```yaml
pool:
  vmImage: 'windows-2019' # Transition to 'windows-2022' or 'windows-latest' later
variables:
  NuGetVersion: '7.0.7521.60'
steps:
- task: NuGetToolInstaller@1
  inputs:
    versionSpec: '5.x'
- task: NuGetCommand@2
  inputs:
    command: 'restore'
    restoreSolution: '$(Build.SourcesDirectory)\Projects\YourSolution.sln'
    feedsToUse: 'config'
    nugetConfigPath: '$(Build.SourcesDirectory)\Projects\nuget.config'
- task: VSBuild@1
  inputs:
    solution: '$(Build.SourcesDirectory)\Projects\YourSolution.sln'
    vsVersion: '17.0'
    msbuildArgs: '/p:BuildTasksDirectory="$(Pipeline.Workspace)\NuGets\Microsoft.Dynamics.AX.Platform.CompilerPackage\DevAlm" /p:MetadataDirectory="$(Build.SourcesDirectory)\Metadata" /p:OutputDirectory="$(Build.BinariesDirectory)"'
- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: '$(Build.BinariesDirectory)'
    artifactName: 'deployablePackage'
```

### Summary
To ensure your Dynamics 365 build and deployment system continues to work while upgrading tool versions on a Microsoft-hosted Windows Server 2019 agent:
1. Verify tool compatibility with your D365FO version.
2. Update NuGet packages in LCS and `packages.config`.
3. Modify pipeline tasks to use the latest NuGet and Visual Studio versions.
4. Test thoroughly in a non-production environment.
5. Prepare for the transition to Windows Server 2022/2025 by testing with `windows-latest`.
6. Manage Azure Artifacts storage and firewall settings.
7. Automate version checks and maintain a rollback plan.

By following these steps, you can maintain a stable build and deployment process while keeping tools up to date. For pricing or subscription details (e.g., SuperGrok or Azure DevOps premium), visit [https://x.ai/grok](https://x.ai/grok) or [https://help.x.com/en/using-x/x-premium](https://help.x.com/en/using-x/x-premium).[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/dev-tools/hosted-build-automation)[](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops)[](https://www.encorebusiness.com/blog/build-automation-using-azure-pipeline-microsoft-hosted-agent/)

##gemini

Ensuring your Azure DevOps builds and deployments for Dynamics 365 applications continue to work seamlessly when using Microsoft-hosted Windows Server 2019 agents, especially as tool versions evolve, requires a strategic approach. Here's how you can achieve this:

**1. Understand Microsoft-Hosted Agents and Their Updates:**

* **Automatic Updates:** Microsoft-hosted agents are automatically updated by Microsoft. This is a significant advantage as it offloads the burden of maintaining the underlying operating system, build tools, and SDKs. You typically don't manually "upgrade" the Windows Server 2019 OS on these agents; Microsoft handles that.
* **Image Updates:** Microsoft regularly updates the images used for hosted agents (e.g., every 3 weeks). This includes pre-installed software like Node.js, npm, Visual Studio versions, .NET SDKs, etc.
* **Deprecation Schedule:** Be aware of deprecation schedules for older Windows Server images (like Windows Server 2019). While `windows-latest` currently points to `windows-2022`, the `windows-2019` image will eventually be deprecated. You'll need to monitor Microsoft's announcements for this.

**2. Leverage the Right Azure DevOps Tools for Dynamics 365:**

* **Microsoft Power Platform Build Tools:** These are crucial for automating build and deployment tasks for Dynamics 365/Power Platform solutions.
    * **Installation:** Ensure you have the Microsoft Power Platform Build Tools extension installed in your Azure DevOps organization from the Azure Marketplace.
    * **Helper Task:** Always include the "Power Platform Tool Installer" task at the beginning of your agent phase in your build pipelines. This task ensures the necessary tools for the Power Platform Build Tasks are available and correctly versioned on the agent. It has parameters for specific tool versions, which you should update as needed.
* **Power DevOps Tools (Alternative/Complementary):** While the Microsoft Power Platform Build Tools are the official offering, "Power DevOps Tools" (also known as Dynamics 365 Build Tools by Wael Hamze) is another popular marketplace extension that provides a comprehensive set of tasks for Dynamics 365 CE/Power Apps/CDS solution automation. If you're using this, keep it updated.

**3. Pipeline Design for Tool Version Management:**

* **Specify Tool Versions in Pipelines (if applicable):** While Microsoft-hosted agents come with pre-installed tools, some tasks allow you to specify exact versions (e.g., `UseDotNet@2` for .NET SDK). Use this capability to ensure consistency, especially if you have dependencies on specific tool versions.
* **Use Tool Installer Tasks:** For tools that aren't inherently managed by the Power Platform Build Tools installer or are not part of the standard agent image, leverage Azure DevOps Tool Installer tasks (e.g., Node.js Tool Installer, .NET SDK Installer). These tasks fetch and install the specified version of the tool on the agent at runtime, ensuring your pipeline uses the correct version regardless of what's pre-installed.
* **Pin to Specific Agent Images (with caution):** While you currently use Windows Server 2019, it's generally recommended to use `windows-latest` as the agent specification in your YAML pipelines. This allows you to automatically benefit from the latest Windows Server image (currently Windows Server 2022, soon Windows Server 2025). If you *must* use `windows-2019` for compatibility reasons, be aware of its upcoming deprecation and plan your migration to a newer image proactively.

**4. Proactive Monitoring and Testing:**

* **Monitor Microsoft Announcements:** Keep an eye on the Azure DevOps release notes and Microsoft Learn documentation for updates to Microsoft-hosted agent images, deprecation schedules, and new versions of Power Platform Build Tools.
* **Regular Pipeline Runs:** Run your build and release pipelines frequently. This helps you identify any breaking changes or inconsistencies introduced by agent updates or tool version changes early.
* **Staging/Pre-Production Environments:** Always test your builds and deployments in non-production environments (e.g., UAT, Sandbox) before deploying to production. This allows you to catch any issues caused by tool version changes without impacting live systems.
* **Automated Testing:** Implement comprehensive automated tests (unit tests, integration tests, UI tests) for your Dynamics 365 application. These tests are invaluable for verifying functionality after any environment or tool changes.
* **Health Checks in Pipelines:** Consider adding tasks to your pipelines that perform basic "health checks" of the environment or installed tools to ensure they are at expected versions or have the necessary components.

**5. Consider Self-Hosted Agents for Granular Control (If Necessary):**

* **When to Use Self-Hosted Agents:** If you have very specific and rigid requirements for tool versions, custom software, or a need to isolate your build environment from Microsoft's update cycles, consider setting up self-hosted agents.
* **Maintenance Overhead:** Be aware that self-hosted agents come with the responsibility of managing and updating the operating system, all build tools, and any dependencies yourself. This can be a significant overhead.
* **Controlled Updates:** With self-hosted agents, you have complete control over when and how tools are updated, allowing you to thoroughly test before rolling out changes.

**In summary, the key to ensuring continuity when using Microsoft-hosted Windows Server 2019 agents for Dynamics 365 builds and deployments is to embrace the automatic updates provided by Microsoft for the agent itself, leverage the Power Platform Build Tools with their helper tasks, and maintain a proactive approach to monitoring, testing, and adapting your pipelines to any changes in the underlying tools or agent images.**