This method works for external git/Mercurial repositories, DropBox and OneDrive.

Here's how to trigger a (fetch) deployment from an external repository:

e.g. You have a git repository at https://example.com/git/RepositoryName.git and you'd like to notify Kudu to trigger a deployment for every new commit.

##### Raw POST
```
POST /deploy HTTP/1.1

Host: $SiteLevelUsername:SiteLevelPassword@<SiteName>.scm.azurewebsites.net
Content-Type: application/json
Accept: application/json
X-SITE-DEPLOYMENT-ID: <SiteName>
Transfer-encoding: chunked

{
    "format":"basic",
    "url":"https://username:password@example.com/git/RepositoryName.git"
}
````
For Mercurial (hg), unless the `url` begins with `hg@`, do add `"scm": "hg"` to the JSON payload.  

Optionally, you can pass `/deploy?isAsync=true` which will make the deployment to run asynchronously.  The response will return immediately with `Location` header where to poll the status of the deployment.

`$SiteLevelUsername:SiteLevelPassword` are [Site-Level credentials](https://github.com/projectkudu/kudu/wiki/Deployment-credentials). They are a good fit here being distinct for each site and randomly generated.

#### Deploy from specific commit id

You could in addition specify commit id (or branch) as part of git URL to deploy from; for instance, ..

```
    "url":"https://username:password@example.com/git/RepositoryName.git#commit_id"
```

#### Azure App Service Specific (using Azure PowerShell):

e.g. Using the "External Repository" deployment method in the Azure Portal.

In addition to the POST above you can accomplish the same thing by calling `Invoke-AzureRmResourceAction`:
````
# Action sync
Invoke-AzureRmResourceAction -ResourceGroupName <ResourceGroupName> `
                             -ResourceType Microsoft.Web/sites `
                             -ResourceName <WebAppName> `
                             -Action sync `
                             -ApiVersion 2015-08-01 `
                             -Force -Verbose
````

For a GitHub specific guide see this page here: [Deploying from GitHub](https://github.com/projectkudu/kudu/wiki/Deploying-from-github)
