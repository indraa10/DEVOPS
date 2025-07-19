# create a azure pipeline in yaml file to sync crm-2016 branch to remediation branch , check for below mentioned constrain / rules

# 1. branch policy is enforced for remediation branch where direct commit is not possible , again 2 separate person review is required .

To **automatically sync changes from `crm-2016` to `remediation` branch** in Azure Repos using a pipeline, while respecting the **branch policy** on `remediation` (i.e., **no direct commits** and **2 reviewers required**), you must **create a Pull Request (PR) from the pipeline**, rather than directly pushing.

Here’s a YAML pipeline that will:

* Checkout `crm-2016` branch.
* Create a new sync branch from `crm-2016`.
* Push the sync branch.
* Create a PR from the sync branch to `remediation`.

---

### ✅ Azure Pipeline YAML to Sync and Create PR

```yaml
trigger:
  branches:
    include:
      - crm-2016

pool:
  vmImage: 'ubuntu-latest'

variables:
  SOURCE_BRANCH: 'crm-2016'
  TARGET_BRANCH: 'remediation'
  SYNC_BRANCH: 'sync/crm-to-remediation-$(Build.BuildId)'

steps:
- checkout: self
  persistCredentials: true

- script: |
    git config user.email "buildagent@yourdomain.com"
    git config user.name "Azure DevOps Agent"

    echo "Fetching all branches..."
    git fetch origin

    echo "Creating new sync branch from $SOURCE_BRANCH"
    git checkout origin/$(SOURCE_BRANCH) -b $(SYNC_BRANCH)

    echo "Pushing new sync branch to remote"
    git push origin $(SYNC_BRANCH)
  displayName: 'Create and Push Sync Branch'

- task: AzureCLI@2
  displayName: 'Create PR to Remediation Branch'
  inputs:
    azureSubscription: 'Your-Azure-Service-Connection'
    scriptType: bash
    scriptLocation: inlineScript
    inlineScript: |
      ORG_URL="https://dev.azure.com/YOUR_ORG"
      PROJECT="YOUR_PROJECT"
      REPO="YOUR_REPO"
      SOURCE_BRANCH="$(SYNC_BRANCH)"
      TARGET_BRANCH="$(TARGET_BRANCH)"
      TITLE="Automated Sync from $SOURCE_BRANCH to $TARGET_BRANCH"
      DESCRIPTION="This PR was automatically created by the sync pipeline"

      echo "Getting personal access token from environment variable..."
      PAT=$(System.AccessToken)

      echo "Creating Pull Request..."
      RESPONSE=$(curl -s -X POST \
        -H "Authorization: Bearer $PAT" \
        -H "Content-Type: application/json" \
        "$ORG_URL/$PROJECT/_apis/git/repositories/$REPO/pullrequests?api-version=7.0" \
        -d "{
              \"sourceRefName\": \"refs/heads/$SOURCE_BRANCH\",
              \"targetRefName\": \"refs/heads/$TARGET_BRANCH\",
              \"title\": \"$TITLE\",
              \"description\": \"$DESCRIPTION\",
              \"reviewers\": []
            }")

      echo $RESPONSE
    env:
      SYSTEM_ACCESSTOKEN: $(System.AccessToken)
```

---

### 🔒 Key Points

| Constraint                        | Handled By                                |
| --------------------------------- | ----------------------------------------- |
| No direct commit to `remediation` | PR is created instead of direct push      |
| Two reviewer policy               | PR requires manual approval               |
| Automation triggers on `crm-2016` | Pipeline is triggered on push to crm-2016 |
| Sync without merge conflicts      | Branch created fresh from source          |

---

### 🔧 Prerequisites

1. **Service Connection** (`Your-Azure-Service-Connection`) must have access to Azure DevOps REST API with required permissions.
2. The pipeline must run with `Allow scripts to access the OAuth token` enabled (`System.AccessToken`).

   * Set in **Project Settings > Pipelines > Settings**
3. Replace the placeholders:

   * `YOUR_ORG`, `YOUR_PROJECT`, `YOUR_REPO`
   * You can also dynamically get the repository name with `Build.Repository.Name` if you prefer.

---

Would you like a version that only runs on demand (manual trigger) instead of every push to `crm-2016`?


# what should be the git config 

 Great question. The `git config` username and email used in an Azure Pipeline are important for traceability and compliance.

### ✅ Recommended `git config` values for Azure Pipelines

```bash
git config user.name "Azure DevOps Pipeline"
git config user.email "buildagent@yourdomain.com"
```

---

### 🔍 Explanation:

| Field        | Recommended Value           | Why?                                                                 |
| ------------ | --------------------------- | -------------------------------------------------------------------- |
| `user.name`  | `Azure DevOps Pipeline`     | Shows commits/branches were created by automation                    |
| `user.email` | `buildagent@yourdomain.com` | A generic email for automation (can be fake if only used internally) |

