# query : Windows Server 2019 was used to run some pipelines, as well as being used in the agent pool. We have resolved this now and have updated all the agent pools to Windows latest. We have also tested the pipelines and can confirm that they are working, but what we wanted to ask was if there was any tips or advice you could give around ensuring that the system will continue to work by upgrading tool versions etc.

Based on the discussion and understanding we can consider two different option centric to agent.

## option1 : using microsoft hosted agents

1. periodical check for microsoft hosted agent depriciation schedule and check for installed tools
official link : https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=windows-images%2Cyaml
https://github.com/actions/runner-images/tree/main/images/windows

2. Use fixed version for critical tools

3. maintain tool installation stage containing all require tools installation task ( parameterized ) prefarabely templated  along with build and deployment stage.

4. maintain parameterized test pipeline to check deployment on lab environment after major tool version upgrade / new tools addition

5. create self-hosted agent options as fall back

## option2 : Using self hosted agent

1. while ms hosted agent compute power is limited to ** , self hosted agent can be provisioned based on the requirements even VM scaleset can be used for high availability and more parallel jobs to run .

2. agent provisioning and tool installation can be fully automated, parameterized and plugged into pipeline , templatized  before running build and deployment pipeline.

# --------------------------------- polished ------------------------------------


Here is the polished and professional version of your content:

---

**# Query:**
*Windows Server 2019 was previously used to run some pipelines and as part of the agent pool. This has now been resolved, and all agent pools have been updated to use `windows-latest`. Pipelines have been tested and are functioning correctly. We would like to ask if there are any recommendations or best practices to ensure long-term system reliability, particularly with regard to tool version upgrades and future-proofing.*

---

Based on the discussion and current understanding, we can consider two options related to agent strategy:

---

## ✅ Option 1: Using Microsoft-Hosted Agents

1. **Monitor Deprecation and Image Changes Regularly**
   Periodically review Microsoft's hosted agent deprecation schedule and toolset updates to avoid unexpected failures.

   * [Azure DevOps Microsoft-hosted Agents Documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=windows-images%2Cyaml)
   * [GitHub Actions Windows Image Definitions](https://github.com/actions/runner-images/tree/main/images/windows)

2. **Pin Fixed Versions for Critical Tools**
   Avoid reliance on default versions by explicitly specifying versions for tools like .NET SDK, Node.js, PowerShell modules, etc.

3. **Maintain a Tool Installation Stage**
   Add a dedicated, **parameterized and templated stage** for installing required tools within the pipeline. This ensures repeatability and control across environments.

4. **Implement a Test Validation Pipeline**
   Use a separate **parameterized pipeline** to deploy to a lab/test environment after any major tool upgrade or new tool addition. This provides early feedback on compatibility issues.

5. **Maintain a Self-Hosted Agent Strategy as a Fallback**
   Prepare a ready-to-deploy self-hosted agent setup to mitigate risks in case hosted agents are deprecated or unavailable.

---

## ✅ Option 2: Using Self-Hosted Agents

1. **Enhanced Scalability and Performance**
   Microsoft-hosted agents have limitations on compute power and concurrency. With self-hosted agents, you can provision resources based on project needs. For high availability and scalability, consider using **Azure Virtual Machine Scale Sets** (VMSS) as agent pools.

2. **Automated Agent Provisioning and Tool Installation**
   The provisioning of agents and installation of necessary tools can be **fully automated, parameterized, and templated**, and integrated into the pipeline. This ensures consistent agent setup before triggering build and deployment stages.

---

Let me know if you'd like this in Markdown format for documentation or as a diagram to present to your team.


Here's the polished version in **Markdown format**, ideal for internal documentation or sharing with your DevOps team:

---

```markdown
# Agent Strategy for Pipeline Reliability and Future-Proofing

## 📌 Context

Windows Server 2019 was previously used in some pipelines and within the agent pool. This setup has now been migrated to `windows-latest`, and pipelines have been successfully tested. To ensure long-term reliability, especially with regard to tooling updates, two strategic options can be considered.

---

## ✅ Option 1: Using Microsoft-Hosted Agents

1. **Monitor Deprecation and Image Updates**
   - Periodically check Microsoft’s hosted agent deprecation schedule and toolset updates.
   - Reference links:
     - [Microsoft-hosted Agents Documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=windows-images%2Cyaml)
     - [GitHub Actions Runner Images - Windows](https://github.com/actions/runner-images/tree/main/images/windows)

2. **Pin Fixed Versions for Critical Tools**
   - Explicitly define versions for .NET SDKs, Node.js, PowerShell modules, etc., instead of relying on default agent image versions.

3. **Tool Installation Stage**
   - Create a **dedicated, parameterized, and templated** pipeline stage for installing required tools before build and deployment steps.
   - This adds control and consistency across pipelines.

4. **Validation Pipeline for Tool Changes**
   - Set up a **parameterized test pipeline** that targets a lab/test environment.
   - Use it to verify pipeline behavior after tool version upgrades or additions.

5. **Fallback to Self-Hosted Agent**
   - Maintain infrastructure-as-code scripts to provision self-hosted agents quickly in case Microsoft-hosted agents become incompatible or deprecated.

---

## ✅ Option 2: Using Self-Hosted Agents

1. **Improved Scalability and Control**
   - Microsoft-hosted agents offer limited compute power and concurrency.
   - Self-hosted agents can be sized and scaled (e.g., via **VM Scale Sets**) to meet project-specific workloads and allow parallel builds.

2. **Automated Provisioning and Configuration**
   - Use **Azure Bicep/Terraform** and **PowerShell/DSC** to fully automate:
     - VM creation
     - Network setup (including NSGs and Bastion access)
     - Agent configuration and registration to Azure DevOps
     - Tool installation
   - Integrate these automation steps into your pipeline as pre-deployment stages.

---

## 🔐 Additional Recommendations

- Use **Azure Key Vault** and **Managed Identity** for storing and retrieving secrets (e.g., PATs for agent registration).
- Monitor [agent image release notes](https://github.com/actions/runner-images/releases) regularly.
- Implement monitoring using **Log Analytics** or **Azure Monitor** to track agent health and service status.
- Consider using **Private Link or Service Endpoints** for private deployments (e.g., App Service, ACR).

---

## 📎 Summary Diagram (Optional for Draw.io or Visio)

```

\[Developer] --> \[Pipeline Trigger]
|
v
+------------------------+
\| Microsoft-Hosted Agent | <---- Option 1
+------------------------+
|
v
\[Tool Install Stage] ---------+
\|                    |
v                    |
\[Build & Deploy]            |
|
+------------------------+     |
\| Self-Hosted Agent Pool | <---- Option 2 (fallback or primary)
+------------------------+
|
\[Bastion Host Access]
|
\[Internal VNet Security]

```

---

Let me know if you’d like this converted into a **Word document, checklist, or visualized in Draw.io** format.
```

