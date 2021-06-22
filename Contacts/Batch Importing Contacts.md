**Using Batching to improve the speed of Contact creation**

To create a Contact in an Exchange Online Mailbox with the Graph API requires a post to the Contacts endpoint of a Mailbox similar to the following example 

 ![](https://github.com/gscales/Graph-Powershell-101-Binder/raw/master/bin/Images/contactcreate1.PNG)



Now if you want to do this many hundreds of times for example if your importing contacts from a CSV file, if you used the above single request method this will take quite some time to execute each request. To speed this up you can use batching which means you have multiple posts in one request that the server will execute either concurrently or serially if you use the dependson header eg

 ![](https://github.com/gscales/Graph-Powershell-101-Binder/raw/master/bin/Images/batchContact2.PNG)

The size your batch can be is either 4 (POST requests) if your using  concurrent batch's or 20 if your using the dependson header, this is because of throttling which I covered in this [post](https://gsexdev.blogspot.com/2020/09/the-mailboxconcurrency-limit-and-using.html). Generally I find that using concurrent batches is faster, because you have 4 threads executing concurrently then you should expect about a 3-4 time performance improvement over a single request execution in your code.

**Authentication** 

If your targeting multiple mailboxes for a Contact Import you most likely want to use Service Principal Authentication which I've covered in https://github.com/gscales/Graph-Powershell-101-Binder/blob/master/Authentication/Service%20Principal%20Authentication.md

**Doing this in PowerShell**

If your importing contacts from a CSV using PowerShell then it has a number of built in cmdlets that help you do this easily the first is Import-CSV eg

```
        Import-Csv -Path $CSVFile | ForEach-Object{
            $User = $_
```

Once you have each of your CSV lines in an Object you can then create a Custom PSObject from each line that can then be converted to a JSON string at time you want to do you POST using PowerShell's ConvertTo-json cmdlet

```powershell
        $batchCount = 1
        $BatchRequestContent = @{}
        $BatchRequestContent.add("requests",@())
        Import-Csv -Path $CSVFile | ForEach-Object{
            $User = $_
            $ContactsBody = @{ 
                'givenName'  = $User.FirstName
                'middleName' = $User.MiddleName
                'surname' =  $User.LastName
                'displayName' = ($User.LastName + "," +  $User.FirstName)
                'fileAs' = ($User.LastName + "," +  $User.FirstName)
                'jobTitle' = $User.Title
                'companyName' = $User.Company
                'department' = $User.Dept
                'mobilePhone' = $User.Mobile
                'homePhones' =  @($User.TelephoneNumber)
                'emailAddresses' =  @( @{
                    'address' = $User.Email
                    'name' = $User.DisplayName
                })
            }
            $BatchEntry = @{}
            $BatchEntry.Add("id",[Int32]$batchCount)
            $BatchEntry.Add("method","POST")
            $BatchEntry.Add("url","/users/$TargetUser/contacts")
            $BatchEntry.Add("body",$ContactsBody)
            $BatchHeaders = @{
                'Content-Type' =  "application/json"
            } 
            $BatchEntry.Add("headers",$BatchHeaders)
            $BatchRequestContent["requests"] += $BatchEntry
            $batchCount++
            if($batchCount -gt 4){
                $headers = @{
                    'Authorization' = "Bearer $AccessToken"
                    'x-AnchorMailbox' = "$TargetUser"
                }
                $RequestURL = "https://graph.microsoft.com/v1.0/`$batch"
                $BatchResponse = (Invoke-RestMethod -Method POST -Uri $RequestURL -UserAgent "GraphBasicsPs101" -Headers $headers -Body (ConvertTo-json  $BatchRequestContent -depth 10 -Compress) -ContentType "application/json" )   
                if($BatchResponse.responses){
                    foreach($Response in $BatchResponse.responses){                        
                        if([Int32]$Response.status -eq 201){
                            $rptObject.ContactSucess++
                            Write-Output ("Contact Created " + $Response.body.displayName)
                        }else{
                            $rptObject.ErrorCount++
                            $rptObject.Errors += $Response.status
                            if([Int32]$Response.status -eq 429){
                                $rptObject.ThrottleCount++                               
                                if(!$TimeOutServed){
                                    Write-Verbose($Response.Headers.'Retry-After')
                                    Write-Verbose("Serving Throttling Timeout " + $Response.Headers.'Retry-After')
                                    Start-sleep -Seconds $Response.Headers.'Retry-After'
                                    $TimeOutServed = $true
                                }
                            }
                            Write-Error "Error creating Contact"
                        }
                    } 
                }else{
                    Write-Error "Error creating Contact"
                }
                $batchCount = 1
                $BatchRequestContent = @{}
                $BatchRequestContent.add("requests",@())
            }
            
        }
```

I've put a full sample of using Service Principal Authentication, Import-CSV on Git Hub here https://github.com/gscales/Powershell-Scripts/blob/master/Graph101/BatchContactCreation.ps1 and there is a sample CSV for this https://github.com/gscales/Powershell-Scripts/blob/master/Graph101/contacts.csv

[Back to 101 Graph PowerShell Binder Mini-Site](https://gscales.github.io/Graph-Powershell-101-Binder/)

