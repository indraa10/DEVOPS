# question : for ms dynamics 365 application / solution build and deployment using azure devops team is facing issues while upgrading dependencies , team is able to make the minor upgrades easily however during major upgrade . team is looking for sop  or specific method to tackle the issue .

# chatgpt : 

Upgrading **Microsoft Dynamics 365** applications (especially **major version upgrades**) can be **complex**, involving schema changes, deprecated APIs, and solution compatibility issues. While **minor version upgrades** often work smoothly, **major upgrades require careful planning and validated execution**.

Here’s a comprehensive **Standard Operating Procedure (SOP)** and best practices to handle major dependency upgrades in **Azure DevOps-based CI/CD pipelines** for Dynamics 365:

---

## ✅ SOP for Handling Major Version Upgrades in Dynamics 365 Solutions Using Azure DevOps

---

### **1. Pre-Upgrade Planning**

#### 🔍 Step 1: **Review Microsoft Release Notes**

* Identify breaking changes, deprecated features, and new requirements.
* Verify supported versions for all components (e.g., Dataverse, plugins, Power Apps CLI, packages).

#### 🧪 Step 2: **Assess Solution Compatibility**

* Export existing solutions as unmanaged and import them into a test environment with the new version.
* Use [Solution Checker](w) and [Power Platform Tools](w) to identify issues.

#### 📦 Step 3: **Pin Tool Versions in Pipeline**

* Avoid using `latest` in tools like Power Platform CLI or NuGet packages.
* Explicitly version dependencies in YAML or PowerShell scripts:

  ```yaml
  - task: PowerPlatformToolInstaller@0
    inputs:
      DefaultVersion: '1.21.4'  # pin exact version
  ```

---

### **2. CI/CD Process Preparation**

#### 🧱 Step 4: **Isolate Upgrade Branch**

* Create a dedicated branch: `upgrade/vX.Y`
* Apply changes incrementally, allowing merge back post-validation.

#### 🧪 Step 5: **Spin Up a Dedicated UAT/Upgrade Environment**

* Clone Production or latest UAT.
* Deploy latest schema, plugins, and test scripts.

---

### **3. Major Upgrade Execution (Dev → SIT → UAT)**

#### 🚀 Step 6: **Update Dependencies via Package Managers**

* Update NuGet or npm packages in plugin/custom code projects.
* Use compatibility matrix from Microsoft to upgrade SDK packages.

#### 🔧 Step 7: **Update YAML Pipeline for Compatibility**

* Update Power Platform CLI, Plugin Registration Tool, and PowerShell modules:

  ```yaml
  - task: PowerPlatformToolInstaller@0
    inputs:
      DefaultVersion: '1.22.0'
  ```

#### 🧹 Step 8: **Clean Old Build Artifacts**

* Ensure solution `.zip` and plugin `.dll` files are rebuilt from scratch.
* Use `dotnet clean` or `msbuild /t:Clean`.

---

### **4. Post-Upgrade Validation**

#### 🧪 Step 9: **Run Regression + Integration Tests**

* Use automated tests (XrmToolBox, EasyRepro, etc.) for UI and API validation.
* Validate business rules, workflows, and Power Automate flows.

#### 🔁 Step 10: **Dry Run Production Deployment**

* Export solution and test import in a sandbox Production clone.
* Validate plugin execution, form rendering, data integrity.

---

### **5. Production Rollout**

#### 🚦 Step 11: **Schedule Maintenance Window**

* Communicate upgrade downtime to stakeholders.
* Export existing solutions for backup.

#### 📤 Step 12: **Deploy via Azure DevOps**

* Use **`ImportSolution` task** with upgrade path:

  ```yaml
  - task: PowerPlatformImportSolution@0
    inputs:
      environmentUrl: 'https://yourorg.crm.dynamics.com'
      solutionInputFile: 'bin/Release/YourSolution.zip'
      importUnmanagedSolution: false
      forceOverwrite: true
      publishWorkflows: true
  ```

#### 🔄 Step 13: **Publish All Customizations**

* Add task or script:

  ```yaml
  - task: PowerPlatformPublishCustomizations@0
    inputs:
      environmentUrl: '$(envUrl)'
  ```

---

### 🛠️ Additional Tips

| Area           | Best Practice                                                           |
| -------------- | ----------------------------------------------------------------------- |
| Source Control | Use **unmanaged solutions** for tracking in Azure Repos                 |
| Tools          | Use **Pac CLI** (`pac solution upgrade`, `pac admin`) for compatibility |
| Feature Flags  | Disable breaking features via **environment variables** if necessary    |
| Rollback       | Keep full backups of previous managed solution                          |
| Monitor        | Use Application Insights + Azure Logs for post-deploy monitoring        |

---

### 📄 Deliverables

Would you like:

* A sample **upgrade branch pipeline YAML**
* A **checklist document** or Excel tracker for upgrade steps
* A **PowerShell fallback script** for offline import

Let me know which format you want this SOP in: Word, Excel, Markdown, or pipeline-ready YAML.


Here is the **pipeline-ready YAML** and **Markdown-formatted SOP document** for managing **major version upgrades** of **Dynamics 365 applications** using **Azure DevOps**:

---

## ✅ 1. Pipeline-Ready YAML for Dynamics 365 Major Upgrade

