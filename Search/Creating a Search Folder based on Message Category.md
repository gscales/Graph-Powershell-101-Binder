# **Creating a Search Folder based on a Message Category**

Searching on the Categories property of a Message can pose a challenge because this property is a Multivalued property eg in Message

![image-20200917131803998](https://github.com/gscales/Graph-Powershell-101-Binder/raw/master/bin/Images/image-20200917131803998.png)

So this needs to be queried in a different way then a normal String property where you could use a number of filter options (eg eq, contains,startswith). In EWS it was only possible to query this property using AQS because of the way SearchFilters translated to the underlying ROP based restrictions used by the Exchange Mailbox Store.  In Graph the Linq format in Filters does translate more favourable so can be used eg the following simple query can find Messages based on a specific category

```
https://graph.microsoft.com/v1.0/me/messages?filter=Categories/any(a:a+eq+'Green+Category')
```

you can create a SearchFolder based on a query such as this which would produce a SearchFolder Creation request like

```
{
  "@odata.type": "microsoft.graph.mailSearchFolder",
  "displayName": "Green Email",
  "includeNestedFolders": true,
  "sourceFolderIds": ["MsgFolderRoot"],
  "filterQuery": "Categories/any(a:a+eq+'Green+Category')"
}
```

Based on a previous post on Create Search Folders I have this script https://github.com/gscales/Powershell-Scripts/blob/master/Graph101/SearchFolder.ps1 which you can use to Create Category Search folder eg

```
Invoke-CreateCategorySearchFolder -MailboxName user@domain.onmicrosoft.com -SearchFolderName GDPRTest -CategoryName "GDPR"
```

