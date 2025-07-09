Here is a detailed report in table format listing all required access levels at the Azure DevOps project level for a DevOps Engineer to perform full operations across Azure Boards, Repos, Pipelines, and to manage service connections and install extensions like Power Platform Tools, SonarCloud, Kubernetes plugin.


---

🔒 Azure DevOps Project-Level Access Requirements for DevOps Engineer

Area	Access Level	Type of Access	Work/Task	Rationale / Why Required

Project Access	Project Administrator	Role / Security Group	Full control over all project resources	Required to manage repos, pipelines, boards, service connections, extensions, etc.
Azure Boards	Basic or Stakeholder (min.)	User Access Level	View/create/edit work items, assign tasks, link commits	Basic access required for full collaboration and automation setup
	Edit work items in this node	Permission	Create/edit work items	Required for automation or scripting updates in work items
	Manage permissions	Permission	Set security roles for Boards	Needed to organize project workflows and permissions
Azure Repos	Contribute	Permission	Push/pull/merge code, create branches, PRs	Essential for code management and CI setup
	Create branch	Permission	Enable branch-based workflows	CI/CD branching strategy depends on this
	Manage permissions	Permission	Manage repo-level security	Needed to protect branches and set PR policies
	Force push	Permission	Needed for automation (use with caution)	Required for special scripted operations
Azure Pipelines	Create/Edit build definitions	Permission	Create and manage build pipelines	Core for CI/CD setup and maintenance
	Queue builds	Permission	Trigger pipelines manually or via automation	Required for testing and running builds manually
	Manage build queue	Permission	Manage build agents and queues	Required for agent pool configurations
	Administer build permissions	Permission	Set security for pipeline folders and jobs	Needed to secure pipeline environments
Service Connections	Manage service connections	Permission	Create/edit/delete service connections (ARM, Docker, etc.)	Required to link Azure, ACR, AKS, D365, etc., for deployments
Extensions	Install Marketplace Extensions	Project Collection Access (via Org-level permission)	Install Power Platform Tool, SonarCloud, Kubernetes, etc.	Required to enable integrations and plugins essential to DevOps tasks
	Project Collection Administrator	Organization-level Role	Install extensions from Visual Studio Marketplace	Must be a Collection Admin to install org-wide extensions
Variable Groups / Libraries	Manage Library	Permission	Add/edit variable groups, secrets, certificates	Needed for secure and dynamic deployment configuration
Environment Permissions	Administer release environment	Permission	Set security for environments (dev, QA, prod)	Required to deploy securely and manage approvals
Agent Pools	Use & Administer Agent Pools	Permission	Use hosted/self-hosted agents	Required to run build/release pipelines
YAML Pipelines	Edit YAML	Permission	Modify infrastructure-as-code pipeline definitions	Core task for modern DevOps workflows
Approvals and Checks	Manage approvals and checks	Permission	Configure manual approvals, gates, branch policies	Required for secure promotion between environments



---

🔧 Summary: Minimum Roles Needed

Project-Level Role: Project Administrator

Org-Level Role: Project Collection Administrator (for installing extensions)

User Access Level: Basic or Visual Studio Subscriber (Basic + Test Plans) (depending on board usage)

Permissions: Granular permissions listed above must be set under Project Settings → Permissions and Security → Repos / Pipelines / Boards.



---

📌 Notes:

Installing extensions like Power Platform Tools, SonarCloud, or Kubernetes plugins requires organization-level permission (Project Collection Administrator) — this cannot be granted only at the project level.

If DevOps Engineer does not have org-level permission, a separate admin user must install the extension, and then the engineer can use it at the project level.

Service Connections, Variable Groups, and Environments often need manual approval flows, so permissions must be customized per environment/pipeline.



---

Would you like this table exported to Excel, Word, or a PDF report?

