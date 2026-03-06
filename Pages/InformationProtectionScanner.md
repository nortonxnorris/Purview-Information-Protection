# Microsoft Purview Information Protection Scanner Guide

The Microsoft Purview Content Explorer (formerly M365 Compliance) is an exceptional tool for identifying data within your M365 environment. It is a crucial first step in the data-classification journey, often referred to as “know your data.”

To locate and classify data stored on-premises, install and configure the Information Protection (AIP) scanner. The scanner runs as a Windows Server service, logs progress to SQL, and produces reports that highlight data matches.

---

## Prerequisites

### Scanner server

- Windows Server that meets the [official requirements](https://learn.microsoft.com/en-us/microsoft-365/compliance/deploy-scanner-prereqs?view=o365-worldwide#windows-server-requirements)
- Network connectivity to required Azure services as listed in the [same requirements](https://learn.microsoft.com/en-us/microsoft-365/compliance/deploy-scanner-prereqs?view=o365-worldwide#windows-server-requirements)
- AIP unified labeling client installed from the [release history page](https://learn.microsoft.com/en-us/azure/information-protection/rms-client/unifiedlabelingclient-version-release-history)

### SQL Server instance

- Preferably hosted on a different machine than the scanner server
- Meets the documented [SQL Server requirements](https://learn.microsoft.com/en-us/microsoft-365/compliance/deploy-scanner-prereqs?view=o365-worldwide#sql-server-requirements)

### Accounts setup

- Service account with the [documented permissions](https://learn.microsoft.com/en-us/microsoft-365/compliance/deploy-scanner-prereqs?view=o365-worldwide#service-account-requirements), including:
  - Azure AD–synced Active Directory identity
  - Log on locally and log on as a service
  - At least one published label
  - Full-control permissions plus site-collection auditor role in SharePoint (enables targeted scanning)
  - Read/Write/Modify rights to file shares
  - `sysadmin` role on the SQL Server instance

---

## Define the scan cluster and job

1. Sign in to compliance.microsoft.com › **Settings** › **Information protection scanner**.
2. Create a cluster (group of scanners that share the workload):
	- Keep scanners in the same region and connect them to the same SQL instance.
	- Use a simple name and avoid special characters.
3. Create a content scan job:
	- Name the job and select the cluster created above.
	- Choose a schedule:
	  - **Manual** for initial discovery.
	  - **Automatic** once jobs are tested and stable.
	- Choose info types:
	  - **Policy only** if sensitivity labels already include sensitive info type (SIT) rules.
	  - **All** if SIT rules are not defined.
	- Treat recommended labeling as automatic: **Off** when automatic classification already exists, **On** when you rely on recommendations.
	- Enable DLP policy rules: **On** to enforce DLP on on-prem repositories, **Off** otherwise.
	- Enforce sensitivity labeling policy: **Off** for discovery mode, **On** to apply labels.
	- Label files based on content: **On** to inspect and apply labels, **Off** to use a default label only.
	- Default label: **None**, **Policy only**, or **Custom** (one of your published labels).
	- Relabel files: **Off** unless the new label has a higher classification, **On** to relabel whenever conditions match.
	- Preserve modification dates: **On** to keep original timestamps, **Off** to update them when files change.
	- Include or exclude file types as required.
	- Default owner: keep **Scanner account** or choose **Custom** to override file ownership metadata.
	- Set repository owner: leave **Off** or specify a SAMAccountName/UPN/SID to grant owner permissions when classification changes.
4. After saving the job, add target repositories:
	- Network share: `\\Server\Folder`
	- SharePoint library: `http://sharepoint.domain.com/Shared%20Documents/Folder`
	- Local path: `C:\Folder`
	- UNC path: `\\Server\Folder`

---

## Track setup details

Keep a text file with the values gathered so far:

```
The following have been set up:
Scanner account:
SQL instance:
Scan cluster name:

These items are captured next:
App name:
AppId:
AppSecret:
TenantId:
```

---

## Register an Azure AD application

1. Go to portal.azure.com › **App registrations** › **New registration**.
2. Provide a name such as `AIP-Scanner`.
3. Choose **Accounts in this organizational directory only**.
4. Set a redirect URI (Platform: Web, Value: https://localhost).
5. Click **Register** and record the app name, AppId, and TenantId.
6. Under **Certificates & secrets**, create a new client secret (for example, `AIPScannerSecret`), pick an expiration, and save the generated value.
7. Configure API permissions:
	- **Azure Rights Management Services** › Application permissions:
	  - `Content.DelegatedReader`
	  - `Content.DelegatedWriter`
	  - `Content.SuperUser` (optional, allows scanning all protected files)
	- **Microsoft Information Protection Sync Service** › Application permissions:
	  - `UnifiedPolicy.Tenant.Read`
8. Grant admin consent.

---

## Install the scanner on Windows Server

1. Log on to the Windows Server and open PowerShell as an administrator.
2. Capture scanner account credentials:
	```powershell
	$serviceacct = Get-Credential -UserName "domain\user" -Message "ScannerAccount"
	```
3. Install the scanner (uses SQL instance and cluster name from your notes):
	```powershell
	Install-AIPScanner -SqlServerInstance "domain\instance" -Profile "cluster" -ServiceUserCredentials $serviceacct
	```
	The setup creates the database automatically and can take several minutes.
4. Configure authentication once the install completes:
	```powershell
	Set-AIPAuthentication -AppId "AppId" -AppSecret "AppSecret" -TenantId "TenantId" -DelegatedUser "user@domain.com" -OnBehalfOf $serviceacct
	```
5. Run diagnostics to confirm a healthy state (all results should be green):
	```powershell
	Start-AIPScannerDiagnostics -OnBehalfOf $serviceacct
	```
6. In Microsoft Purview (Settings › Information protection scanner), the server should now appear as an idle scanner node.

---

## Run a scan

- In Microsoft Purview: Settings › Information protection scanner › Content scan job, select the job and choose **Scan now**.
- Or, on the Windows Server, run PowerShell as admin and use:
  - `Start-AIPScan`
  - `Get-AIPScannerStatus`
  - `Stop-AIPScan`

---

## Post-scan artifacts

After each scan, reports are saved on the scanner server under `%localappdata%\Microsoft\MSIP\Scanner\Reports`.

---

## Troubleshooting

- Review [Issues with Microsoft Purview Information Protection scanner](https://learn.microsoft.com/en-us/microsoft-365/troubleshoot/information-protection-scanner/resolve-deployment-issues) for deployment guidance.