---

### 💡 Best Practices

* Use a **dedicated identity** (like `buildagent@yourcompany.com`) to distinguish bot commits from human commits.
* If your organization uses Microsoft Entra ID (AAD), you can create a service account for better visibility.
* Avoid using a personal email (e.g., a team member’s email) to prevent confusion about the source of changes.

---

### 🛠️ Example in Pipeline

```yaml
- script: |
    git config user.name "Azure DevOps Pipeline"
    git config user.email "buildagent@yourdomain.com"
```

Would you like the pipeline to **auto-assign reviewers** or **auto-complete** the PR if all checks pass?


# create two separate azure pipeline one with linux agent and another with windows agent for syncing crm-2016 branch to remediation branch  in azure repo where pipeline should auto merge all the files and folders except Script/Javascript path and for Script/Javascript the same pipeline raise PR .

remediation branch is alreday enforced with policy to restrict direct commit and require 2 reviewer . pipeline should bypass that . it will also generate report what all files are changed . 

list what all integration is require and how to integrate in azure devops.

# chatgpt : 

Here's a complete solution for your requirements:

---

## ✅ **Scenario Recap**

* **Source Branch**: `crm-2016`
* **Target Branch**: `remediation`
* **Automation Tasks**:

  1. Auto-merge all files/folders **except** `Script/Javascript` directly to `remediation` (bypassing branch policy).
  2. For `Script/Javascript`, create a **Pull Request** for manual review.
  3. Generate a **report of changed files**.
  4. Implement two pipelines:

     * One using a **Linux agent**
     * One using a **Windows agent**

---

## 🧩 Required Integrations

| Integration                          | Purpose                 | How to Set Up                                                                             |
| ------------------------------------ | ----------------------- | ----------------------------------------------------------------------------------------- |
| **System.AccessToken**               | Auth to REST API / repo | Enable `Allow scripts to access the OAuth token` in pipeline settings                     |
| **Project Collection Build Service** | Bypass branch policy    | Grant this identity **Contribute** + **Bypass policies** in `remediation` branch security |
| **Azure DevOps REST API**            | Create PR from pipeline | Use `curl` or `Invoke-RestMethod` with `System.AccessToken`                               |

---

## 🔐 Grant Bypass Permission to Pipeline

1. Go to **Repos > Branches > remediation > Branch Security**
2. Add: `Project Collection Build Service (YOUR_PROJECT)`
3. Set:

   * **Contribute**: ✅ Allow
   * **Bypass policies when pushing**: ✅ Allow

---

## 🧪 Pipeline Logic Overview

### Flow:

1. Fetch `crm-2016` and `remediation`.
2. Identify changes via `git diff`.
3. Separate changed paths:

   * Files **not in** `Script/Javascript`: push directly to `remediation`
   * Files **in** `Script/Javascript`: create a sync branch and raise PR
4. Generate a report of changed files

---

## 🐧 Linux Agent YAML Pipeline

```yaml
# azure-pipelines-linux.yml
trigger:
  branches:
    include:
      - crm-2016

pool:
  vmImage: 'ubuntu-latest'

variables:
  SOURCE_BRANCH: 'crm-2016'
  TARGET_BRANCH: 'remediation'
  SYNC_BRANCH: 'sync/js-only-$(Build.BuildId)'

steps:
- checkout: self
  persistCredentials: true

- script: |
    git config user.name "Azure DevOps Pipeline"
    git config user.email "buildagent@yourdomain.com"

    git remote set-url origin https://$(System.AccessToken)@dev.azure.com/$(System.CollectionUri)/$(System.TeamProject)/_git/$(Build.Repository.Name)

    git fetch origin
    git checkout origin/$(TARGET_BRANCH) -b $(TARGET_BRANCH)
    git checkout origin/$(SOURCE_BRANCH) -b $(SOURCE_BRANCH)

    echo "Finding changed files..."
    git diff --name-only origin/$(TARGET_BRANCH)..origin/$(SOURCE_BRANCH) > changed_files.txt
    cat changed_files.txt

    mkdir non_js_files
    mkdir js_files

    while read file; do
      if [[ "$file" == Script/Javascript/* ]]; then
        echo "$file -> PR path"
        cp --parents "$file" js_files/ 2>/dev/null || true
      else
        echo "$file -> Direct push path"
        cp --parents "$file" non_js_files/ 2>/dev/null || true
      fi
    done < changed_files.txt

    echo "Merging non-JS changes directly..."
    git checkout $(TARGET_BRANCH)
    cp -r non_js_files/* .
    git add .
    git commit -m "Auto-sync non-JS files from crm-2016"
    git push origin HEAD:$(TARGET_BRANCH)

    echo "Creating sync branch for JS changes..."
    git checkout -b $(SYNC_BRANCH)
    cp -r js_files/* .
    git add .
    git commit -m "Sync JS changes from crm-2016"
    git push origin $(SYNC_BRANCH)
  displayName: 'Sync Non-JS & Push JS to Sync Branch'

- task: AzureCLI@2
  displayName: 'Create PR for JS Changes'
  inputs:
    azureSubscription: 'YOUR_SERVICE_CONN'  # Optional if System.AccessToken is enough
    scriptType: bash
    scriptLocation: inlineScript
    inlineScript: |
      ORG="https://dev.azure.com/YOUR_ORG"
      PROJECT="YOUR_PROJECT"
      REPO="YOUR_REPO"
      TITLE="PR: JS Sync from crm-2016"
      DESC="Only Script/Javascript files were modified"

      curl -X POST "$ORG/$PROJECT/_apis/git/repositories/$REPO/pullrequests?api-version=7.1-preview.1" \
        -H "Authorization: Bearer $SYSTEM_ACCESSTOKEN" \
        -H "Content-Type: application/json" \
        -d "{
          \"sourceRefName\": \"refs/heads/$(SYNC_BRANCH)\",
          \"targetRefName\": \"refs/heads/$(TARGET_BRANCH)\",
          \"title\": \"$TITLE\",
          \"description\": \"$DESC\",
          \"reviewers\": []
        }"
    env:
      SYSTEM_ACCESSTOKEN: $(System.AccessToken)

- task: PublishBuildArtifacts@1
  displayName: 'Upload Changed File Report'
  inputs:
    PathtoPublish: 'changed_files.txt'
    ArtifactName: 'ChangedFiles'
```

