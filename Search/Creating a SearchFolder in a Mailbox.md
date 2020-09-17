# **Creating a SearchFolder in a Mailbox using Microsoft Graph and PowerShell**

Search folders are one of the suite of Exchange search options you can use programmatically to provide users with a different view of their mailbox data in Outlook desktop (in Office365 Search Folders do not work in OWA or Outlook Mobile). Essentially a search folder is like a regular mailbox folder, except that it contains only links to messages in other folders that meet the criteria set in the search filter restriction which means that search folders are great for nonchanging, nondynamic queries.

Creating a Search Folder

To create a SearchFolder you have 4 variables you can/need to set

- SearchFolderName which is the DisplayName for your searchfolder
- Filter which is the Graph Filter string 
- Root Folder to Search
- Whether to Search nested folders (eg do you want a deep traversal)

Eg this produces a JSON post request like the following

```
{
  "@odata.type": "microsoft.graph.mailSearchFolder",
  "displayName": "Important Stuff",
  "includeNestedFolders": true,
  "sourceFolderIds": ["MsgFolderRoot"],
  "filterQuery": "contains(subject, 'Important')"
}
```

which you should be posted to SearchFolders WellKnownFolder endpoint

```
https://graph.microsoft.com/v1.0/me/mailFolders('SearchFolders')/childFolders
```

(This is the Finder folder in the NON_IPM_Subtree in your mailbox)

The documentation for the filterQuery parameter can be found on https://docs.microsoft.com/en-us/graph/query-parameters#filter-parameter

**Example Script for create a Search-Folder**

I have a full script https://github.com/gscales/Powershell-Scripts/blob/master/Graph101/SearchFolder.ps1 that can be used to create a SearchFolder in a Mailbox using the Microsoft Graph. Example

```
Invoke-CreateSearchFolder -MailboxName user@domain.onmicrosoft.com -SearchFolderName 'Important Stuff' -Filter "contains(subject, 'Important')"
```

To See all the available Search Folders use 

```
Get-SearchFolders -MailboxName user@domain.onmicrosoft.com 
```

This just does a paged get Childfolders Request like

```
https://graph.microsoft.com/v1.0/me/mailFolders('SearchFolders')/childFolders
```

To Delete a Search Folder use

```
Invoke-RemoveSearchFolder -MailboxName user@domain.onmicrosoft.com -SearchFolderName 'FoldertoRemove'
```

in this case the Code will search for the folder by name using a query like

```
https://graph.microsoft.com/v1.0/me/mailFolders('SearchFolders')/childFolders?$Filter=displayname eq 'FolderToRemove'
```

[Back to 101 Graph PowerShell Binder Mini-Site](https://gscales.github.io/Graph-Powershell-101-Binder/)