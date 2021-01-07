# **Accessing a Shared Mailbox Folder**

**Background:** While for the most part I'll be talking about actual shared Mailboxes https://docs.microsoft.com/en-us/microsoft-365/admin/email/create-a-shared-mailbox?view=o365-worldwide the concepts are also valid for accessing Shared Mailbox folders that you have been given access to via Outlook Delegates (or Folder permissions).

**Permissions:**

To use the Graph API to access a Shared Mailbox folder your Application registration must have been granted (and consented to) one of the following shared permissions

![](https://github.com/gscales/Graph-Powershell-101-Binder/raw/master/bin/Images/sharep.PNG)

(if your going to be making changes use ReadWrite otherwise just Read is better)

It's important to note this just gives you the permissions for you to access Shared Mailboxes through the Graph API, the actual Mailbox Access level permissions are still controlled by Exchange and may have been granted by one of many different mechanisms

- Mailbox Folder Permissions in Outlook or OWA (this can be done by an end-user)
- Mailbox Delegates in OWA or Outlook (this can be done by an end-user)
- Mailbox Permissions via Add-MailboxPermission (or the Exchange Admin Center) (Admin Only)
- Mailbox Folder Permission via Add-MailboxFolderPermission (Admin Only)

**Access Token**

Once your Application Registration is correct and you have the underlying Mailbox Folder permissions you just need to ensure that you include the correct scope in your authentication(token) request eg

```
scope=User.Read.All+Mail.Read+Mail.Read.Shared&redirect_uri=...&code=..&grant_type=authorization_code&client_id=5471030d-f311-4c5d-91ef-74ca885463a7
```

**Folder Access Requests** 

The actual request you make should then just be targeted to the Shared Mailbox eg to get the Inbox of a Shared Mailbox you would use something like

```
https://graph.microsoft.com/v1.0/users('sharedmailbox1@domainxyz.com')/MailFolders/Inbox
```

**Folder Item Access Requests**

```
https://graph.microsoft.com/v1.0/users('sharedmailbox1@domainxyz.com')/MailFolders/Inbox/Messages
```

**X-AnchorMailbox Header**

Like all Exchange API requests you should include this header to ensure the correct routing of requests for SharedMailbox request set this to the Shared Mailbox (rather then that of the credentials) eg.

```
GET https://graph.microsoft.com/v1.0/users('sharedmailbox1@datarumble.com')/MailFolders('Inbox')Binary%200xfff')) HTTP/1.1
AnchorMailbox: sharedmailbox1@datarumble.com
Authorization: Bearer ey...
User-Agent: GraphBasicsPs101
Host: graph.microsoft.com

```

**Script examples**

I've created a demo script module to go along with this post (and other Shared Mailbox post) which can be found https://github.com/gscales/Powershell-Scripts/blob/master/Graph101/SharedMailboxOps.ps1

**Get the Inbox of a Shared Mailbox**

```
Get-SharedWellKnownFolder -FolderName Inbox -TargetMailbox sharedmailbox1@datarumble.com -LogonMailbox gsclaes@datarumble.com
```

**Get a Custom Folder from a Shared Mailbox**

Note because this actually searches for a Folder it would request full access to all the Mailbox folder, otherwise you need to know the underlying FolderId and use Get-SharedFolderFromId

```
Get-SharedFolderFromPath -FolderPath ''\Inbox\folder1' -TargetMailbox sharedmailbox1@datarumble.com -LogonMailbox gscales@datarumble.com
```

**Read the last Item in the Inbox of a Shared Mailbox**

```
Get-SharedLastEmail -TargetMailbox SharedMalbox1@datarumble.com -LogonMailbox gscales@datarumble.com -MessageCount 1
```

**Read the last 2 Items in from the SendItems of a Shared Mailbox**

```
Get-SharedLastEmail -TargetMailbox jcool@datarumble.com -LogonMailbox gscales@datarumble.com -MessageCount 2 -FolderName SentItems
```

[Back to 101 Graph PowerShell Binder Mini-Site](https://gscales.github.io/Graph-Powershell-101-Binder/)