---

## 🪟 Windows Agent YAML Pipeline

```yaml
# azure-pipelines-windows.yml
trigger:
  branches:
    include:
      - crm-2016

pool:
  vmImage: 'windows-latest'

variables:
  SOURCE_BRANCH: 'crm-2016'
  TARGET_BRANCH: 'remediation'
  SYNC_BRANCH: 'sync/js-only-$(Build.BuildId)'

steps:
- checkout: self
  persistCredentials: true

- powershell: |
    git config user.name "Azure DevOps Pipeline"
    git config user.email "buildagent@yourdomain.com"

    git remote set-url origin https://$(System.AccessToken)@dev.azure.com/$(System.CollectionUri)/$(System.TeamProject)/_git/$(Build.Repository.Name)

    git fetch origin
    git checkout origin/$env:TARGET_BRANCH -b $env:TARGET_BRANCH
    git checkout origin/$env:SOURCE_BRANCH -b $env:SOURCE_BRANCH

    git diff --name-only origin/$env:TARGET_BRANCH..origin/$env:SOURCE_BRANCH > changed_files.txt
    type changed_files.txt

    mkdir non_js_files
    mkdir js_files

    Get-Content changed_files.txt | ForEach-Object {
      if ($_ -like "Script/Javascript/*") {
        Copy-Item $_ -Destination js_files -Recurse -Force -ErrorAction SilentlyContinue
      } else {
        Copy-Item $_ -Destination non_js_files -Recurse -Force -ErrorAction SilentlyContinue
      }
    }

    git checkout $env:TARGET_BRANCH
    Copy-Item non_js_files\* -Recurse -Force
    git add .
    git commit -m "Auto-sync non-JS files from crm-2016"
    git push origin HEAD:$env:TARGET_BRANCH

    git checkout -b $env:SYNC_BRANCH
    Copy-Item js_files\* -Recurse -Force
    git add .
    git commit -m "Sync JS changes from crm-2016"
    git push origin $env:SYNC_BRANCH
  displayName: 'Sync and Push (Windows)'

- task: AzureCLI@2
  displayName: 'Create PR for JS Changes (Windows)'
  inputs:
    azureSubscription: 'YOUR_SERVICE_CONN'
    scriptType: ps
    scriptLocation: inlineScript
    inlineScript: |
      $org = "https://dev.azure.com/YOUR_ORG"
      $project = "YOUR_PROJECT"
      $repo = "YOUR_REPO"
      $title = "PR: JS Sync from crm-2016"
      $desc = "Only Script/Javascript files were modified"

      $body = @{
        sourceRefName = "refs/heads/$env:SYNC_BRANCH"
        targetRefName = "refs/heads/$env:TARGET_BRANCH"
        title = $title
        description = $desc
        reviewers = @()
      } | ConvertTo-Json -Depth 10

      Invoke-RestMethod -Uri "$org/$project/_apis/git/repositories/$repo/pullrequests?api-version=7.1-preview.1" `
        -Method Post `
        -Headers @{ Authorization = "Bearer $env:SYSTEM_ACCESSTOKEN"; "Content-Type" = "application/json" } `
        -Body $body
    env:
      SYSTEM_ACCESSTOKEN: $(System.AccessToken)

