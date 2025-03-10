---
title: "How to identify the type of hold placed on an Exchange Online mailbox"
ms.author: markjjo
author: markjjo
manager: laurawi
ms.date: 
audience: ITPro
ms.topic: article
ms.service: O365-seccomp
ms.collection: M365-security-compliance
localization_priority: Normal
search.appverid:
- MET150
ms.assetid: 6057daa8-6372-4e77-a636-7ea599a76128
description: "Learn how to identify the different types of hold that can be placed on an Office 365 mailbox. These types of holds include Litigation Hold, eDiscovery holds, and Office 365 retention policies. You can also determine if a user has been excluded from an organization-wide retention policy"

---

# How to identify the type of hold placed on an Exchange Online mailbox

This article explains how to identify holds placed on Exchange Online mailboxes in Office 365.

Office 365 offers several ways that your organization can prevent mailbox content from being permanently deleted. This allows your organization to retain content to meet compliance regulations or during legal and other types of investigations. Here's a list of the retention features (also called *holds*) in Office 365:

- **[Litigation Hold](create-a-litigation-hold.md):** Holds that are applied to user mailboxes in Exchange Online.

- **[eDiscovery hold](ediscovery-cases.md#step-4-place-content-locations-on-hold):** Holds that are associated with an eDiscovery case in the security and compliance center. eDiscovery holds can be applied to user mailboxes and to the corresponding mailbox for Office 365 Groups and Microsoft Teams.

- **[In-Place Hold](https://docs.microsoft.com/Exchange/security-and-compliance/create-or-remove-in-place-holds):** Holds that are applied to user mailboxes by using the In-Place eDiscovery & Hold tool in the Exchange admin center in Exchange Online.

- **[Office 365 retention policies](retention-policies.md):** Can be configured to retain (or retain and then delete) content in user mailboxes in Exchange Online and in the corresponding mailbox for Office 365 Groups and Microsoft Teams. You can also create a retention policy to retain Skype for Business Conversations, which are stored in user mailboxes.

  There are two types of Office 365 retention policies that can be assigned to mailboxes.

    - **Specific location retention policies:** These are policies that are assigned to the content locations of specific users. You use the **Get-Mailbox** cmdlet in Exchange Online PowerShell to get information about retention policies assigned to specific mailboxes.

    - **Organization-wide retention policies:** These are policies that are assigned to all content locations in your organization. You use the **Get-OrganizationConfig** cmdlet in Exchange Online PowerShell to get information about organization-wide retention policies.
  For more information, see the "Applying a retention policy to an entire organization or specific locations" section in [Overview of Office 365 retention policies](retention-policies.md#applying-a-retention-policy-to-an-entire-organization-or-specific-locations).

- **[Office 365 retention labels](labels.md):** If a user applies an Office 365 retention label (one that's configured to retain content or retain and then delete content) to *any* folder or item in their mailbox, a hold is placed on the mailbox as if the mailbox was placed on Litigation Hold or assigned to an Office 365 retention policy. For more information, see the [Identifying mailboxes on hold because a retention label has been applied to a folder or item](#identifying-mailboxes-on-hold-because-a-retention-label-has-been-applied-to-a-folder-or-item) section in this article.

To manage mailboxes on hold, you may have to identify the type of hold that's placed on a mailbox so that you can perform tasks such as changing the hold duration, temporarily or permanently removing the hold, or excluding a mailbox from an Office 365 retention policy. In these cases, the first step is to identify the type of hold placed on the mailbox. And because multiple holds (and different types of holds) can be placed on a single mailbox, you have to identify all holds placed on a mailbox if you want to remove or change a hold.

## Step 1: Obtain the GUID for holds placed on a mailbox

You can run the following two cmdlets in Exchange Online PowerShell to get the GUID of the holds that are placed on a mailbox. After you obtain a GUID, you use it to identify the specific hold in Step 2. A Litigation Hold isn't identified by a GUID. Litigation Holds are either enabled or disabled for a mailbox.

- **Get-Mailbox:** Use this cmdlet to determine whether Litigation Hold is enabled for a mailbox and to get the GUIDs for eDiscovery holds, In-Place Holds, and Office 365 retention policies that are specifically assigned to a mailbox. The output of this cmdlet will also indicate if a mailbox has been explicitly excluded from an organization-wide retention policy.

- **Get-OrganizationConfig:** Use this cmdlet to get the GUIDs for organization-wide retention policies.

To connect to Exchange Online PowerShell, see [Connect to Exchange Online PowerShell](https://docs.microsoft.com/powershell/exchange/exchange-online/connect-to-exchange-online-powershell/connect-to-exchange-online-powershell?view=exchange-ps).

### Get-Mailbox

Run the following command to get information about the holds and Office 365 retention policies applied to a mailbox.

```powershell
Get-Mailbox <username> | FL LitigationHoldEnabled,InPlaceHolds
```

> [!TIP]
> If there are too many values in the InPlaceHolds property and not all of them are displayed, you can run the `Get-Mailbox <username> | Select-Object -ExpandProperty InPlaceHolds` command to display each GUID on a separate line.

The following table describes how to identify different types of holds based on the values in the *InPlaceHolds* property when you run the **Get-Mailbox** cmdlet.


|Hold type  |Example value  |How to identify the hold  |
|---------|---------|---------|
|Litigation Hold     |    `True`     |     Litigation Hold is enabled for a mailbox when the *LitigationHoldEnabled* property is set to `True`.    |
|eDiscovery hold     |  `UniH7d895d48-7e23-4a8d-8346-533c3beac15d`       |   The *InPlaceHolds property* contains the GUID of any hold associated with an eDiscovery case in the security and compliance center. You can tell this is an eDiscovery hold because the GUID starts with the `UniH` prefix (which denotes a Unified Hold).      |
|In-Place Hold     |     `c0ba3ce811b6432a8751430937152491` <br/> or <br/> `cld9c0a984ca74b457fbe4504bf7d3e00de`  |     The *InPlaceHolds* property contains the GUID of the In-Place Hold that's placed on the mailbox. You can tell this is an In-Place Hold because the GUID either doesn't start with a prefix or it starts with the `cld` prefix.     |
|Office 365 retention policy specifically applied to the mailbox     |    `mbxcdbbb86ce60342489bff371876e7f224:1` <br/> or <br/> `skp127d7cf1076947929bf136b7a2a8c36f:3`     |     The InPlaceHolds property contains GUIDs of any specific location retention policy that's applied to the mailbox. You can identify retention policies because the GUID starts with the `mbx` or the `skp` prefix. The `skp` prefix indicates that the retention policy is applied to Skype for Business conversations in the user's mailbox.    |
|Excluded from an organization-wide Office 365 retention policy     |   `-mbxe9b52bf7ab3b46a286308ecb29624696`      |     If a mailbox is excluded from an organization-wide Office 365 retention policy, the GUID for the retention policy the mailbox is excluded from is displayed in the InPlaceHolds property and is identified by the `-mbx` prefix.    |

### Get-OrganizationConfig
If the *InPlaceHolds* property is empty when you run the **Get-Mailbox** cmdlet, there still may be one or more organization-wide Office 365 retention policies applied to the mailbox. Run the following command in Exchange Online PowerShell to get a list of GUIDs for organization-wide Office 365 retention policies.

```powershell
Get-OrganizationConfig | FL InPlaceHolds
```

> [!TIP]
> If there are too many values in the InPlaceHolds property and not all of them are displayed, you can run the `Get-OrganizationConfig | Select-Object -ExpandProperty InPlaceHolds` command to display each GUID on a separate line.

The following table describes the different types of organization-wide holds and how to identify each type based on the GUIDs contained in *InPlaceHolds* property when you run the **Get-OrganizationConfig** cmdlet.


|Hold type  |Example value  |Description  |
|---------|---------|---------|
|Office 365 retention policies applied to Exchange mailboxes, Exchange public folders, and Teams chats    |      `mbx7cfb30345d454ac0a989ab3041051209:2`   |   Organization-wide retention policies applied to Exchange mailboxes, Exchange public folders, and 1xN chats in Microsoft Teams are identified by GUIDs that start with the `mbx` prefix. Note 1xN chats are stored in the mailbox of the individual chat participants.      |
|Office 365 retention policy applied to Office 365 Groups and Teams channel messages     |   `grp1a0a132ee8944501a4bb6a452ec31171:3`      |    Organization-wide retention policies applied to Office 365 groups and channel messages in Microsoft Teams are identified by GUIDs that start with the `grp` prefix. Note channel messages are stored in the group mailbox that is associated with a Microsoft Team.     |

For more information retention policies applied to Microsoft Teams, see the "Teams location" section [Overview of retention policies](retention-policies.md#applying-a-retention-policy-to-an-entire-organization-or-specific-locations).

### Understanding the format of the InPlaceHolds value for retention policies

In addition to the prefix (mbx, skp, or grp) that identifies an item in the InPlaceHolds property as an Office 365 retention policy, the value also contains a suffix that identifies the type of retention action that's configured for the policy. For example, the action suffix is highlighted in bold type in the following examples:

   `skp127d7cf1076947929bf136b7a2a8c36f`**:1**

   `mbx7cfb30345d454ac0a989ab3041051209`**:2**

   `grp1a0a132ee8944501a4bb6a452ec31171`**:3**

The following table defines the three possible retention actions:

|Value  |Description  |
|---------|---------|
|**1**     | Indicates that the retention policy is configured to delete items. The policy doesn't retain items.        |
|**2**    |    Indicates that the retention policy is configured to hold items. The policy doesn't delete items after the retention period expires.     |
|**3**     |   Indicates that the retention policy is configured to hold items and then delete them after the retention period expires.      |

For more information about retention actions, see the "Retaining content for a specific period of time" section in [Overview of retention policies](retention-policies.md#retaining-content-for-a-specific-period-of-time).
   
## Step 2: Use the GUID to identify the hold

After you obtain the GUID for a hold that is applied to a mailbox, the next step is to use that GUID to identify the hold. The following sections show how to identify the name of the hold (and other information) by using the hold GUID.

### eDiscovery holds

Run the following commands in Security & Compliance Center PowerShell to identify an eDiscovery hold that's applied to the mailbox. Use the GUID (not including the UniH prefix) for the eDiscovery hold that you identified in Step 1. The first command creates a variable that contains information about the hold. This variable is used in the other commands. The second command displays the name of the eDiscovery case the hold is associated with. The third command displays the name of the hold and a list of the mailboxes the hold applies to.

```powershell
$CaseHold = Get-CaseHoldPolicy <hold GUID without prefix>
```

```powershell
Get-ComplianceCase $CaseHold.CaseId | FL Name
```

```powershell
$CaseHold | FL Name,ExchangeLocation
```

To connect to Security & Compliance Center PowerShell, see  [Connect to Security & Compliance Center PowerShell](https://docs.microsoft.com/powershell/exchange/office-365-scc/connect-to-scc-powershell/connect-to-scc-powershell?view=exchange-ps).

### In-Place Holds

Run the following command in Exchange Online PowerShell to identify the In-Place Hold that's applied to the mailbox. Use the GUID for the In-Place Hold that you identified in Step 1. The command displays the name of the hold and a list of the mailboxes the hold applies to.

```powershell
Get-MailboxSearch -InPlaceHoldIdentity <hold GUID> | FL Name,SourceMailboxes
```

If the GUID for the In-Place Hold starts with the `cld` prefix, be sure to include the prefix when running the previous command.

### Office 365 retention policies

Run the following command in Security & Compliance Center PowerShell to identity the Office 365 retention policy (organization-wide or specific location) that's applied to the mailbox. Use the GUID (not including the mbx, skp, or grp prefix or the action suffix) that you identified in Step 1.

```powershell
Get-RetentionCompliancePolicy <hold GUID without prefix or suffix> -DistributionDetail  | FL Name,*Location
```

## Identifying mailboxes on hold because a retention label has been applied to a folder or item

Whenever a user applies a retention label that's configured to retain content or retain and then delete content to any folder or item in their mailbox, the *ComplianceTagHoldApplied* mailbox property is set to **True**. When this happens, the mailbox is considered to be on hold, as if it was placed on Litigation Hold or assigned to an Office 365 retention policy. When the *ComplianceTagHoldApplied* property is set to **True**, the following things may occur:

- If the mailbox or the user's Office 365 user account is deleted, the mailbox becomes an [inactive mailbox](inactive-mailboxes-in-office-365.md).
- You aren't able to disable the mailbox (either the primary mailbox or the archive mailbox, if it's enabled).
- Items in the mailbox may be retained longer than expected. This is because the mailbox is on hold and therefore no items are permanently deleted (purged).

To view the value of the *ComplianceTagHoldApplied* property, run the following command in Exchange Online PowerShell:

```powershell
Get-Mailbox <username> |FL ComplianceTagHoldApplied
```

For more information about retention labels, see [Overview of Office 365 retention labels](labels.md).

## Managing mailboxes on delay hold

After any type of hold is removed from a mailbox, the value of the *DelayHoldApplied* mailbox property is set to **True**. This occurs the next time the Managed Folder Assistant processes the mailbox and detects that a hold was removed. This is called a *delay hold* and means that the actual removal of the hold is delayed for 30 days to prevent data from being permanently deleted (purged) from the mailbox. This gives admins an opportunity to search for or recover mailbox items that will be purged after the hold is removed. When a delay hold is placed on the mailbox, the mailbox is still considered to be on hold for an unlimited duration, as if the mailbox was on Litigation Hold. After 30 days, the delay hold expires, and Office 365 will automatically attempt to remove the delay hold (by setting the *DelayHoldApplied* property to **False**) so that the hold is removed. After the *DelayHoldApplied* property to **False**, items that are marked for removal are purged the next time the mailbox is processed by the Managed Folder Assistant.

To view the value for the *DelayHoldApplied* property for a mailbox, run the following command in Exchange Online PowerShell.

```powershell
Get-Mailbox <username> | FL DelayHoldApplied
```

To remove the delay hold before it expires, you can run the following command in Exchange Online PowerShell: 
 
```powershell
Set-Mailbox <username> -RemoveDelayHoldApplied
```

You must be assigned the Legal Hold role in Exchange Online to use the *RemoveDelayHoldApplied* parameter 

To remove the delay hold on an inactive mailbox, run the following command in Exchange Online PowerShell:

```powershell
Set-Mailbox <DN or Exchange GUID> -InactiveMailbox -RemoveDelayHoldApplied
```

> [!TIP]
> The best way to specify an inactive mailbox in the previous command is to use its Distinguished Name or Exchange GUID value. Using one of these values helps prevent accidentally specifying the wrong mailbox. 

## Next steps

After you identify the holds that are applied to a mailbox, you can perform tasks such as changing the duration of the hold, temporarily or permanently removing the hold, or excluding an inactive mailbox from an Office 365 retention policy. For more information about performing tasks related to holds, see the one of the following topics:

- Run the [Set-RetentionCompliancePolicy -AddExchangeLocationException \<user mailbox>](https://docs.microsoft.com/powershell/module/exchange/policy-and-compliance-retention/Set-RetentionCompliancePolicy?view=exchange-ps) command in Security & Compliance Center PowerShell to exclude a mailbox from an organization-wide Office 365 retention policy. This command can only be used for retention policies where the value for the *ExchangeLocation* property equals `All`.

- Run the [Set-Mailbox -ExcludeFromOrgHolds \<hold GUID without prefix or suffix>](https://docs.microsoft.com/powershell/module/exchange/mailboxes/set-mailbox?view=exchange-ps) command in Exchange Online PowerShell to exclude an inactive mailbox from an organization-wide Office 365 retention policy.

- [Change the hold duration for an inactive mailbox in Office 365](change-the-hold-duration-for-an-inactive-mailbox.md)

- [Delete an inactive mailbox in Office 365](delete-an-inactive-mailbox.md)

- [Delete items in the Recoverable Items folder of cloud-based mailboxes on hold](delete-items-in-the-recoverable-items-folder-of-mailboxes-on-hold.md)
