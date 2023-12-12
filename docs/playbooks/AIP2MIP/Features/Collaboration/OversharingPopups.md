# Oversharing Popups Playbook

## Introduction

Labeled emails contain sensitive information to your organization to be shared with intended recipients only. Oversharing popups enables an admin to configure popups that ensure the end-user sending a labeled email or attachment is aware of your organization’s policies. These policies can be configured to warn the user before send, request business justification input from the user or block send of the sensitive labeled content. Previously available in AIP add-in, oversharing popups is now available in MIP built-in labeling for preview. For more information about custom settings in AIP Add-in, view our [admin guide for the AIP client](https://learn.microsoft.com/en-us/azure/information-protection/rms-client/clientv2-admin-guide-customizations#implement-pop-up-messages-in-outlook-that-warn-justify-or-block-emails-being-sent).

## Scenarios in Scope

### Get a list of existing Oversharing Popup settings

To determine whether your organization’s current configuration of oversharing popups in AIP add-in is available for preview, please run the following PowerShell cmdlets. You will need an administrator email with Compliance administrator or Global administrator role and the `<policy name>` that is configured with oversharing popups.

1.	Connect to Security & Compliance PowerShell using an administrator email (Link)
2.	Once you have connected to the Security & Compliance PowerShell, get the label policy configuration:

```PS C:\> (Get-LabelPolicy -Identity Global).settings```

The PowerShell terminal will show the label policy configuration that includes all custom settings for that policy.

### Scenarios in-scope for preview

Previously available in AIP add-in, administrators can now use DLP to show popup messages to end users in Outlook desktop for windows for the scenarios below:
1)	Warning messages that prompt users to verify the content that they're sending
2)	Emails that require justification or explicit acknowledgement before they can be sent (DLP override)
3)	Blocked emails that cannot be sent out

|AIP Add-In Custom Setting|Configuration Scenario|
|------------------|------------------|
|OutlookWarnUntrustedCollaborationLabel / OutlookWarnTrustedDomains|#1 Warn Popup and Trusted Domains|
|OutlookJustifyUntrustedCollaborationLabel / OutlookJustifyTrustedDomains|#2 Justify Popup and Trusted Domains|
|OutlookBlockUntrustedCollaborationLabel / OutlookBlockTrustedDomains|#3 Block Popup and Trusted Domains|
|OutlookUnlabeledCollaborationAction|#4 Unlabeled Content Predicate for any Popup|
|OutlookOverrideUnlabeledCollaborationExtensions|#5 File Extension Predicate for any Popup|
|OutlookUnlabeledCollaborationActionOverrideMailBodyBehavior|Unlabeled emails without attachment - Not supported|
|OutlookCollaborationRule|#6 Customized popups|