```yaml
trigger:
  branches:
    include:
      - upgrade/*

pool:
  vmImage: 'windows-latest'

variables:
  environmentUrl: 'https://your-org.crm.dynamics.com'
  solutionFile: 'solutions/YourSolution.zip'
  solutionName: 'YourSolution'

steps:

# Step 1: Install Power Platform Tools (pinned version)
- task: PowerPlatformToolInstaller@0
  inputs:
    DefaultVersion: '1.21.14'  # Replace with latest verified version for upgrade

# Step 2: Publish customizations (optional pre-clean)
- task: PowerPlatformPublishCustomizations@0
  inputs:
    environmentUrl: '$(environmentUrl)'

# Step 3: Backup existing solution (retrieve and store unmanaged)
- task: PowerPlatformExportSolution@0
  inputs:
    authenticationType: 'PowerPlatformSPN'
    environmentUrl: '$(environmentUrl)'
    solutionName: '$(solutionName)'
    managed: false
    targetFolder: 'backup'
    runAsync: false

# Step 4: Import upgraded solution
- task: PowerPlatformImportSolution@0
  inputs:
    authenticationType: 'PowerPlatformSPN'
    environmentUrl: '$(environmentUrl)'
    solutionInputFile: '$(solutionFile)'
    importUnmanagedSolution: false
    forceOverwrite: true
    publishWorkflows: true

# Step 5: Publish all customizations post-deploy
- task: PowerPlatformPublishCustomizations@0
  inputs:
    environmentUrl: '$(environmentUrl)'

# Step 6: Run solution checker (optional but recommended)
- task: PowerPlatformChecker@0
  inputs:
    environmentUrl: '$(environmentUrl)'
    ruleset: 'SolutionChecker'
    fileInput: '$(solutionFile)'
```

> 🔐 Replace `PowerPlatformSPN` with your Azure DevOps service connection (use App Registration with required `Dynamics CRM` API permissions).

---

## 📄 2. Markdown SOP for Major Dependency Upgrade – Dynamics 365 + Azure DevOps

```markdown
# 📘 SOP: Handling Major Version Upgrades for Dynamics 365 via Azure DevOps

---

## 1. 📌 Pre-Upgrade Planning

- Review Microsoft release notes for major versions.
- Identify deprecated APIs, new breaking changes.
- Check compatibility of current custom plugins, PCF controls, etc.
- Ensure CI/CD tools (e.g., Power Platform CLI, NuGet packages) are updated to compatible versions.

---

## 2. 🏗️ Pipeline Branch Preparation

- Create upgrade branch: `upgrade/vX.Y`
- Build in isolation before merging to `main` or `release`
- Update pipeline variables and tools:
  - Pin exact versions (`PowerPlatformToolInstaller`, SDKs)

---

## 3. 🧪 Create UAT/Test Upgrade Environment

- Clone production data (or UAT backup)
- Import solution manually for initial validation
- Run full regression testing

---

## 4. 🛠️ Update Solution & Dependencies

- Update NuGet packages for plugins:
  - Microsoft.CrmSdk.CoreAssemblies
  - Microsoft.PowerPlatform.Dataverse.Client
- Rebuild all plugins
- Export as managed solution (from upgraded environment)

---

## 5. 🔁 CI/CD Pipeline Execution

### Steps:

1. **Install Power Platform Tools**
2. **Backup current solution (unmanaged export)**
3. **Import upgraded solution**
4. **Publish customizations**
5. **Run Solution Checker**

---

## 6. 🧹 Post-Upgrade Validation

- Verify:
  - Form loads
  - Plugin execution
  - Business rules
  - PCF controls
  - API integrations
- Perform user testing in upgraded UAT

---

## 7. 🚀 Production Deployment

- Schedule maintenance window
- Backup production solutions (managed + unmanaged)
- Deploy using Azure DevOps pipeline
- Validate immediately post-deploy

---

## 8. 🔄 Rollback Plan

- Keep previous version `.zip` backups in DevOps artifacts
- Maintain rollback pipeline to import previous version if needed
- Store backup solution zip in Azure Blob or Repos

---

## 9. 📈 Monitoring & Alerts

- Enable Application Insights or Log Analytics
- Alert for:
  - Plugin errors
  - Import failures
  - Missing components

---

## 🔑 Notes

- Always avoid using `latest` in SDKs or tools
- Pin `Power Platform CLI`, `pac`, and DevOps tasks to tested versions
- Use `Solution Checker` before and after each major upgrade
```

---

Let me know if you'd like this content exported as a downloadable `.md`, `.yaml`, or `.docx` file — or embedded in an Azure DevOps wiki page format.


# deepseek : 

# Standard Operating Procedure (SOP) for Dynamics 365 Solution Major Dependency Upgrades Using Azure DevOps

## Purpose
This SOP provides a structured approach for handling major dependency upgrades in Dynamics 365 solutions deployed via Azure DevOps, minimizing downtime and regression risks.

## 1. Pre-Upgrade Preparation

### 1.1 Impact Assessment
- [ ] Conduct dependency audit using `pac solution list` command
- [ ] Map all dependent components and their relationships
- [ ] Identify breaking changes in major version release notes
- [ ] Document integration points affected by the upgrade

### 1.2 Environment Setup
- [ ] Provision isolated sandbox environment labeled "Upgrade Test"
- [ ] Clone production solution to test environment using:
  ```powershell
  pac solution clone --name "YourSolution" --version "1.0.0" --target "UpgradeTest"
  ```

### 1.3 Backup Procedures
- [ ] Export current solution package from all affected environments
- [ ] Backup source code repository branch
- [ ] Create Azure DevOps pipeline snapshot

## 2. Upgrade Execution Process

