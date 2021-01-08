# Sending an Email from a Shared Mailbox **

There is detailed documentation around Sending email as another user using the Graph API here https://docs.microsoft.com/en-us/graph/outlook-send-mail-from-other-user this doc will just look at some scripted examples of this

**Background:** While for the most part I'll be talking about actual shared Mailboxes https://docs.microsoft.com/en-us/microsoft-365/admin/email/create-a-shared-mailbox?view=o365-worldwide the concepts are also valid for send from a Mailbox you have been grant either SendAS or SendOnBehalf.

**Permissions:**

To use the Graph API to access to Send Mailbox from a  Shared Mailbox folder your Application registration must have been granted (and consented to) following shared permissions

![](https://github.com/gscales/Graph-Powershell-101-Binder/raw/master/bin/Images/msperms.png)



It's important to note this just gives you the permissions for you to Send Mail (at the API level) for Shared Mailboxes through the Graph API, the actual Mailbox Access level permissions (SendAs) are still controlled by Azure AD and need to have been granted by one of many different mechanisms

- Mailbox Permissions via Add-RecipientPermission (or the Exchange Admin Center) (Admin Only)
- SendOnBehalf can be granted via Outlook Delegates

see https://docs.microsoft.com/en-us/exchange/recipients-in-exchange-online/manage-permissions-for-recipients for more details

**Access Token**

Once your Application Registration is correct and you have the underlying Mailbox Folder permissions you just need to ensure that you include the correct scope in your authentication(token) request eg

```
scope=Mail.Send.Shared&redirect_uri=...&code=..&grant_type=authorization_code&client_id=5471030d-f311-4c5d-91ef-74ca885463a7
```

**Mail Submission EndPoint ** 

You can use either the 

```
https://graph.microsoft.com/v1.0/me/SendMail
```

or 

```
https://graph.microsoft.com/v1.0/users('sharedmailbox1@domainxyz.com')/SendMail
```

Endpoint when sending a Message the difference between these two affects which mailbox the Sent Message gets stored in. If you want the Sent Message to be stored in the SharedMailbox then use the ShareMailboxes Submission URL.

SendAS

**Setting the From Address**

The only property that will differ when sending a Mail is the From address eg

```
{
   "Message":{
      "Subject":"Test Message1",
      "From":{
         "EmailAddress":{
            "Name":"gscales@datarumble.com",
            "Address":"gscales@datarumble.com"
         }
      },
      "Body":{
         "ContentType":"HTML",
         "Content":"Hello"
      },
      "ToRecipients":[
         {
            "EmailAddress":{
               "Name":"blah@datarumble.com",
               "Address":"blah@datarumble.com"
            }
         }
      ]
   },
   "SaveToSentItems":"true"
}
```



**SendOnBehalf**

Set the From Address to the credentials you using and Sender Address to who you are sending on Behalf of

```
{
   "Message":{
      "Subject":"Test Message1",
      "Sender":{
         "EmailAddress":{
            "Name":"sharedmailbox1@datarumble.com",
            "Address":"sharedmailbox1@datarumble.com"
         }
      },
      "From":{
         "EmailAddress":{
            "Name":"gscales@datarumble.com",
            "Address":"gscales@datarumble.com"
         }
      },
      "Body":{
         "ContentType":"HTML",
         "Content":"Hello"
      },
      "ToRecipients":[
         {
            "EmailAddress":{
               "Name":"blah@datarumble.com",
               "Address":"blah@datarumble.com"
            }
         }
      ]
   },
   "SaveToSentItems":"true"
}
```

**X-AnchorMailbox Header**

Like all Exchange API requests you should include this header to ensure the correct routing of requests for SharedMailbox request set this to the Shared Mailbox (rather then that of the credentials) eg.

```
POST https://graph.microsoft.com/v1.0/users('sharedmailbox1@datarumble.com')/SendMail HTTP/1.1
AnchorMailbox: sharedmailbox1@datarumble.com
Authorization: Bearer ey...
User-Agent: GraphBasicsPs101
Host: graph.microsoft.com

```

Note if your using the /Me/SendMail endpoint then Anchor it to the Authentication creds instead

**Script examples**

I've created a demo script module to go along with this post (and other Shared Mailbox post) which can be found https://github.com/gscales/Powershell-Scripts/blob/master/Graph101/SharedMailboxOps.ps1

**Send a Mail as a Shared Mailbox and Store it in the Shared Mailboxes Sent Items Folder**

```
Send-SharedMessage -TargetMailbox sharedmailbox1@datarumble.com -Subject "Test Message1" -To blah@datarumble.com -Body "Hello" -LogonMailbox gscales@datarumble.com
```

**Send a Mail as a Shared Mailbox and Store it in the Sending users Sent Items Folder**

```
Send-SharedMessage -TargetMailbox sharedmailbox1@datarumble.com -Subject "Test Message1" -To blah@datarumble.com -Body "Hello" -LogonMailbox gscales@datarumble.com -SaveToLogonMailbox
```

If you don't want the message you sending saved to the Sent Items folder use

```
Send-SharedMessage -TargetMailbox sharedmailbox1@datarumble.com -Subject "Test Message1" -To blah@datarumble.com -Body "Hello" -LogonMailbox gscales@datarumble.com -DontSaveToSentItems
```



[Back to 101 Graph PowerShell Binder Mini-Site](https://gscales.github.io/Graph-Powershell-101-Binder/)

