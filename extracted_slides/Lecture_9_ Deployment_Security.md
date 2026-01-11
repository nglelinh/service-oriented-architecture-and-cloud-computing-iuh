# Lecture_9_ Deployment_Security


## Slide 1

### Deployment and Security


## Slide 2

### Deploying Azure Resources


Infrastructure as Code
ARMs and Biceps
Terraform

GitHub Actions / DevOps Build Pipelines


## Slide 3

### Deployment is Easy Necessary


“Infrastructure as Code” – The ability to deploy all your services in the cloud from code rather than manually.

Important for:
IT management of resources
Geo-replication / Global availability and scale
Security


## Slide 4

### Infrastructure as Code Options


ARM Templates


Bicep Language


Terraform


Pro: Azure can generate them.

Con: Difficult to read and edit.


Pro: From Microsoft. Easy to read.

Con: Lacking in examples and docs.


Pro: From HashiCorp. Multicloud.

Con: Proprietary and Pricey


Bicep language for deploying Azure resources - Azure Resource Manager | Microsoft Docs


Templates overview - Azure Resource Manager | Microsoft Docs


Docs overview | hashicorp/azurerm | Terraform Registry


## Slide 5

### {
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    },
    "storageAccountName": {
      "type": "string",
      "defaultValue": "[format(myStorage{0}', uniqueString(resourceGroup().id))]"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-06-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2",
      "properties": {
        "accessTier": "Hot"
      }
    }
  ]
}


IaC LanguageLearning Curve


param location string = resourceGroup().location
param storageAccountName string = myStorage${uniqueString(resourceGroup().id)}'

resource storageAccount 'Microsoft.Storage/storageAccounts@2021-06-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    accessTier: 'Hot'
  }
}


resource "azurerm_storage_account" "mystorage" {
name = "myStorage${random_integer.id.result}"
resource_group_name = azurerm_resource_group.rg.name
location = azurerm_resource_group.rg.location 
account_tier = "Standard"
account_replication_type = "LRS"
tags = { environment = ”dev" }
}


ARM (JSON)


BICEP


TF (HCL)


## Slide 6

### Deploying with GitHub Actions


## Slide 7

### Steps to Deploy with Terraform in GitHub Actions


## Slide 8

### Security, Compliance, and Data Governance


Role-Based Access Controls
Azure Active Directory
Access Control Lists in Azure Storage

Compliance
HIPAA, HITRUST, SOX, ISO
Blueprints, etc.

Data Governance
Azure Purview


## Slide 9

### Azure Active Directory


Identity management for users and applications.
Single Sign-On (SSO)
App registrations
External applications
Service Principals and Managed Identities
Role assignments
Group organization


## Slide 10

### RBAC andAccess Control Lists


Common Role-Based Access Controls:
Owner
Grants full access to manage all resources, including the ability to assign roles in Azure RBAC.
Contributor
Grants full access to manage all resources, but does not allow you to assign roles in Azure RBAC
Reader
View all resources but does not allow you to make any changes.

RBAC: Broad Access to Resources


ACLs: Finer Access to Data


## Slide 11

### Compliancenoun - com·​pli·​ance | \ kəm-ˈplī-ən(t)s \ “Conformity in fulfilling official requirements”


## Slide 12

### Global
CIS benchmark 
CSA STAR Attestation 
CSA STAR Certification 
CSA STAR self-assessment 
SOC 1 
SOC 2 
SOC 3
ISO 20000-1 
ISO 22301 
ISO 27001 
ISO 27017 
ISO 27018 
ISO 27701 
ISO 9001 
WCAG | US government
CJIS 
CMMC 
CNSSI 1253 
DFARS 
DoD IL2 
DoD IL4 
DoD IL5 
DoD IL6 
DoE 10 CFR Part 810 
EAR 
FedRAMP 
FIPS 140
ICD 503 
IRS 1075 
ITAR 
JSIG 
NDAA 
NIST 800-161 
NIST 800-171 
NIST 800-53 
NIST 800-63 
NIST CSF 
Section 508 VPATs 
StateRAMP | Financial services
23 NYCRR Part 500 (US) 
AFM and DNB (Netherlands) 
AMF and ACPR (France) 
APRA (Australia) 
CFTC 1.31 (US) 
EBA (EU) 
FCA and PRA (UK) 
FFIEC (US) 
FINMA (Switzerland)
FINRA 4511 (US) 
FISC (Japan) 
FSA (Denmark) 
GLBA (US) 
KNF (Poland) 
MAS and ABS (Singapore) 
NBB and FSMA (Belgium) 
OSFI (Canada) 
OSPAR (Singapore) 
PCI 3DS 
PCI DSS 
RBI and IRDAI (India) 
SEC 17a-4 (US) 
SEC Regulation SCI (US) 
SOX (US) 
TruSight | Automotive, education, energy, media, and telecommunication
CDSA 
DPP (UK) 
FACT (UK) 
FERPA (US) 
MPA 
GSMA 
NERC (US) 
TISAX | 
 |  |  | Regional - Americas
Argentina PDPA 
Canada privacy laws 
Canada Protected B 
US CCPA | Regional - EMEA
EU Cloud CoC 
EU EN 301 549 
ENISA IAF 
EU GDPR 
EU Model Clauses 
Germany C5 
Germany IT-Grundschutz workbook 
Netherlands BIR 2012
Russia personal data law 
Spain ENS High 
Spain LOPD 
UAE DESC 
UK Cyber Essentials Plus 
UK G-Cloud 
UK PASF
Healthcare and life sciences
ASIP HDS (France) 
EPCS (US) 
GxP (FDA 21 CFR Part 11) 
HIPAA (US) 
HITRUST 
MARS-E (US) 
NEN 7510 (Netherlands) |  |  | Regional - Asia Pacific
Australia IRAP 
China GB 18030 
China DJCP (MLPS) 
China TCS 
India MeitY 
Japan CS Gold Mark 
Japan ISMAP 
Japan My Number Act 
Korea K-ISMS 
New Zealand ISPC 
Singapore MTCS | Regional - EMEA
EU Cloud CoC 
EU EN 301 549 
ENISA IAF 
EU GDPR 
EU Model Clauses 
Germany C5 
Germany IT-Grundschutz workbook 
Netherlands BIR 2012
Russia personal data law 
Spain ENS High 
Spain LOPD 
UAE DESC 
UK Cyber Essentials Plus 
UK G-Cloud 
UK PASF


Azure Compliance Offerings


## Slide 13

### HIPAA, HITECT, and HITRUST


HIPAA: Health Insurance Portability and Accountability Act of 1996
Sets requirements for using, disclosing, and handling protected health information (PHI)

HITECH: Health Information Technology for Economic and Clinical Health Act of 2009
Extends HIPAA for electronic health records and more modern, data-driven technology in healthcare

HITRUST: Health Information Trust Alliance
A private company that sets standards, based on HIPAA/HITECH,  and provides certification.


https://docs.microsoft.com/en-us/azure/compliance/offerings/offering-hipaa-us


## Slide 14

### SOX


Sarbanes-Oxley Act of 2002
US federal law from the  Securities and Exchange Commission (SEC)
Mandates financial record keeping practices
Thanks to the ENRON and WorldCom scandals


## Slide 15

### Azure Blueprints


Set of organizational standards and requirements for compliance or policy adherence.

Orchestrate the deployment of artifacts:
Role Assignments
Policy Assignments
Azure Resource Manager templates (ARM templates)
Resource Groups


## Slide 16

### ISO 27001


International Standard for information security
14 main Controls
Information security policies, HR security, asset management, access control, cryptography, physical security, etc.