### 2.1 Dependency Upgrade Steps
1. **Update NuGet packages**:
   ```powershell
   Update-Package Microsoft.PowerPlatform.Dataverse.Client -Version [new-version]
   Update-Package Microsoft.PowerPlatform.Dataverse.Utils -Version [new-version]
   ```

2. **Update Power Platform CLI**:
   ```bash
   npm update -g @microsoft/powerplatform-cli
   ```

3. **Update SDK References**:
   - Update Microsoft.CrmSdk.* packages in all projects
   - Verify assembly binding redirects in web.config/app.config

### 2.2 Solution Upgrade Methodology

#### Phased Approach:
1. **Phase 1 - Core Components**:
   ```powershell
   pac solution sync --name "CoreComponents" --process "Upgrade"
   ```

2. **Phase 2 - Extended Components**:
   ```powershell
   pac solution sync --name "ExtendedFeatures" --process "Upgrade"
   ```

3. **Phase 3 - Customizations**:
   ```powershell
   pac solution sync --name "CustomComponents" --process "Upgrade"
   ```

### 2.3 Azure DevOps Pipeline Modifications
1. Update pipeline variables for new versions:
   ```yaml
   variables:
     PowerPlatformCliVersion: '1.2.3'
     DataverseSdkVersion: '9.1.0'
   ```

2. Add validation step:
   ```yaml
   - task: PowerShell@2
     name: ValidateDependencies
     inputs:
       script: |
         # Verify all assemblies are compatible
         pac solution check-dependencies --report-format json
   ```

## 3. Testing Protocol

### 3.1 Automated Tests
- [ ] Execute all unit tests with new dependencies
- [ ] Run integration test suite
- [ ] Validate plugin registration:
  ```powershell
  pac plugin validate --assembly "bin\Debug\YourPlugin.dll"
  ```

### 3.2 Manual Test Matrix
| Test Area          | Validation Steps                          | Success Criteria |
|--------------------|-------------------------------------------|------------------|
| API Endpoints      | Verify all custom APIs respond            | HTTP 200/204     |
| Business Processes | Test all workflows/business rules         | Expected output  |
| UI Customizations  | Validate forms/views/web resources        | Proper rendering |
| Integrations       | Test all external system connections      | Data syncs       |

## 4. Rollback Procedure

### 4.1 Automated Rollback Trigger
Conditions initiating rollback:
- More than 15% test failure rate
- Critical functionality broken
- Performance degradation > 30%

### 4.2 Rollback Steps
1. Revert Azure DevOps release:
   ```powershell
   pac solution deploy --name "PreviousVersion" --activate
   ```

2. Restore NuGet packages:
   ```powershell
   Install-Package Microsoft.PowerPlatform.Dataverse.Client -Version [old-version]
   ```

3. Reactivate previous pipeline version

## 5. Post-Upgrade Activities

### 5.1 Monitoring
- [ ] Configure Application Insights alerts for 48 hours post-upgrade
- [ ] Monitor Dataverse API throttling
- [ ] Track plugin execution times

### 5.2 Documentation Update
- [ ] Update solution version in all manifests
- [ ] Revise deployment runbooks
- [ ] Archive previous version artifacts

## 6. Special Considerations for Major Upgrades

### 6.1 Breaking Change Mitigation
1. **API Version Transition**:
   ```csharp
   // Use new ServiceClient constructor for v9+
   var service = new ServiceClient(new Uri(url), tokenProvider);
   ```

2. **Deprecated Feature Replacement**:
   - Replace OrganizationServiceProxy with IOrganizationService
   - Update all uses of deprecated messages

### 6.2 Performance Optimization
1. Implement batch processing for data operations:
   ```csharp
   var batchRequest = new ExecuteTransactionRequest()
   {
       Requests = new OrganizationRequestCollection()
   };
   ```

2. Configure connection pooling:
   ```csharp
   var service = new ServiceClient(connectionString)
   {
       MaxConnectionLimit = 10
   };
   ```

## 7. Recommended Upgrade Cadence

| Upgrade Type | Frequency   | Recommended Approach               |
|--------------|-------------|------------------------------------|
| Minor        | Monthly     | Direct update in DEV, pipeline test|
| Major        | Bi-annually | Follow full SOP with 2-week cycle  |
| Critical     | As needed   | Emergency patch process            |

This SOP ensures controlled major dependency upgrades while maintaining system stability. Teams should adjust timeframes based on solution complexity.

# grok : 

Upgrading dependencies in a Microsoft Dynamics 365 application during major upgrades using Azure DevOps can be complex due to the intricate dependencies between application components, platform updates, and customizations. Major upgrades often involve significant changes to the application version, platform, or database schema, which can introduce compatibility issues, breaking changes, or deployment failures. Below is a detailed **Standard Operating Procedure (SOP)** for upgrading dependencies in a Dynamics 365 application using Azure DevOps during major upgrades, addressing the challenges and providing a structured approach to mitigate issues.

---

### Standard Operating Procedure (SOP) for Upgrading Dependencies in Microsoft Dynamics 365 Using Azure DevOps During Major Upgrades

#### Objective
To provide a standardized, repeatable process for upgrading dependencies in a Microsoft Dynamics 365 application (e.g., Finance and Operations or Business Central) during major upgrades, ensuring minimal downtime, compatibility, and successful deployment using Azure DevOps.

#### Scope
This SOP applies to major upgrades of Dynamics 365 applications, including application and platform updates, custom code, and third-party dependencies, deployed via Azure DevOps pipelines to cloud-hosted or Microsoft-managed environments.

---