## General guidance for DLP Configuration:
Create and deploy a [data loss prevention policy](https://learn.microsoft.com/purview/dlp-create-deploy-policy)
1.	Choose what you want to monitor
2.	Choose the Policy Scoping(preview)
3.	Choose where you want to monitor
4.	Choose the conditions that must be matched for a policy to be applied to an item
5.	Choose the action to take when the policy conditions are met
For PowerShell configuration, refer to the [PowerShell reference](https://learn.microsoft.com/powershell/exchange/connect-to-exchange-online-powershell?view=exchange-ps)


## Configuration Steps

To get create and deploy DLP policies, view the Microsoft Purview [DLP docs](https://learn.microsoft.com/en-us/microsoft-365/compliance/dlp-create-deploy-policy?view=o365-worldwide#scenario-2-show-policy-tip-as-oversharing-popup-preview) and create a policy matching scenario 2. For each AIP matched configuration, follow the "Steps to create policy for scenario 2" with the following modifications:

### 1. Warn Popup with Trusted Domains

Skip step 17 and follow the rest of the steps.
This ensures **block access for everyone** is not configured. 

Once deployed, users see this warn popup on send:

![image](https://user-images.githubusercontent.com/25543918/224189550-26887252-0e5d-4c24-9f4f-4918eaac3ff1.png)

### 2. Justify Popup with Trusted Domains

Follow all steps and replace step 20 with the following:

20. Select **Allow overrides from M365 services** and **Require a business justification to override**. (optional) To show the acknowledgement option, select **Require the end user to explicitly acknowledge the override**.

Once deployed, users see this justify popup (with optional acknowledgement option) on send:

![image](https://user-images.githubusercontent.com/25543918/224188889-3d9a0c82-0dad-4c56-b616-b41914c3abb7.png)

### 3. Block Popup with Trusted Domains

Follow all steps.

Once deployed, users see this block popup on send:

![image](https://user-images.githubusercontent.com/25543918/224189075-3e2e32fd-64ca-4720-88ea-718b9cfb953e.png)

!!! info
    In Outlook, untrusted recipients are listed in the policy tip while the email is drafted. Previously in AIP, untrusted recipients were shown in the popup dialogue.

### 4. Unlabeled Content Predicate for any Popup
UX instructions
a.	Choose “Content is not labeled”
b.	Choose the target scope for this predicate evaluation –
iv.	Message & attachment (default): It helps detect if the entire email envelope is unlabeled.
v.	Message only: It helps detect if message body is unlabeled.
vi.	Attachment only: It helps detect if any of the attachments is unlabeled.
![image](https://github.com/microsoft/ComplianceCxE/assets/25543918/1a448e7b-e3da-43f5-9d28-63c436d4395c)

PowerShell instructions:
- Set-DlpComplianceRule -MessageIsNotLabeled $true
- Set-DlpComplianceRule -AttachmentIsNotLabeled $true
- Set-DlpComplianceRule -ContentIsNotLabeled $true 

### 5. File Extension Predicate for any Popup
UX instructions
a.	Choose “File extension is”
b.	Input the extensions you wish to detect or exempt.
![image](https://github.com/microsoft/ComplianceCxE/assets/25543918/36e6a51a-0a6f-45dd-b9c2-2eae690ba6be)

### 6. Customized Popups
Create a JSON file with content for the customized popups:

 ![image](https://github.com/microsoft/ComplianceCxE/assets/25543918/48a9f7b6-e2ef-4f63-92c4-3216021a9841)

The above content could be uploaded for DLP using below steps:
$content = Get-Content "path to the file" | Out-String
New/Set-DlpComplianceRule -Name <Rule_name> -Policy <Policy_name> -NotifyPolicyTipCustomDialog $content -NotifyPolicyTipDisplayOption Dialog 

When the above cmdlet is executed, there will be some validation checks on the content passed for the -AdvancedSettings like char limit, formatting, mandatory presence of 1 default language, etc and the admin will be notified of any errors for correction.

## PowerShell Instructions (Advanced)

DLP policies and rules can also be configured in PowerShell. To configure oversharing popups using PowerShell, first create a DLP policy and add DLP rules for each warn, justify or block popup type.

1.	Configure and scope your DLP Policy using [New-DlpCompliancePolicy](https://learn.microsoft.com/powershell/module/exchange/new-dlpcompliancepolicy?view=exchange-ps#-exchangelocation)
2.	Configure each oversharing rule using [New-DlpComplianceRule](https://learn.microsoft.com/powershell/module/exchange/new-dlpcompliancerule?view=exchange-ps)

To configure a new DLP policy:

```PS C:\> New-DlpCompliancePolicy -Name <DLP Policy Name> -ExchangeLocation All```

The sample DLP policy is scoped to all users in your organization. Scope your DLP Policies using ```-ExchangeSenderMemberOf``` and ```-ExchangeSenderMemberOfException```.

|Parameter|Configuration|
|-----------------|-----------------|
|-ContentContainsSensitiveInformation|Configures one or more sensitivity label conditions. This sample includes one. At least one label is mandatory.|
|-ExceptIfRecipientDomainIs|List of trusted domains.|
|-NotifyAllowOverride|"WithJustification" enables justification radio buttons, "WithoutJustification" disables them.|
|-NotifyOverrideRequirements|"WithAcknowledgement" enables the new acknowledgement option. Optional.|

### Scenario #1 Warn Popup and Trusted Domains

To configure a new DLP rule:

```PS C:\> New-DlpComplianceRule -Name <DLP Rule Name> -Policy <DLP Policy Name> -NotifyUser Owner -NotifyPolicyTipDisplayOption "Dialog" -ContentContainsSensitiveInformation @(@{operator = "And"; groups = @(@{operator="Or";name="Default";labels=@(@{name=<Label GUID>;type="Sensitivity"})})}) -ExceptIfRecipientDomainIs @("contoso.com","microsoft.com")```

### Scenario #2 Justify Popup and Trusted Domains

To configure a new DLP rule:

```PS C:\> New-DlpComplianceRule -Name <DLP Rule Name> -Policy <DLP Policy Name> -NotifyUser Owner -NotifyPolicyTipDisplayOption "Dialog" -BlockAccess $true -ContentContainsSensitiveInformation @(@{operator = "And"; groups = @(@{operator = "Or"; name = "Default"; labels = @(@{name=<Label GUID 1>;type="Sensitivity"},@{name=<Label GUID 2>;type="Sensitivity"})})}) -ExceptIfRecipientDomainIs @("contoso.com","microsoft.com") -NotifyAllowOverride "WithJustification"```

### Scenario #3 Block Popup and Trusted Domains

To configure a new DLP rule:

```PS C:\> New-DlpComplianceRule -Name <DLP Rule Name> -Policy <DLP Policy Name> -NotifyUser Owner -NotifyPolicyTipDisplayOption "Dialog" -BlockAccess $true -ContentContainsSensitiveInformation @(@{operator = "And"; groups = @(@{operator = "Or"; name = "Default"; labels = @(@{name=<Label GUID 1>;type="Sensitivity"},@{name=<Label GUID 2>;type="Sensitivity"})})}) -ExceptIfRecipientDomainIs @("contoso.com","microsoft.com")```

## Additional Customization Features

### Customize Policy Tips

In DLP Rule configuration, select “Customize the policy tip text” and enter the custom text option.

![image](https://user-images.githubusercontent.com/25543918/224189143-744a276d-4c0d-481e-b742-edde5558e11d.png)

Localize your custom policy tips with ```Set-DlpComplianceRule cmdlet``` and [-NotifyPolicyTipCustomTextTranslations](https://learn.microsoft.com/powershell/module/exchange/new-dlpcompliancerule#-notifypolicytipcustomtexttranslations) in Security & Compliance PowerShell.

### Customize Compliance URL for “Learn More”

In DLP Rule configuration, select “Provide a compliance URL for the end user to learn more about your organization’s policies.”

![image](https://user-images.githubusercontent.com/25543918/224189191-85c27113-8380-47a5-aa85-dbfc2afdaaf6.png)

When a user clicks “Learn more” in the popup body, the user will be redirected to the link configured.

![image](https://user-images.githubusercontent.com/25543918/224189203-c66a1518-e654-4199-8efe-61ed82c7f222.png)

## New Feature: Acknowledgement Option

In DLP Rule configuration, select “Allow overrides from M365 services” and “Require the end user to explicitly acknowledge the override” to enable the new acknowledgement option. 

![image](https://user-images.githubusercontent.com/25543918/224189217-fb90029c-2d42-4b98-b8bd-bdebe9b7cfeb.png)

If “Require a business justification to override” is selected, the business justification radio button options will be enabled in the popup UX.
In Outlook, the acknowledgement option requires the user to explicitly check the box to enable send:

![image](https://user-images.githubusercontent.com/25543918/224189240-399005f0-2a99-41ac-a4e2-d09dd0fdcac7.png)

## Additional existing features supported by DLP Oversharing for E5 users
- Top DLP Predicates
- Content contains Sensitive Info Types (Works for email and unencrypted Microsoft 365 and PDF files)
- Content contains sensitivity labels (Works for email and Office & PDF file types)
- Content is not labeled
- Content is shared
- Sender is
- Sender is member of (Only Distribution lists, Azure-based Dynamic Distribution groups, and email-enabled Security groups are supported.)
- Sender domain is
- Recipient is
- Recipient is a member of (Only Distribution lists, Azure-based Dynamic Distribution groups, and email-enabled Security groups are supported.)
- Recipient domain is
- Subject contains words
- Advanced classifiers like Named Entities, Exact Data Match, Trainable Classifiers, Cred scan 

## Known limitations for Private Preview
- OOB template with EXO workload would not have the Message contains and Attachment contains predicates available
- For the four new predicates (Message is not labeled/Attachment is not labeled/Message contains/Attachment contains) with per component eval turned ON, the matched conditions might not be accurate in the GIR/Audit/end user email/Activity Explorer/Alert.
- Content contains SITs with per component evaluation ON will not support Trainable Classifiers
- For the new 2 predicates Message contains & Attachment contains (irrespective of per component eval), Trainable Classifiers will not be supported