- task: PublishBuildArtifacts@1
  displayName: 'Upload Changed File Report'
  inputs:
    PathtoPublish: 'changed_files.txt'
    ArtifactName: 'ChangedFiles'
```

---

## ✅ Next Steps

1. Replace placeholders:

   * `YOUR_ORG`, `YOUR_PROJECT`, `YOUR_REPO`
   * Optionally hardcode `azureSubscription` or use `System.AccessToken` directly
2. Add both pipelines in Azure DevOps with separate YAML files
3. Ensure permissions are set correctly (bypass policy)
4. Trigger the pipelines manually or via push

---

Would you like a **Markdown PR template** to guide the reviewers as well?

# deepseek 

# Azure Pipelines for CRM-2016 to Remediation Branch Sync

## Overview

I'll create two separate Azure Pipelines (Linux and Windows agents) that will:
1. Auto-merge all files/folders except `Script/Javascript` from CRM-2016 to remediation branch
2. For `Script/Javascript` path, raise a PR instead of direct merge
3. Bypass branch policies requiring 2 reviewers
4. Generate a change report

## Required Integrations

1. **Azure Repos**: For source code management and branch operations
2. **Azure Pipelines**: For CI/CD automation
3. **REST API Access**: For PR creation and branch policy bypass
4. **PowerShell/Bash Scripting**: For file operations and reporting
5. **Service Connections**: For authentication
6. **Pipeline Variables/Secrets**: For secure storage of credentials

## Integration Setup

### 1. Create Service Principal for Bypassing Policies
- Go to Azure DevOps Organization Settings > Service Connections
- Create a new "Personal Access Token (PAT)" service connection
- Generate a PAT with full repo and build permissions
- Store this securely in pipeline variables

### 2. Configure Pipeline Permissions
- Go to Project Settings > Pipelines > Settings
- Enable "Allow scripts to access the OAuth token"
- Under Repository, grant "Contribute" and "Create branch" permissions to the build service identity

## Pipeline YAML Definitions

### Linux Agent Pipeline (`linux-sync-pipeline.yml`)

```yaml
trigger:
  branches:
    include:
    - CRM-2016
  paths:
    exclude:
    - Scripts/Javascript/*

pool:
  vmImage: 'ubuntu-latest'

variables:
  sourceBranch: 'CRM-2016'
  targetBranch: 'remediation'
  system.accessToken: $(System.AccessToken)

steps:
- checkout: self
  persistCredentials: true
  clean: true

- script: |
    # Install git and jq if not present
    sudo apt-get update && sudo apt-get install -y git jq
    
    # Configure git
    git config --global user.email "azure-pipelines@example.com"
    git config --global user.name "Azure Pipeline"
    
    # Fetch both branches
    git fetch origin $(sourceBranch)
    git fetch origin $(targetBranch)
    
    # Create temp branch
    git checkout -b temp-sync $(targetBranch)
    
    # Merge all except Scripts/Javascript
    git merge --no-ff --no-commit origin/$(sourceBranch)
    
    # Remove Javascript files if any were merged
    git rm --cached -r Scripts/Javascript/
    git reset -- Scripts/Javascript/
    git checkout origin/$(targetBranch) -- Scripts/Javascript/
    
    # Commit the merge
    git commit -m "Auto-merge from $(sourceBranch) to $(targetBranch) [skip ci]"
    
    # Get changed files list for report
    git diff --name-only $(targetBranch)..temp-sync > changed_files.txt
    
    # Push changes directly (bypassing policies)
    git push origin temp-sync:$(targetBranch) -f
    
    # Create PR for Javascript changes if they exist
    if [ -d "Scripts/Javascript" ]; then
      changes=$(git diff --name-only $(targetBranch)..origin/$(sourceBranch) -- Scripts/Javascript/)
      if [ ! -z "$changes" ]; then
        echo "##vso[task.setvariable variable=hasJsChanges]true"
        git diff $(targetBranch)..origin/$(sourceBranch) -- Scripts/Javascript/ > js_changes.diff
      fi
    fi
  displayName: 'Merge branches and prepare changes'
  
- task: PowerShell@2
  condition: eq(variables.hasJsChanges, 'true')
  inputs:
    targetType: 'inline'
    script: |
      $jsChanges = Get-Content "$(Build.SourcesDirectory)/js_changes.diff"
      $changedFiles = Get-Content "$(Build.SourcesDirectory)/changed_files.txt"
      
      # Create PR via REST API
      $url = "$($env:SYSTEM_TEAMFOUNDATIONCOLLECTIONURI)$($env:SYSTEM_TEAMPROJECTID)/_apis/git/repositories/$(Build.Repository.Name)/pullrequests?api-version=6.0"
      $body = @{
        sourceRefName = "refs/heads/$(sourceBranch)"
        targetRefName = "refs/heads/$(targetBranch)"
        title = "Javascript Changes from $(sourceBranch) to $(targetBranch)"
        description = "Automated PR for Javascript changes. Changed files:`n$($jsChanges -join "`n")"
        reviewers = @()
      } | ConvertTo-Json
      
      $headers = @{
        Authorization = "Bearer $(System.AccessToken)"
        "Content-Type" = "application/json"
      }
      
      try {
        $response = Invoke-RestMethod -Uri $url -Method Post -Headers $headers -Body $body
        Write-Host "PR created: $($response.url)"
      } catch {
        Write-Host "Failed to create PR: $($_.Exception.Message)"
        exit 1
      }
      
      # Generate report
      $report = @"
      Merge Report - $(Get-Date)
      ==========================
      Files Merged Directly:
      $($changedFiles -join "`n")
      
      Javascript Changes Requiring PR:
      $($jsChanges -join "`n")
      "@
      
      Set-Content -Path "$(Build.ArtifactStagingDirectory)/merge_report.txt" -Value $report
  displayName: 'Create PR for Javascript changes and generate report'

- task: PublishPipelineArtifact@1
  inputs:
    targetPath: '$(Build.ArtifactStagingDirectory)/merge_report.txt'
    artifact: 'MergeReport'
    publishLocation: 'pipeline'
```

### Windows Agent Pipeline (`windows-sync-pipeline.yml`)

```yaml
trigger:
  branches:
    include:
    - CRM-2016
  paths:
    exclude:
    - Scripts/Javascript/*

pool:
  vmImage: 'windows-latest'

variables:
  sourceBranch: 'CRM-2016'
  targetBranch: 'remediation'
  system.accessToken: $(System.AccessToken)

steps:
- checkout: self
  persistCredentials: true
  clean: true

- powershell: |
    # Configure git
    git config --global user.email "azure-pipelines@example.com"
    git config --global user.name "Azure Pipeline"
    
    # Fetch both branches
    git fetch origin $(sourceBranch)
    git fetch origin $(targetBranch)
    
    # Create temp branch
    git checkout -b temp-sync $(targetBranch)
    
    # Merge all except Scripts/Javascript
    git merge --no-ff --no-commit origin/$(sourceBranch)
    
    # Remove Javascript files if any were merged
    git rm --cached -r Scripts/Javascript/
    git reset -- Scripts/Javascript/
    git checkout origin/$(targetBranch) -- Scripts/Javascript/
    
    # Commit the merge
    git commit -m "Auto-merge from $(sourceBranch) to $(targetBranch) [skip ci]"
    
    # Get changed files list for report
    git diff --name-only $(targetBranch)..temp-sync > changed_files.txt
    
    # Push changes directly (bypassing policies)
    git push origin temp-sync:$(targetBranch) -f
    
    # Check for Javascript changes
    if (Test-Path "Scripts/Javascript") {
      $jsChanges = git diff --name-only $(targetBranch)..origin/$(sourceBranch) -- Scripts/Javascript/
      if ($jsChanges) {
        Write-Host "##vso[task.setvariable variable=hasJsChanges]true"
        git diff $(targetBranch)..origin/$(sourceBranch) -- Scripts/Javascript/ > js_changes.diff
      }
    }
  displayName: 'Merge branches and prepare changes'

- task: PowerShell@2
  condition: eq(variables.hasJsChanges, 'true')
  inputs:
    targetType: 'inline'
    script: |
      $jsChanges = Get-Content "$(Build.SourcesDirectory)/js_changes.diff"
      $changedFiles = Get-Content "$(Build.SourcesDirectory)/changed_files.txt"
      
      # Create PR via REST API
      $url = "$($env:SYSTEM_TEAMFOUNDATIONCOLLECTIONURI)$($env:SYSTEM_TEAMPROJECTID)/_apis/git/repositories/$(Build.Repository.Name)/pullrequests?api-version=6.0"
      $body = @{
        sourceRefName = "refs/heads/$(sourceBranch)"
        targetRefName = "refs/heads/$(targetBranch)"
        title = "Javascript Changes from $(sourceBranch) to $(targetBranch)"
        description = "Automated PR for Javascript changes. Changed files:`n$($jsChanges -join "`n")"
        reviewers = @()
      } | ConvertTo-Json
      
      $headers = @{
        Authorization = "Bearer $(System.AccessToken)"
        "Content-Type" = "application/json"
      }
      
      try {
        $response = Invoke-RestMethod -Uri $url -Method Post -Headers $headers -Body $body
        Write-Host "PR created: $($response.url)"
      } catch {
        Write-Host "Failed to create PR: $($_.Exception.Message)"
        exit 1
      }
      
      # Generate report
      $report = @"
      Merge Report - $(Get-Date)
      ==========================
      Files Merged Directly:
      $($changedFiles -join "`n")
      
      Javascript Changes Requiring PR:
      $($jsChanges -join "`n")
      "@
      
      Set-Content -Path "$(Build.ArtifactStagingDirectory)/merge_report.txt" -Value $report
  displayName: 'Create PR for Javascript changes and generate report'

- task: PublishPipelineArtifact@1
  inputs:
    targetPath: '$(Build.ArtifactStagingDirectory)/merge_report.txt'
    artifact: 'MergeReport'
    publishLocation: 'pipeline'
```

## Implementation Steps

1. **Create the Pipelines**:
   - Go to Azure DevOps > Pipelines > New Pipeline
   - Select your repository
   - Choose "Existing Azure Pipelines YAML file" and select the appropriate YAML file

2. **Configure Pipeline Permissions**:
   - For each pipeline, go to Pipeline Settings > Permissions
   - Grant "Bypass policies when pushing" permission to the build service identity

3. **Set Up Variables**:
   - Add any required variables (like authentication tokens) in Pipeline > Variables
   - Mark sensitive variables as "Secret"

4. **Test the Pipelines**:
   - Run the pipelines manually first to test
   - Verify the merge behavior and PR creation

5. **Monitor and Adjust**:
   - Review the merge reports
   - Adjust the paths or behavior as needed

## Notes

1. The pipelines use `System.AccessToken` which is automatically available and has the permissions of the build service identity.

2. The force push (`-f` flag) is used to bypass branch policies. Ensure your service identity has sufficient permissions.

3. The PR creation will include all Javascript changes in a single PR. Adjust if you need separate PRs per file.

4. The `[skip ci]` in commit messages prevents triggering new builds from these automated commits.

5. # grok

trigger:
  branches:
    include:
      - CRM-2016

pool:
  vmImage: 'ubuntu-latest'

steps:
- checkout: self
  persistCredentials: true

- task: Bash@3
  displayName: 'Sync CRM-2016 to remediation and handle PR'
  env:
    SYSTEM_ACCESSTOKEN: $(System.AccessToken)
  inputs:
    targetType: 'inline'
    script: |
      # Configure git
      git config --global user.email "pipeline@azuredevops.com"
      git config --global user.name "Azure Pipeline"

      # Fetch and checkout remediation branch
      git fetch origin
      git checkout remediation

      # Merge CRM-2016 into remediation
      git merge origin/CRM-2016 --no-ff -m "Auto-merge from CRM-2016 to remediation"

      # Get list of changed files
      git diff --name-only HEAD^ HEAD > changed_files.txt

      # Separate Script/Javascript files and non-Script files
      mkdir -p scripts non_scripts
      while IFS= read -r file; do
        if [[ "$file" == Scripts/* ]] || [[ "$file" == *.js ]] || [[ "$file" == *.ts ]]; then
          echo "$file" >> scripts/changed_scripts.txt
        else
          echo "$file" >> non_scripts/changed_non_scripts.txt
        fi
      done < changed_files.txt

      # Auto-merge non-Script/Javascript files
      if [ -s non_scripts/changed_non_scripts.txt ]; then
        git push origin remediation
      fi

      # Create PR for Script/Javascript files if they exist
      if [ -s scripts/changed_scripts.txt ]; then
        git checkout -b remediation-scripts-$(date +%Y%m%d%H%M%S)
        git add .
        git commit -m "Changes for Script/Javascript files"
        git push origin remediation-scripts-$(date +%Y%m%d%H%M%S)
        az repos pr create \
          --repository MyRepo \
          --source-branch remediation-scripts-$(date +%Y%m%d%H%M%S) \
          --target-branch remediation \
          --title "PR for Script/Javascript changes from CRM-2016" \
          --description "Auto-generated PR for Script/Javascript files" \
          --bypass-policy true \
          --bypass-policy-reason "Automated PR for Script/Javascript files"
      fi

      # Generate and publish report
      echo "Changed Files Report" > file_changes_report.txt
      echo "====================" >> file_changes_report.txt
      echo "Non-Script/Javascript Files (Auto-merged):" >> file_changes_report.txt
      cat non_scripts/changed_non_scripts.txt >> file_changes_report.txt 2>/dev/null || echo "None" >> file_changes_report.txt
      echo -e "\nScript/Javascript Files (PR Created):" >> file_changes_report.txt
      cat scripts/changed_scripts.txt >> file_changes_report.txt 2>/dev/null || echo "None" >> file_changes_report.txt

- task: PublishPipelineArtifact@1
  displayName: 'Publish File Changes Report'
  inputs:
    targetPath: 'file_changes_report.txt'
    artifact: 'FileChangesReport'
    publishLocation: 'pipeline'

# gemini

Of course. Here are two separate Azure Pipelines to sync changes from a `crm-2016` branch to a `remediation` branch with conditional merging and PR creation, along with the necessary integration steps.

The core logic is identical for both pipelines; they use cross-platform PowerShell, with the only difference being the agent pool (`vmImage`).

-----

### \#\# ⛓️ Required Integrations & Permissions

Before running the pipelines, you must grant the pipeline's identity the necessary permissions to bypass branch policies and create pull requests.

The pipeline runs under a special identity called the **Project Build Service**. Its name follows this format: `[Your Project Name] Build Service ([Your Organization Name])`.

#### **How to Configure Permissions:**

1.  Navigate to your **Project Settings** (bottom-left corner).
2.  Go to **Repositories** under the "Repos" section.
3.  Select the specific repository containing your branches.
4.  Switch to the **Branches** tab.
5.  Find the `remediation` branch, click the **three dots** (...) on the right, and select **Branch policies**.
6.  Scroll down to the **Permissions** section on the policies page.
7.  Click the **+** button or use the search box to find your build service account (e.g., `MyProject Build Service...`).
8.  Set the following permissions for that identity:
      * **Bypass policies when pushing**: Set to `Allow`. This is crucial for the auto-merge step.
      * **Contribute**: Set to `Allow`. This allows pushing new branches for the PR.
      * **Create pull request**: Set to `Allow`.

-----

### \#\# 🐧 Linux Agent Pipeline

This pipeline uses an `ubuntu-latest` agent. Create a new pipeline and point it to a file named `azure-pipelines-linux.yml` with the following content.

**File: `azure-pipelines-linux.yml`**

```yaml
# Pipeline for Linux Agent

trigger:
  branches:
    include:
      - crm-2016
pr: none

pool:
  vmImage: 'ubuntu-latest'

steps:
- checkout: self
  persistCredentials: true # Important: Allows the script to push back to the repo

- task: PowerShell@2
  name: SyncAndPullRequest
  displayName: 'Sync Branches, Auto-Merge, and Create PR'
  env:
    # System.AccessToken is a special variable that provides the pipeline's OAuth token
    AZURE_DEVOPS_EXT_PAT: $(System.AccessToken)
  inputs:
    targetType: 'inline'
    script: |
      # 1. Configure Git
      git config --global user.email "pipeline@devops.com"
      git config --global user.name "$(Build.DefinitionName) - $(Build.BuildId)"

      # 2. Identify all changed files between the branches
      $baseBranch = "origin/remediation"
      $sourceBranch = "origin/crm-2016"
      $diffs = git diff --name-only $baseBranch..$sourceBranch
      
      # Exit if there are no changes
      if (-not $diffs) {
        Write-Host "No changes detected between $sourceBranch and $baseBranch. Exiting."
        exit 0
      }
      
      # Save changes to a file for the report
      $diffs | Out-File -FilePath "$(Build.ArtifactStagingDirectory)/changed_files.txt"
      Write-Host "--- All Changed Files ---"
      cat "$(Build.ArtifactStagingDirectory)/changed_files.txt"
      Write-Host "-------------------------"

      # 3. Separate files for auto-merge and for PR
      $filesToAutoMerge = $diffs | Where-Object { $_ -notmatch "^Script/Javascript/" }
      $filesForPR = $diffs | Where-Object { $_ -match "^Script/Javascript/" }

      # 4. Process Auto-Merge Files (Direct Push)
      if ($filesToAutoMerge) {
        Write-Host "## Processing files for direct merge..."
        git checkout remediation
        git pull
        
        # Apply changes from the source branch for specific files
        $filesToAutoMerge | ForEach-Object { git checkout $sourceBranch -- $_ }
        
        git commit -m "Automated Sync: Merged non-script files from crm-2016 [skip ci]"
        git push origin HEAD:remediation
        Write-Host "Successfully pushed non-script changes to remediation branch."
      } else {
        Write-Host "No files found for direct merge."
      }
      
      # 5. Process Script Files (Create Pull Request)
      if ($filesForPR) {
        Write-Host "## Processing Script/Javascript files for PR..."
        $prBranch = "feature/sync-scripts-$(Build.BuildId)"
        
        # Ensure we start from the latest state of remediation branch
        git checkout remediation
        git pull
        
        git checkout -b $prBranch
        
        # Apply script changes from the source branch
        $filesForPR | ForEach-Object { git checkout $sourceBranch -- $_ }
        
        git commit -m "Feature: Sync Script/Javascript files from crm-2016 [skip ci]"
        git push origin $prBranch
        
        # Create the Pull Request using Azure DevOps CLI
        az extension add --name azure-devops
        az repos pr create --source-branch $prBranch `
                           --target-branch remediation `
                           --title "Automated PR: Sync Script/Javascript from crm-2016" `
                           --description "This PR was generated automatically to sync changes in the 'Script/Javascript' folder. Please review and approve." `
                           --organization "$(System.TeamFoundationCollectionUri)" `
                           --project "$(System.TeamProject)" `
                           --repository "$(Build.Repository.Name)"
        
        Write-Host "Successfully created Pull Request for script files."
      } else {
        Write-Host "No Script/Javascript files found to create a PR."
      }

- task: PublishBuildArtifacts@1
  displayName: 'Publish Change Report'
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'ChangeReport'
    publishLocation: 'Container'
```

-----

### \#\# 🪟 Windows Agent Pipeline

This pipeline uses a `windows-latest` agent. The PowerShell script is identical, demonstrating its cross-platform power.

**File: `azure-pipelines-windows.yml`**

```yaml
# Pipeline for Windows Agent

trigger:
  branches:
    include:
      - crm-2016
pr: none

pool:
  vmImage: 'windows-latest' # The only change is here

steps:
- checkout: self
  persistCredentials: true

- task: PowerShell@2
  name: SyncAndPullRequest
  displayName: 'Sync Branches, Auto-Merge, and Create PR'
  env:
    AZURE_DEVOPS_EXT_PAT: $(System.AccessToken)
  inputs:
    targetType: 'inline'
    script: |
      # 1. Configure Git
      git config --global user.email "pipeline@devops.com"
      git config --global user.name "$(Build.DefinitionName) - $(Build.BuildId)"

      # 2. Identify all changed files between the branches
      $baseBranch = "origin/remediation"
      $sourceBranch = "origin/crm-2016"
      $diffs = git diff --name-only $baseBranch..$sourceBranch
      
      if (-not $diffs) {
        Write-Host "No changes detected between $sourceBranch and $baseBranch. Exiting."
        exit 0
      }
      
      $diffs | Out-File -FilePath "$(Build.ArtifactStagingDirectory)/changed_files.txt"
      Write-Host "--- All Changed Files ---"
      cat "$(Build.ArtifactStagingDirectory)/changed_files.txt"
      Write-Host "-------------------------"

      # 3. Separate files for auto-merge and for PR
      $filesToAutoMerge = $diffs | Where-Object { $_ -notmatch "^Script/Javascript/" }
      $filesForPR = $diffs | Where-Object { $_ -match "^Script/Javascript/" }

      # 4. Process Auto-Merge Files (Direct Push)
      if ($filesToAutoMerge) {
        Write-Host "## Processing files for direct merge..."
        git checkout remediation
        git pull
        $filesToAutoMerge | ForEach-Object { git checkout $sourceBranch -- $_ }
        git commit -m "Automated Sync: Merged non-script files from crm-2016 [skip ci]"
        git push origin HEAD:remediation
        Write-Host "Successfully pushed non-script changes to remediation branch."
      } else {
        Write-Host "No files found for direct merge."
      }
      
      # 5. Process Script Files (Create Pull Request)
      if ($filesForPR) {
        Write-Host "## Processing Script/Javascript files for PR..."
        $prBranch = "feature/sync-scripts-$(Build.BuildId)"
        git checkout remediation
        git pull
        git checkout -b $prBranch
        $filesForPR | ForEach-Object { git checkout $sourceBranch -- $_ }
        git commit -m "Feature: Sync Script/Javascript files from crm-2016 [skip ci]"
        git push origin $prBranch
        
        az extension add --name azure-devops
        az repos pr create --source-branch $prBranch `
                           --target-branch remediation `
                           --title "Automated PR: Sync Script/Javascript from crm-2016" `
                           --description "This PR was generated automatically to sync changes in the 'Script/Javascript' folder. Please review and approve." `
                           --organization "$(System.TeamFoundationCollectionUri)" `
                           --project "$(System.TeamProject)" `
                           --repository "$(Build.Repository.Name)"
        
        Write-Host "Successfully created Pull Request for script files."
      } else {
        Write-Host "No Script/Javascript files found to create a PR."
      }

- task: PublishBuildArtifacts@1
  displayName: 'Publish Change Report'
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'ChangeReport'
    publishLocation: 'Container'
```

-----

### \#\# ⚙️ How It Works & Report Generation

  * **Trigger**: The pipeline automatically runs whenever a commit is pushed to the `crm-2016` branch.
  * **File Differentiation**: The PowerShell script uses `git diff` to find all changed files. It then uses a regular expression (`-match "^Script/Javascript/"`) to split the list of files into two groups.
  * **Auto-Merge**: For files *not* in `Script/Javascript`, the pipeline checks out the `remediation` branch, applies only those specific file changes, commits, and pushes directly. This push bypasses the branch policy because of the permissions you set.
  * **Pull Request**: For files inside `Script/Javascript`, the pipeline creates a new feature branch (e.g., `feature/sync-scripts-123`), pushes the script changes there, and then uses the Azure DevOps CLI (`az repos pr create`) to open a pull request. This PR will be subject to your normal review policies.
  * **Report**: A file named `changed_files.txt` is generated with a list of all detected changes. It is published as a pipeline artifact called **ChangeReport**, which you can download from the pipeline run summary page.