### Prerequisites
1. **Azure DevOps Setup**:
   - An Azure DevOps project with configured source control (e.g., Git or TFVC).
   - Pipelines for build and release, including the Dynamics 365 Finance and Operations Tools extension.[](https://blog.ateliware.com/step-by-step-to-deploy-d365fo-with-azure-devops/)
   - Service connections to Microsoft Dynamics Lifecycle Services (LCS) and Azure resources.
2. **Environment Access**:
   - Access to LCS for downloading update packages and managing environments.
   - Cloud-hosted or Microsoft-managed environments (development, sandbox, production).
3. **Backup**:
   - A full backup of the production database and source code.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/data-upgrade-2012)
   - A sandbox environment for testing upgrades.
4. **Permissions**:
   - Administrative access to Azure DevOps, LCS, and Dynamics 365 environments.
   - A service account with no MFA for Azure DevOps pipelines.[](https://blog.ateliware.com/step-by-step-to-deploy-d365fo-with-azure-devops/)
5. **Tools**:
   - Visual Studio with Dynamics 365 development tools.
   - PowerShell and Azure CLI for scripting and automation.
   - NuGet CLI for package management.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/dev-tools/hosted-build-automation)

---

### Step-by-Step Analysis of Issues During Major Upgrades

#### Common Challenges
1. **Dependency Conflicts**:
   - Major upgrades may introduce breaking changes in APIs, metadata, or platform components, causing custom code or third-party extensions to fail.
   - NuGet packages or Retail SDK dependencies may not align with the target version.[](https://learn.microsoft.com/en-us/dynamics365/commerce/dev-itpro/new-environments-visual-studio-teams-branch-retail-projects)
2. **Database Schema Changes**:
   - Data upgrade processes may fail due to schema incompatibilities or missing hotfixes.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/self-service-upgrade)
3. **Build and Deployment Failures**:
   - Pipelines may fail due to incorrect package versions, missing configurations, or validation errors in LCS.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/deployment/apply-deployable-package-system)
4. **Cache and Configuration Issues**:
   - Cached metadata or outdated configurations can cause deployment errors, requiring manual intervention.
5. **Environment Limitations**:
   - Limited disk space or resource constraints in cloud-hosted environments can cause failures during data upgrades.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/data-upgrade-2012)

#### Root Cause Analysis
- **Inadequate Testing**: Major upgrades require thorough testing in a sandbox environment, which may be skipped or incomplete.
- **Version Mismatch**: NuGet packages or platform versions may not match the target environment’s version.
- **Custom Code**: Customizations may not be compatible with the new version, requiring code adjustments.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/upgrade-data-to-latest-update)
- **Pipeline Configuration**: Incorrect pipeline tasks or missing parameters (e.g., license files, solution paths) can lead to build failures.[](https://medium.com/%40zia_ur_rahman_dynamics_geek/implementci-cd-in-d365-with-azure-devops-build-pipeline-1a718dac3f03)
- **Data Upgrade Errors**: Missing hotfixes or unsynchronized data (e.g., Commerce Data Exchange jobs) can cause data upgrade failures.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/upgrade-data-to-latest-update)

---

### SOP: Steps for Upgrading Dependencies During Major Upgrades

#### Step 1: Plan the Upgrade
1. **Identify Target Version**:
   - Check the target version of Dynamics 365 in LCS (e.g., 10.0.26). Review release notes for breaking changes and required hotfixes.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/self-service-upgrade)
   - Verify compatibility with custom code and third-party extensions.
2. **Schedule the Upgrade**:
   - Plan the upgrade during a maintenance window to minimize downtime. Use LCS to schedule updates for Microsoft-managed environments.[](https://www.encorebusiness.com/blog/how-are-upgrades-different-in-dynamics-365/)
   - Notify stakeholders and set up a rollback plan.
3. **Prepare Environments**:
   - Deploy a sandbox environment for testing, preferably a Tier 2 or higher environment for realistic testing.[](https://learn.microsoft.com/en-us/dynamics365/commerce/dev-itpro/new-environments-visual-studio-teams-branch-retail-projects)
   - Back up the production database using SQL Server Management Studio or AzCopy for Azure Blob Storage.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/upgrade-data-to-latest-update)
4. **Document Dependencies**:
   - List all NuGet packages, Retail/Commerce SDKs, and third-party dependencies in the current environment.
   - Check version compatibility with the target version using LCS Shared Asset Library.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/upgrade-data-to-latest-update)

