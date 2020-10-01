### Restoring a Deleted Item using the Microsoft Graph API

(Sample Script for this Article can be found https://github.com/gscales/Powershell-Scripts/blob/master/Graph101/Dumpster.ps1 )

As a pre-req for this article it builds on [this](https://github.com/gscales/Graph-Powershell-101-Binder/blob/master/Mailbox-Dumpster/Getting%20the%20Recoverable%20Items%20(Dumpster%20v2)%20Folders.md) and [this](https://github.com/gscales/Graph-Powershell-101-Binder/blob/master/Mailbox-Dumpster/Getting%20the%20Recoverable%20Items%20in%20a%20Mailbox.md)

When an Email item is deleted in a Mailbox a special property called the Last Active Parent FolderId (LAPFID) property gets set on an Email. This enables the [original folder item recovery feature](https://blogs.technet.microsoft.com/exchange/2017/06/13/announcing-original-folder-item-recovery/) that was rolled out to both Exchange Online and Exchange OnPrem (2016) last year. To Access the LAPFID property you need to use the extended property definition for it eg

![](https://2.bp.blogspot.com/-zjDt025tiZE/W8gj_Z7PhgI/AAAAAAAACII/PZYtjPazGmgwamTK2brOwXZkwjByk1ctwCLcBGAs/s1600/Lafid.JPG)

In the Microsoft Graph this means making a request such as the following 

```
https://graph.microsoft.com/v1.0/users('user@domain.com')/MailFolders('RecoverableItemsDeletions')/messages?$Top=10&$expand=SingleValueExtendedProperties($filter=(Id%20eq%20'Binary%200x348A')) 
```

![](https://github.com/gscales/Graph-Powershell-101-Binder/raw/master/bin/Images/LAPFidResult.PNG)

The Value is a Base64 Encoded version of the HexString in the first Image

To understand what that property Id value represents you need to first understand a little bit more about the Folder EntryId format (PR_EntryId) that exchange uses which is documented in this Exchange Protocol Document https://msdn.microsoft.com/en-us/library/ee217297(v=exchg.80).aspx . A different visual representation of this with the individual components highlighted would look something like this

[![img](https://4.bp.blogspot.com/-dSlqjHm-FRs/W8guVCIJVGI/AAAAAAAACIU/4NmQ5CZshFA736OIM6-f5vU3OiOJFzjCACLcBGAs/s1600/highlighted.JPG)](https://4.bp.blogspot.com/-dSlqjHm-FRs/W8guVCIJVGI/AAAAAAAACIU/4NmQ5CZshFA736OIM6-f5vU3OiOJFzjCACLcBGAs/s1600/highlighted.JPG)

Hopefully from this you can see that the LAPFID is comprised of the DatabaseGUID and GlobalCounter constituents of the FolderEntryId. To make sense of the LAPFID you need to resolve it back to the actual Mailbox Folder that it represents.  To resolve that back to an Actual Mailbox folder you could either enumerate all the folders parse down and Index the Id's or build it using an existing FolderId if your sure it exits in the same Mailbox (eg it isn't from the Online Archive).

In the Sample script I've taken the first approach which does a batch request of all the folders in a Mailbox and index's the folder Id's so when you do any of the Item request you should see the folders they where deleted from for example

```
PS C:\> Get-RecoverableItemsDeletions -Mailbox gscales@datarumble.com -filter "internetMessageId eq '<SIXPR04MB07943FE6A08A363FD78084F3C81B0@SIXPR04MB0794.apcprd04.prod.outlook.com>'"                                                         

                            : 1
LastActiveParentFolderPath                  : \\Sent Items
LastActiveParentFolder                      : @{id=AQM....==;
                                              childFolderCount=0; unreadItemCount=0; totalItemCount=832; singleValueExt
                                              endedProperties@odata.context=https://graph.microsoft.com/v1.0/$metadata#
                                              users('gscales%40datarumble.com')/mailFolders('msgfolderroot')/childFolde
                                              rs('A...D%3D')/singleValueExte
                                              ndedProperties; singleValueExtendedProperties=System.Object[];
                                              FolderPath=\\Sent Items; FolderRestURI=https://graph.microsoft.com/v1.0/u
                                              sers('user@domain.com')/MailFolders('AQMkADczNDE4YWEAMC03ZWZiLTQyM
                                              2QtODA1Yi02MmIyNmJkYWMyNmQALgAAA74c3T2WBidIkPPeS33fvkkBAHUQR-0Y6jBNnUCxQo
                                              usINAAAAIBCQAAAA=='); PR_ENTRYID=00000000BE1CDD3D9606274890F3DE4B7DDFBE49
                                              0100751047FD18EA304D9D40B1428BAC20D00000000001090000}
```

To Restore an Item back to its last parent folder id you just need to move it using the FolderId of the destination folder you moving it to for example the request to restore the above item back to its Last Active folder you would need to use a POST in the Graph API that look like

```
POST https://graph.microsoft.com/v1.0/users('gscales@datarumble.com')/MailFolders('AQMkAD....')/messages/AAMkA.../move

{
        "destinationId": "AQMkADczNDE4....."
}
```

*In the Sample Script*

```
$Item = Get-RecoverableItemsDeletions -Mailbox user@domain.com -filter "internetMessageId eq '<SIXPR04MB07943FE6A08A363FD78084F3C81B0@SIXPR04MB0794.apcprd04.prod.outlook.com>'"                                              Invoke-RestoreItemToLastActiveParentFolderId -Item $Item -MailboxName user@domain.com  
```

​                  

(Sample Script for this Article can be found https://github.com/gscales/Powershell-Scripts/blob/master/Graph101/Dumpster.ps1 )