## Critical Pre-Migration Clarification Questions  

### **I. Azure DevOps Environment**  
1. **Source & Target Versions**:  
   - Exact version of current Azure DevOps Server (e.g., 2019 RTW, 2020.1)?  
   - Target: Azure DevOps Services (cloud) or new on-prem server (specify version)?  

2. **Migration Scope**:  
   - Migrating entire project collection or selected projects?  
   - Should historical data (build/release logs, work items) be preserved?  
   - Are test plans, artifacts, and variable groups included?  

3. **Tool Constraints**:  
   - If using Azure DevOps Migration Tool:  
     - Is source server version within [supported versions](https://learn.microsoft.com/en-us/azure/devops/migrate/migration-tool-supported-versions?view=azure-devops)?  
   - If using third-party tools (e.g., OpsHub):  
     - Are there compliance/security restrictions?  

4. **Extensions & Integrations**:  
   - List all installed extensions (e.g., Power Platform Build Tools, SonarQube).  
   - Are there legacy XAML builds or deprecated tasks requiring replacement?  

5. **Permissions & Security**:  
   - How should service connections (Power Platform SPNs) be recreated?  
   - Should Azure AD groups/permissions be replicated or redesigned?  

---

### **II. Power Pages Configuration**  
6. **Source Environment**:  
   - Current Power Pages version and solution type (managed/unmanaged)?  
   - Are there patched solutions? (Patched solutions block standard exports.)  

7. **Customizations**:  
   - List all custom code (Liquid templates, JavaScript, CSS) and its storage location.  
   - Are there portal web files not in source control?  

8. **Data Migration**:  
   - Should portal content (e.g., web pages, blogs) be migrated?  
   - How will authentication providers (AAD/B2C) be reconfigured?  

9. **Target Environment**:  
   - Is a new Power Platform environment provisioned?  
   - Should the portal retain the same URL? (Requires DNS changes.)  

---

### **III. Pipeline Architecture**  
10. **Agent Infrastructure**:  
    - Current agent OS (Windows/Linux) and version?  
    - Will you use Microsoft-hosted agents or upgrade self-hosted agents?  

11. **Pipeline Definition**:  
    - Are pipelines defined in YAML or classic UI?  
    - Should pipelines be restructured (e.g., multi-stage YAML, environments)?  

12. **Secrets Management**:  
    - Where are secrets stored (Azure Key Vault, variable groups)?  
    - Who will rotate SPN credentials post-migration?  

13. **Testing Requirements**:  
    - Are automated tests (e.g., Pester, Selenium) integrated?  
    - What validation steps are required post-deployment?  

---

### **IV. Operational Risks**  
14. **Downtime Tolerance**:  
    - Maximum allowed downtime for portal during migration?  
    - Should a blue-green deployment strategy be used?  

15. **Rollback Plan**:  
    - Fallback procedure if migration fails (e.g., restore DB snapshots)?  
    - How quickly can the old environment be reactivated?  

16. **Compliance & Licensing**:  
    - Are there regulatory constraints (GDPR, HIPAA) for data migration?  
    - Confirm Power Pages user licensing in the target environment.  

---

### **V. Timeline & Resources**  
17. **Execution Window**:  
    - Approved maintenance window dates/times?  
18. **Stakeholder Alignment**:  
    - Who approves go/no-go decisions at each phase?  
    - Who validates post-migration functionality (devs, admins, business users)?  

> **Next Step**: Schedule a workshop with Azure DevOps admins, Power Platform architects, and security team to resolve these. Document decisions in a migration runbook.