#### Step 2: Update Source Control and Dependencies
1. **Create a New Branch**:
   - In Azure DevOps, create a new branch (e.g., `Dev_Upgrade_10.0.26`) from the main branch for upgrade-related changes.[](https://learn.microsoft.com/en-us/dynamics365/commerce/dev-itpro/new-environments-visual-studio-teams-branch-retail-projects)
   - Map the branch to the local model store and projects folder in Visual Studio.[](https://community.dynamics.com/blogs/post/?postid=e11c8506-e1a1-47d5-bbe1-bd3054664444)
2. **Update NuGet Packages**:
   - Download the latest NuGet packages for the target version from LCS Shared Asset Library (e.g., `Microsoft.Dynamics.AX.Application.DevALM.BuildXpp`, `Microsoft.Dynamics.AX.Platform.CompilerPackage`).[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/dev-tools/hosted-build-automation)
   - Update the `nuget.config` and `packages.config` files in the repository with the correct version numbers. Example `packages.config`:
     ```xml
     <?xml version="1.0" encoding="utf-8"?>
     <packages>
       <package id="Microsoft.Dynamics.AX.Application.DevALM.BuildXpp" version="10.0.2177.37" targetFramework="net40" />
       <package id="Microsoft.Dynamics.AX.Platform.DevALM.BuildXpp" version="7.0.7521.60" targetFramework="net40" />
     </packages>
     ```
    [](https://denistrunin.com/d365fo-buildrelease/)
   - Run `nuget restore` to update dependencies in the build environment.
3. **Integrate SDKs**:
   - If using Commerce, migrate from Retail SDK to Commerce SDK (Retail SDK support ended October 2023).[](https://learn.microsoft.com/en-us/dynamics365/commerce/dev-itpro/new-environments-visual-studio-teams-branch-retail-projects)
   - Update the build definition to include the Commerce SDK as per Microsoft’s guidelines.[](https://learn.microsoft.com/en-us/dynamics365/commerce/dev-itpro/new-environments-visual-studio-teams-branch-retail-projects)
4. **Apply Hotfixes**:
   - Install required hotfixes listed in the upgrade documentation for the target version. Check these into Azure DevOps.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/self-service-upgrade)
   - For example, run SQL scripts to fix known issues like `RELATIONTYPE` errors.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/upgrade-data-to-latest-update)

#### Step 3: Configure Azure DevOps Pipelines
1. **Update Build Pipeline**:
   - Import or update the build pipeline using the `xpp-classic-ci.json` template from Microsoft’s GitHub repository.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/dev-tools/hosted-build-automation)
   - Configure the pipeline to use the correct branch, agent pool (e.g., `windows-latest`), and solution path.[](https://medium.com/%40zia_ur_rahman_dynamics_geek/implementci-cd-in-d365-with-azure-devops-build-pipeline-1a718dac3f03)
   - Add tasks to restore NuGet packages, build the solution, and generate a deployable package:
     ```yaml
     steps:
     - task: NuGetCommand@2
       inputs:
         command: 'restore'
         restoreSolution: '$(Build.SourcesDirectory)/Projects/AxBuild.sln'
     - task: VSBuild@1
       inputs:
         solution: '$(Build.SourcesDirectory)/Projects/AxBuild.sln'
         msbuildArgs: '/p:Configuration=Release'
     ```
2. **Update Release Pipeline**:
   - Create a new release pipeline or update an existing one. Add tasks for:
     - Installing MSAL.PS for authentication.
     - Uploading the package to LCS using the Dynamics LCS Asset Upload task.
     - Deploying the package to LCS using the Dynamics LCS Asset Deployment task.[](https://denistrunin.com/d365fo-buildrelease/)
   - Example task configuration:
     ```yaml
     - task: DynamicsLCSAssetUpload@1
       inputs:
         connection: 'LCS_Service_Connection'
         assetName: 'MajorVersionDataUpgrade'
     ```
   - Disable “Wait for Completion” to avoid timeouts.[](https://denistrunin.com/d365fo-buildrelease/)
3. **Set Environment Variables**:
   - Add variables for LCS secrets (e.g., service account credentials, client ID) in the pipeline, stored securely in Azure Key Vault.[](https://learn.microsoft.com/en-us/azure/architecture/example-scenario/devops/manage-microsoft-365-tenant-configuration-microsoft365dsc-devops)

#### Step 4: Test the Upgrade in a Development Environment
1. **Deploy Development Environment**:
   - Use LCS to deploy a cloud-hosted development environment with the target version.[](https://learn.microsoft.com/en-us/dynamics365/commerce/dev-itpro/new-environments-visual-studio-teams-branch-retail-projects)
   - Restore the production database backup to the development environment using AzCopy for faster transfer.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/upgrade-data-to-latest-update)
2. **Run Data Upgrade**:
   - Download the data upgrade package (e.g., `AX2012DataUpgrade-10.0.26`) from LCS and extract it to `C:\Temp`.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/data-upgrade-2012)
   - Stop AOS services to avoid locking issues.
   - Run the data upgrade using PowerShell:
     ```powershell
     .\AXUpdateInstaller.exe generate -runbookid="MajorVersionDataUpgrade-runbook" -topologyfile="DefaultTopologyData.xml" -servicemodelfile="DefaultServiceModelData.xml" -runbookfile="MajorVersionDataUpgrade-runbook.xml"
     .\AXUpdateInstaller.exe import -runbookfile="MajorVersionDataUpgrade-runbook.xml"
     .\AXUpdateInstaller.exe execute -runbookid="MajorVersionDataUpgrade-runbook"
     ```
    [](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/data-upgrade-2012)
3. **Validate Custom Code**:
   - Test customizations and extensions against the upgraded dataset. Debug and adjust code as needed.
   - Run unit tests to ensure functionality.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/dev-tools/hosted-build-automation)

#### Step 5: Deploy to Sandbox Environment
1. **Validate Package**:
   - Upload the deployable package to LCS Asset Library and validate it. Ensure the package status is “Valid” before deployment.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/deployment/apply-deployable-package-system)
2. **Apply Upgrade**:
   - Use LCS to apply the package to the sandbox environment. Select the “Upgrade” option to roll up previous patches and remove outdated components.[](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/update-solutions)
   - Monitor the deployment for errors. If it fails, download logs from LCS to identify the root cause.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/self-service-upgrade)
3. **Update Aggregate Measurements**:
   - After the upgrade, refresh aggregate measurements in System Administration > Setup > Entity Store.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/self-service-upgrade)
4. **Test in Sandbox**:
   - Perform user acceptance testing (UAT) to validate functionality, data integrity, and performance.
   - Check Commerce Data Exchange (CDX) jobs to ensure no unsynchronized transactional data exists.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/upgrade-data-to-latest-update)

