# **Finding Emails in a Mail Folder older then a specific date**

If you are doing any archiving, clean-up or just searching for Messages that are older then a specific date you will need to make use of a filter on the receivedDateTime. 

**The receivedDateTime property**

This property represents when the Message was received expressed and stored in UTC time, this means when you query it you should also make sure your query value is in UTC. 

So for instance if I where looking for all the Email in the JunkEmail folder older then 30 days I could use

```
/v1.0/users('gscales@datarumble.com')/MailFolders('JunkEmail')/messages?$Top=10&$filter=receivedDateTime lt 2020-10-21T00:00:00Z
```

If the mailboxes are in a TimeZone other then UTC then first convert the actual date to the local date you want to include to UTC eg in Powershell something like

```
(Get-Date).adddays(-30).Date.ToUniversalTime().ToString("o")
```

This means if my timezone is +11 UTC my actual query time would look like

2020-10-20T13:00:00.0000000Z based on an Actual day value of 2020-10-21T00:00:00.0000000+11:00

**Precision**

The last thing to consider around receivedDateTime  is precision, Exchange Server stores the Item with a precision down to the milliseconds in EWS you could adjust the precision of your queries down to the millisecond however the Graph API doesn't allow you to do this. So if your querying items in the Graph and using the GT or LT operators you may see them work more as GE and LE due to the loss in precision.  While this sounds like a little thing it can be quite painful depending on what your trying to do.

Orderby

By default when you use a filter you will get the results back in ascending order so if you want to see the latest messages first make sure you use

```
OrderBy=receivedDateTime desc
```

if you add more fields into the Filter clause they will also need to be added to the OrderBy 

I've created a Powershell Script to go along with the doc call ItemAgeModule.ps1 available from https://github.com/gscales/Powershell-Scripts/blob/master/Graph101/ItemAgeModule.ps1

Some Examples are get the last 10 email in the Inbox older then 30 days would look like

```
Get-EmailOlderThan -MailboxName gscales@domain.com -FolderName Inbox -OlderThanDays 30 -MessageCount 10
```

To get all the Email in the Junk-Email Folder older the 30 days use the following and it will page them back 100 items at a time

```
Get-EmailOlderThan -MailboxName gscales@domain.com -FolderName Inbox -OlderThanDays 30
```

To get all Email older then 30 days with a specific subject then you can use (note the OrderByExtraFields is needed which should have a comma seperated list of extra fields used in the filter)

```
Get-EmailOlderThan -MailboxName gscales@domain.com -FolderName Inbox -OlderThanDays 30 -filter "Subject eq 'test'" -OrderByExtraFields subject
```

**Moving the Emails you find**

As mentioned one of the main reasons you may have for finding messages of a specific age is to Archive, Move or Delete them. For this you need to use the move operation on the Item endpoint eg in graph this is basically a post like

```
https://graph.microsoft.com/v1.0/me/messages/AAK.....=/move
```

with the destination folder in the body of the POST (either a folderId or WellknownFolderName) eg

```
{  "destinationId": "archive" }
```

Would move messages to the mailbox's archive folder (not my in-place archive)

So if I wanted to archive all the Messages in my Inbox from a particular sender that was older then 2 years I could use

```
Move-EmailOlderThan -MailboxName gscales@domain.com -FolderName Inbox -OlderThanDays 30 -filter "from/emailAddress/address eq 'user@domain.com'" -OrderByExtraFields "from/emailAddress/address" -Destination archive
```

This code makes use of batching and moves items in lots of 4 as not to go over the concurrent user quota

Or if I wanted to soft delete items I could move them to the RecoverableItemsDeletions folder

eg

```
Move-EmailOlderThan -MailboxName gscales@domain.com -FolderName Inbox -OlderThanDays 30 -filter "from/emailAddress/address eq 'info@twitter.com'" -OrderByExtraFields "from/emailAddress/address" -Destination RecoverableItemsDeletions -verbose
```

the script for this post can be found https://github.com/gscales/Powershell-Scripts/blob/master/Graph101/ItemAgeModule.ps1

[Back to 101 Graph PowerShell Binder Mini-Site](https://gscales.github.io/Graph-Powershell-101-Binder/)