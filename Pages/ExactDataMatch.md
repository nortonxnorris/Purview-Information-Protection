# Exact Data Match Guide

The updated Exact Data Match (EDM) experience makes it easier to build custom sensitive info type (SIT) classifiers. You need EDM when out-of-the-box SITs miss nuanced or frequently changing data, such as employee records that include name, date of birth, and employee ID. EDM relies on uploaded source data to craft a custom classifier, significantly reducing false positives.


**Tips**

- Use simple classifier names with no spaces.
- When uploading CSV/TSV data, keep column names free of spaces and underscores.

---

## Step 1 – Create a sample upload file

- Match the sample file’s columns to the final upload file.
- Use CSV or TSV format and keep the sample under 2.5 MB.
- Actual upload limits:
  - Up to 100 million rows.
  - Up to 32 columns per data source.
  - Up to 10 searchable columns.

---

## Step 2 – Create the EDM classifier (new experience)

Microsoft Purview › Data Classification › Classifiers › **Create EDM Classifier**

- Provide a name and description.
- Upload the sample file to auto-generate the schema, or define it manually.
- Validate column names and data types.
- Choose up to 10 unique primary elements (for example, SSN instead of name or DOB).
- Configure case sensitivity and delimiter handling.
- Review the auto-generated High and Medium confidence rules; customize as needed.
- After creation, record the schema name (needed later in Step 5).

---

## Step 3 – Create the EDM_DataUploaders security group

- Add every operator who will hash and upload data to this Microsoft 365 security group.

---

## Step 4 – Choose the hashing/upload topology

**Option 1: Single device**

- Use when the workstation is secure and can store plaintext sensitive data.

**Option 2: Separate devices**

- Hash data on a managed, secure device and upload from a different (possibly public-facing) device.
- Install and authorize the EDM upload tool on every participating machine.
- Download the EDM upload tool from the [official documentation](https://learn.microsoft.com/en-us/microsoft-365/compliance/sit-get-started-exact-data-match-hash-upload?view=o365-worldwide#links-to-edm-upload-agent-by-subscription-type).

---

## Step 5 – Prepare and authorize devices

Perform the one-time setup below (paths are examples):

1. Create directories:
	- `C:\EDM`
	- `C:\EDM\Hash` (hashed output lives here)
	- `C:\EDM\Data` (store plaintext upload files here)
2. In an elevated PowerShell session, navigate to the EDM upload tool folder and authorize the device:
	```powershell
	.\EdmUploadAgent.exe /Authorize
	```
	- Sign in with an account that belongs to the `EDM_DataUploaders` group.
3. Download the schema XML and note the filename for later:
	```powershell
	.\EdmUploadAgent.exe /SaveSchema /DataStoreName <schema-name> /OutputDir C:\EDM\Data
	```

---

## Step 6a – Hash and upload from a single device

1. Validate the upload file against the schema:
	```powershell
	.\EdmUploadAgent.exe /ValidateData /DataFile C:\EDM\Data\<upload-file> /Schema C:\EDM\Data\<schema.xml>
	```
2. Hash and upload in one step:
	```powershell
	.\EdmUploadAgent.exe /UploadData /DataStoreName <schema-name> /DataFile C:\EDM\Data\<upload-file> /HashLocation C:\EDM\Hash /Schema C:\EDM\Data\<schema.xml> /AllowedBadLinesPercentage 0
	```
3. Confirm completion:
	```powershell
	.\EdmUploadAgent.exe /GetDataStore
	```
4. In Microsoft Purview, verify the EDM indexing status.

---

## Step 6b – Hash and upload from separate devices

1. On the secure hashing device, create the hash file:
	```powershell
	.\EdmUploadAgent.exe /CreateHash /DataFile C:\EDM\Data\<upload-file> /HashLocation C:\EDM\Hash /Schema C:\EDM\Data\<schema.xml> /AllowedBadLinesPercentage 0
	```
2. Transfer the generated `.edmHash` files from `C:\EDM\Hash` to the upload device.
3. On the upload device, submit the hash:
	```powershell
	.\EdmUploadAgent.exe /UploadHash /DataStoreName <schema-name> /HashFile C:\EDM\Hash\<file.edmHash>
	```
4. Confirm completion and monitor the EDM indexing status:
	```powershell
	.\EdmUploadAgent.exe /GetDataStore
	```