#### Step 6: Deploy to Production
1. **Schedule Production Upgrade**:
   - Schedule the upgrade in LCS during a planned maintenance window.[](https://www.encorebusiness.com/blog/how-are-upgrades-different-in-dynamics-365/)
   - Notify users and stakeholders of expected downtime.
2. **Apply Package**:
   - Use the same validated package from the sandbox environment. Apply it via LCS with the “Upgrade” option.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/deployment/apply-deployable-package-system)
   - Monitor the deployment and verify completion via email notification from LCS.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/self-service-upgrade)
3. **Post-Upgrade Tasks**:
   - Refresh aggregate measurements again.
   - Validate business processes and customizations.
   - Monitor logs for errors using Azure Log Analytics or Application Insights.

#### Step 7: Rollback Plan
- If the data upgrade fails, select “Rollback” in LCS to restore the database to its pre-upgrade state. Download logs to diagnose the issue before retrying.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/self-service-upgrade)
- Maintain a backup of the source database and deployable package for quick recovery.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/upgrade-data-to-latest-update)

#### Step 8: Monitor and Optimize
- **Set Up Alerts**: Use Azure Monitor to set alerts for deployment failures or resource issues.
- **Clean Up**: Deallocate unused sandbox environments after 10 days to avoid automatic deletion.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/self-service-upgrade)
- **Document Lessons Learned**: Update the SOP with any issues encountered and solutions applied.

---

### Mitigation Steps for Common Issues
1. **Dependency Conflicts**:
   - Use the correct NuGet package versions from LCS. Verify versions in `packages.config` match the target environment.[](https://denistrunin.com/d365fo-buildrelease/)
   - Test third-party extensions in a development environment before deployment.
2. **Database Schema Errors**:
   - Apply required hotfixes and run SQL scripts to fix schema issues (e.g., `RELATIONTYPE` fix).[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/upgrade-data-to-latest-update)
   - Ensure the database collation is `SQL_Latin1_General_CP1_CI_AS`.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/data-upgrade-2012)
3. **Pipeline Failures**:
   - Validate pipeline tasks and parameters. Ensure the correct solution path and license files are specified.[](https://medium.com/%40zia_ur_rahman_dynamics_geek/implementci-cd-in-d365-with-azure-devops-build-pipeline-1a718dac3f03)
   - Use Microsoft-hosted agents to avoid resource constraints.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/dev-tools/hosted-build-automation)
4. **Cache Issues**:
   - Clear metadata cache in the development environment before testing.
   - Configure pipelines to clean the workspace before builds (`clean: true` in YAML).
5. **Resource Constraints**:
   - Use premium storage for cloud-hosted environments to improve performance.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/data-upgrade-2012)
   - Monitor host disk space usage and restart the environment if needed.

---

