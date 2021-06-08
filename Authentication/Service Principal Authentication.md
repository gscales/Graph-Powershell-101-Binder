# Service Principal Authentication

**Background:**  Service Principal Authentication in Azure allows an Application to authenticate using a SSL Certificate or Client secret. This application can then use Application permissions to access one or more resource in a tenant. For instance an application could be granted read access to every users mail in a tenant, this access can also be scope to restrict it to only certain mailbox in a tenant using https://docs.microsoft.com/en-us/graph/auth-limit-mailbox-access.

**Authentication requirements:**

To use Service Principal Authentication in the application registration you must either add a SSL certificate (eg Self singed certificate which you have the private key for) or create a Client secret in the Azure portal. eg

![](https://github.com/gscales/Graph-Powershell-101-Binder/raw/master/bin/Images/certauth.PNG)

You then need to assign one or more application permissions in the Application registration.

To perform the authentication against Azure you must create a local JWT token using the SSL Certificate you uploaded to the Application and sign that with the private key associated with the certificate. This client assertions in then used to generate an Access token eg

![](https://raw.githubusercontent.com/gscales/Graph-Powershell-101-Binder/master/bin/Images/localjwtauth.PNG)

I've created a demo script module that can be used to perform a Service principal authentication located here https://github.com/gscales/Powershell-Scripts/blob/master/Graph101/SPAuth.ps1

To use this function you first need to acquire the local certificate which could be either stored in the Local Certificates Store or in a PFX file with a password. eg

```
$certificate = Get-ChildItem -Path Cert:LocalMachine\MY\FCE7328421DDDB4CC8EA59C51B6F156765272CFA
```

or from a PFX file

```
$certificate = Get-PfxCertificate -FilePath C:\temp\certo.pfx
```

Once you have the certificate you can then request the token like

```
$token = Get-AccessTokenForGraphFromCertificate -Certificate $Certificate -ClientId e6fd6f09-9c63-4b30-877a-3520ee1e1e9a -TenantDomain domain.com
```

This access_token can the be used in Request like

```
$AccessToken = $token.access_token
$headers = @{'Authorization' = "Bearer $AccessToken"}
Invoke-RestMethod -Uri https://graph.microsoft.com/v1.0/users/gscales@datarumble.com/mailboxsettings -Headers $headers
```



[Back to 101 Graph PowerShell Binder Mini-Site](https://gscales.github.io/Graph-Powershell-101-Binder/)

