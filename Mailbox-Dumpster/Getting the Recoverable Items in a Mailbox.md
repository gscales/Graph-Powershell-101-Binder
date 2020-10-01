### Getting the Recoverable Items in a Mailbox

(Sample Script for this Article can be found )

In Exchange Online [Single Item Recovery](https://docs.microsoft.com/en-us/exchange/recipients/user-mailboxes/single-item-recovery?view=exchserver-2019) is enabled by default which means when are user (or a piece of code) deletes a message located in Mailbox folder or empties the Deleted-Items these items are then moved to one of the Recoverable Item Folders. The Recoverable Items folders are folders that are located under the NON_IPM_Subtree of a Mailbox (wellknownfolder root) meaning that aren't visible to user. Eg

![image-20200917131803998](https://github.com/gscales/Graph-Powershell-101-Binder/raw/master/bin/Images/dumpsterFolders.JPG)



The Deletions and Purges folders are the folders you would work with most as this is where you can restore (or find Messages) that may have been deleted accidentally (or by Bad Actors etc).

**Types of Deletes**

When an Item is deleted there are typically 3 delete modes

- Move to Deleted-Items : this is where the Item is just moved to the DeleteItems folder which means it still visible
- SoftDelete : Typically this happens when someone empties the DeleteItems folder or does a Shift Delete. Emails that are soft deleted are moved to the Deletions folder where they are held until the Retention period expires. These items are still recoverable using the dumpster feature in Outlook,OWA etc
- HardDelete : This can be done by an application like MFCMapi or using code in EWS. When items are hard deleted they are move to the Purges folder where they will sit until the next cycle of the [Managed Folder Assistant](https://docs.microsoft.com/en-us/exchange/policy-and-compliance/mrm/configure-managed-folder-assistant?view=exchserver-2019) (MFA)

To get Items from the soft deleted Items in a Mailbox using the Microsoft Graph API you can make a request like

```
https://graph.microsoft.com/v1.0/users('user@domain.com')/MailFolders('RecoverableItemsDeletions')/messages?$Top=10
```

*Using the Sample Script* 

```
Get-RecoverableItemsDeletions -MailboxName user@domain.com -MessageCount 100
```

To get Items from the purges folder using the Microsoft Graph API you can make a request like

```
https://graph.microsoft.com/v1.0/users('user@domain.com')/MailFolders('RecoverableItemsPurges')/messages?$Top=10
```

*Using the Sample Script* 

```
Get-RecoverableItemsPurges -Mailbox user@domain.com -MessageCount 100
```

**Filtering the results**

Because these folders can contain a large number of Items you will generally want to filter the results that are being returned. To do this in the Microsoft Graph API you can use Filters (or Search). For example if you had the InternetMessageId of a Message that you needed to investigate but had already been deleted by the user you could use the following search to find that message

```
https://graph.microsoft.com/v1.0/users('user@domain.com')/MailFolders('RecoverableItemsDeletions')/messages?$filter=internetMessageId eq '<SIXPR04MB07943FE6A08A363FD78084F3C81B0@SIXPR04MB0794.apcprd04.prod.outlook.com>'
```

*Using the Sample Script* 

```
Get-RecoverableItemsDeletions -Mailbox user@domain.com -filter "internetMessageId eq '<SIXPR04MB07943FE6A08A363FD78084F3C81B0@SIXPR04MB0794.apcprd04.prod.outlook.com>'"
```

A more basic example would be to search based on the Sender

```
https://graph.microsoft.com/v1.0/users('user@domain.com')/MailFolders('RecoverableItemsDeletions')/messages?$filter=from/emailAddress/address eq 'gscales@domain.com'
```

*Using the Sample Script* 

```
Get-RecoverableItemsDeletions -Mailbox user@domain.com -filter "from/emailAddress/address eq 'gscales@datarumble.com'"
```

**Exporting a Deleted Message**

In the case where you need to export the Message you have found in the above step to do further analysis on it you can do this in the Microsoft Graph using the following request which will return the MIME content of a Message. You can then save this content as a .EML file which will make it accessible using a Mailbox client like Outlook or Thunderbird etc.

```
https://graph.microsoft.com/v1.0/users('gscales@datarumble.com')/messages/AAMkAD../$value
```

*Using the Sample Script* 

```
$Item = Get-RecoverableItemsDeletions -Mailbox gscales@datarumble.com -filter "internetMessageId eq '<SIXPR04MB07943FE6A08A363FD78084F3C81B0@SIXPR04MB0794.apcprd04.prod.outlook.com>'"
Invoke-ExportItem -MailboxName gscales@datarumble.com -item $Item | Out-File -FilePath c:\temp\exportmail.eml
```