### Best Practices
- **Automate Dependency Checks**: Use Dependabot in Azure DevOps to monitor and update dependencies automatically. Configure it to target the `develop` branch and limit open pull requests.[](https://www.freecodecamp.org/news/keep-package-dependencies-up-to-date-on-azure-devops/)
- **Test Incrementally**: Test minor updates in development environments before attempting major upgrades.
- **Use Source Control**: Maintain all custom code and configurations in Azure DevOps for traceability.[](https://community.dynamics.com/blogs/post/?postid=e11c8506-e1a1-47d5-bbe1-bd3054664444)
- **Leverage LCS**: Use LCS for package validation and environment management to reduce errors.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/deployment/apply-deployable-package-system)
- **Document Changes**: Maintain a changelog for all dependency updates and customizations.

---

### Additional Notes
- **Commerce SDK**: If using Dynamics 365 Commerce, ensure migration to the Commerce SDK, as Retail SDK is deprecated.[](https://learn.microsoft.com/en-us/dynamics365/commerce/dev-itpro/new-environments-visual-studio-teams-branch-retail-projects)
- **Community Support**: Engage with the Dynamics 365 Community Forum (https://community.dynamics.com) for troubleshooting specific errors.[](https://community.dynamics.com/forums/thread/?redirectedFrom=https://community.dynamics.com/365/f/dynamics-365-general-forum/305090/dynamics-365-devops-supported-by-microsoft)
- **Microsoft Support**: If issues persist, raise a support ticket via LCS or contact the Dynamics Service Engineering (DSE) team.[](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/migration-upgrade/self-service-upgrade)

This SOP provides a structured approach to handle major upgrades in Dynamics 365 using Azure DevOps, addressing the team’s challenges with dependency management and deployment. Let me know if you need specific pipeline configurations or further assistance with troubleshooting!

# Gemini : 

Major upgrades for Dynamics 365 applications and solutions, especially when managing dependencies with Azure DevOps, can be significantly more complex than minor upgrades. The reason is often a combination of breaking changes in the platform, deprecated components, stricter dependency enforcement, and the sheer volume of changes across multiple layers (platform, applications, custom solutions, third-party solutions).

Here's a comprehensive Standard Operating Procedure (SOP) and specific methods to tackle these issues:

## Understanding Dynamics 365 Upgrade Types

Before diving into the SOP, it's crucial to distinguish between upgrade types:

* **Minor Upgrades (Service Updates/Hotfixes):** These are typically regular "One Version" updates from Microsoft (e.g., from 10.0.X to 10.0.Y). They usually involve bug fixes, performance improvements, and small feature additions. Backward compatibility is generally maintained, and dependency changes are often additive or non-breaking. Your team's current success here indicates good minor upgrade processes.
* **Major Upgrades (Application Version Upgrades or Significant Platform Changes):** This refers to moving between major application versions (e.g., from an older version of Dynamics 365 Finance and Operations to a much newer one, or a significant change in the underlying Power Platform Dataverse). These can involve:
    * **Breaking API changes.**
    * **Schema changes in the underlying Dataverse/database.**
    * **Deprecation of features or components.**
    * **Changes in the solution packaging or deployment model.**
    * **Compatibility issues with third-party solutions or custom code.**
    * **Changes in required tooling or build environments.**

The issues your team faces with major upgrades stem from these inherent complexities.

## Standard Operating Procedure (SOP) for Dynamics 365 Major Dependency Upgrades with Azure DevOps

This SOP outlines a methodical approach, emphasizing preparation, testing, and meticulous dependency management.

### Phase 1: Planning and Discovery (Pre-Upgrade)

1.  **Understand the Target Version Changes:**
    * **Thoroughly review Microsoft's release notes:** This is paramount. Look for deprecated features, breaking changes, new functionalities that might impact existing customizations, and updated system requirements. Pay close attention to "What's new" and "Deprecations" for both the platform (Dataverse/Power Platform) and the specific Dynamics 365 applications (e.g., Sales, Finance, Supply Chain).
    * **Identify new dependencies:** Major upgrades often introduce new system components or framework dependencies. Understand what these are.
    * **Check compatibility matrix:** Verify the compatibility of your existing third-party solutions and integrations with the target Dynamics 365 version. Contact vendors early.

2.  **Inventory Current State:**
    * **Document all custom solutions:** List all managed and unmanaged solutions, their publishers, versions, and dependencies.
    * **Identify all third-party solutions:** Include their versions and the vendors.
    * **List all integrations:** Document how your Dynamics 365 instance integrates with other systems (e.g., Azure Functions, Logic Apps, external APIs, data warehouses).
    * **Review custom code (plugins, custom workflows, web resources, client-side scripts):** Identify any code that might rely on deprecated APIs or components.
    * **Assess current build and release pipelines:** Note the versions of Power Platform Build Tools, agent images, and any custom scripts used.

3.  **Establish a Dedicated Upgrade Environment:**
    * **Provision a clean, isolated Dynamics 365 environment:** This should be a full copy of your production environment, or as close as possible, but completely isolated. This is where the initial upgrade attempts and extensive testing will happen.
    * **Ensure the build agent/VM has necessary tooling:** For Dynamics 365 Finance & Operations, this might mean a specific build VM. For Customer Engagement (Sales, Service etc.) on Dataverse, ensure your Azure DevOps agents (Microsoft-hosted or self-hosted) have the latest Power Platform Build Tools installed.

4.  **Version Control Strategy:**
    * **Create a dedicated feature branch for the upgrade:** All upgrade-related changes (solution updates, code modifications, pipeline adjustments) should happen in this branch. This keeps your main development branch stable for ongoing minor work.
    * **Consider a "release branch" model:** If your team uses a release branch for production deployments, create a new release branch for the target D365 version once the upgrade is stable.

### Phase 2: Execution and Troubleshooting (Upgrade Cycle)

1.  **Upgrade the Development Environment (First Pass):**
    * **Attempt the upgrade on the isolated development environment.** This might involve:
        * Applying the new Dynamics 365 version update.
        * Importing your existing solutions.
        * For Finance & Operations, updating your build VM and potentially re-installing base packages.
    * **Address immediate import/application issues:** The environment might flag missing dependencies or compatibility problems.
    * **Document all errors and resolutions:** This will be invaluable for subsequent iterations and for future major upgrades.

2.  **Iterative Solution Export and Import (Key for Dependencies):**
    * **For Dataverse (Customer Engagement):**
        * **Export all unmanaged solutions (except default):** Start with solutions at the lowest dependency level.
        * **Attempt to import these solutions into the upgraded target environment.**
        * **Troubleshoot "Missing Dependency" errors:**
            * **Identify the missing component:** The error message will usually tell you what's missing (e.g., a specific entity, field, process, or a base application/solution).
            * **Determine the source:** Is it a missing Microsoft component (application, feature), a missing unmanaged customization, or a missing managed third-party solution?
            * **Resolve:**
                * **Microsoft components:** Ensure the target environment has all required base applications installed and updated to the correct version in the Power Platform Admin Center.
                * **Unmanaged customizations:** If a component is missing from your solution, add it to your solution in the source environment, export again, and re-import.
                * **Managed third-party solutions:** Import the *compatible version* of the third-party solution into the target environment *before* importing your dependent solutions.
        * **Repeat the process for solutions in a dependency order (lowest to highest).** This is critical. Solutions that depend on others must be imported *after* their dependencies.
        * **Focus on smaller, incremental imports:** Don't try to import everything at once after a major upgrade.

    * **For Finance & Operations (F&O):**
        * **Update your build environment/VM:** Ensure it's running the correct Visual Studio, development tools, and hotfixes compatible with the target F&O version.
        * **Update NuGet packages:** Your Azure DevOps pipeline will likely use NuGet packages for the compiler tools, platform X++, and application X++. These *must* be updated to match the target F&O version. This is often a significant source of "dependency" issues in F&O builds.
        * **Recompile all custom code (X++):** A major version jump almost guarantees the need for a full recompile.
        * **Run Customization Analysis Report (CAR):** This tool helps identify best practice violations and potential issues in your custom code that might break with the new version. Address warnings and errors.
        * **Generate new deployable packages:** Your Azure DevOps build pipeline should create deployable packages from the recompiled code.

3.  **Update Custom Code and Configurations:**
    * **Review and refactor deprecated APIs/features:** Replace deprecated calls with their new equivalents. Microsoft usually provides migration guides.
    * **Adjust web resources (JavaScript):** Browser compatibility or changes in D365 client-side APIs might require updates.
    * **Update custom plugins and workflows:** Recompile against the new SDK. Test thoroughly for behavioral changes.
    * **Update configuration data:** Some configuration entities or settings might change with major versions, requiring data adjustments or re-creation.

4.  **Update Azure DevOps Pipelines:**
    * **Power Platform Build Tools:** Ensure you are using the *latest version* of the Power Platform Build Tools extension in Azure DevOps. Microsoft constantly updates these.
    * **Agent Specification:** Verify your agent pools (Microsoft-hosted or self-hosted) are configured to use an agent image compatible with the new D365 version's build requirements (e.g., `windows-latest` for Dataverse, or a specific F&O build VM image).
    * **Pipeline Tasks:** Review each task in your build and release pipelines.
        * **Power Platform CLI (PAC CLI):** For Dataverse, migrating from older PowerShell-based tools to PAC CLI is highly recommended as it's more robust and actively developed.
        * **F&O Specific Tasks:** Ensure your F&O build pipeline tasks (NuGet installation, build solution, deployable package creation) are configured for the new F&O version's NuGet package paths and compiler versions.
    * **Variable Updates:** Any version numbers or paths stored as pipeline variables will need to be updated.

5.  **Extensive Testing:**
    * **Unit Tests:** Re-run all existing unit tests for custom code.
    * **Integration Tests:** Thoroughly test all integrations with external systems.
    * **Functional Testing:** Perform comprehensive functional testing of all core business processes, custom features, and third-party solutions.
    * **Regression Testing:** Ensure existing functionalities were not broken by the upgrade.
    * **User Acceptance Testing (UAT):** Involve key business users early to validate critical processes.
    * **Performance Testing:** Assess if the upgrade introduced any performance degradations.

### Phase 3: Deployment and Post-Upgrade

1.  **Production Deployment Strategy:**
    * **Plan the cutover:** Define a detailed plan for deploying the upgraded solutions to production, including downtime considerations.
    * **Backup everything:** Full backups of your production Dynamics 365 environment and any associated databases.
    * **Staged Rollout (if feasible):** For critical systems, consider deploying to a subset of users or regions first.

2.  **Monitoring and Rollback Plan:**
    * **Monitor closely post-deployment:** Use Azure Monitor, Application Insights, and Dynamics 365 diagnostics to identify any issues immediately.
    * **Define clear rollback procedures:** What is the plan if the production upgrade fails catastrophically? This typically involves restoring from a backup.

## Specific Methods and Best Practices

* **Managed vs. Unmanaged Solutions:**
    * **Prefer Managed Solutions for deployment to higher environments.** This provides better ALM capabilities and dependency tracking.
    * **Avoid unmanaged layers in production.** Unmanaged solutions can introduce implicit dependencies and make upgrades harder. Aim to develop in unmanaged, then package as managed for deployment.
* **Small, Incremental Solution Structure:**
    * Instead of one monolithic solution, break down your customizations into smaller, logical, and dependent solutions. This makes dependency management easier during upgrades, as you can upgrade smaller pieces.
    * Identify "base" solutions that other solutions depend on and upgrade those first.
* **Clean Up Unused Components:** Before a major upgrade, clean up any unused fields, entities, processes, or other components from your solutions. This reduces complexity and potential dependency issues.
* **Dependency Walker Tools:** When facing missing dependency errors, leverage tools provided by Microsoft (e.g., solution dependency viewer in Power Apps portal or looking at `solution.xml` directly if exporting unmanaged) to understand the full dependency chain.
* **Microsoft Lifecycle Services (LCS) for F&O:** For Dynamics 365 Finance and Operations, LCS is central to managing environments, applying updates, and downloading deployable packages and NuGet packages. Ensure your Azure DevOps pipelines are integrated correctly with LCS.
* **Power Platform Admin Center (PPAC):** For Dataverse, the PPAC is where you manage environments, install base applications, and apply updates. Regularly check for available updates for core D365 apps (Sales, Service, etc.) and apply them before your custom solution upgrades.
* **Personal Access Tokens (PATs) and Service Connections:** Ensure your Azure DevOps service connections used for connecting to Dynamics 365 environments have the correct permissions and are kept up-to-date. Using Service Principals with Workload Identity Federation is the recommended secure approach.
* **Automated Testing:** Invest heavily in automated unit, integration, and UI tests. These are your safety net during major upgrades. They allow you to quickly identify if new versions have broken existing functionality.
* **Dedicated Build Agents (for F&O):** For F&O, consider self-hosted build agents (VMs) that precisely match your development environment configuration. This avoids potential discrepancies with Microsoft-hosted agents during major compiler/platform updates.
* **"Go-Live" Checklist:** Create a detailed checklist for pre-upgrade, during-upgrade, and post-upgrade steps, including all manual checks and automated pipeline runs.
* **Communication:** Maintain clear and consistent communication with stakeholders, developers, and testers throughout the upgrade process.

By adopting this structured SOP and incorporating these best practices, your team can significantly reduce the pain points associated with major Dynamics 365 dependency upgrades, making them more predictable and successful within your Azure DevOps pipeline.