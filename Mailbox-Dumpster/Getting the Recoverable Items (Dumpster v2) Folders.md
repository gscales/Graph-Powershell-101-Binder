### **Getting the Recoverable Items (Dumpster v2) Folders**

In Exchange Online [Single Item Recovery](https://docs.microsoft.com/en-us/exchange/recipients/user-mailboxes/single-item-recovery?view=exchserver-2019) is enabled by default which means when a user (or a piece of code) deletes a message located in Mailbox folder or empties the Deleted-Items these items are then moved to one of the Recoverable Item Folders. The Recoverable Items folders are folders that are located under the NON_IPM_Subtree of a Mailbox (wellknownfolder root) meaning that are not visible to user. Eg

![image-20200917131803998](https://github.com/gscales/Graph-Powershell-101-Binder/raw/master/bin/Images/dumpsterFolders.JPG)



The Deletions and Purges folder are the ones you would work with most as this is where you can restore (or find Messages) that may have been deleted accidentally (or by Bad Actors etc).

In the Microsoft Graph API to display all the Recoverable Items Folders as in the above Image you can use a Request like

```
 https://graph.microsoft.com/v1.0/users('user@domain.com')/MailFolders('RecoverableItemsRoot')/ChildFolders 
```

If you wanted to include the FolderSize in the response you can use the PR_Folder_Size property which would produce a request like

```
https://graph.microsoft.com/v1.0/users('user@domain.com')/MailFolders('RecoverableItemsRoot')/ChildFolders?$expand=SingleValueExtendedProperties($filter=(Id%20eq%20'Long%200x66B3'))
```

I've include an example of this in the following script 

Use it like 

```
Get-RecoverableItemsFolders -MailboxName user@domain.com
```

