This API is accessed the same way as the git endpoint. e.g. if your git URL is `https://yoursite.scm.azurewebsites.net/yoursite.git`, then the API to get the list of deployments will be `https://yoursite.scm.azurewebsites.net/deployments`.

The credentials you use are the same as when you git push. See [[Deployment-credentials]] for more details.


## scm

Note: in older builds, use `/live/scm` instead of `/api/scm`

    GET /api/scm/info
    Get information about the repository
    
    POST /api/scm/clean	
    Clean the repository, using 'git clean -xdff'
    
    DELETE /api/scm	
    Delete the repository
    	
## Command

    POST /api/command
    Executes an arbitrary command line and return its output

The JSON body of the post should look like this, passing the command and the folder it should run in:

    {
        "command": 'echo Hello World',
        "dir": 'site\\repository'
    }

    	
## VFS
The VFS API is based on <https://github.com/c9/vfs-http-adapter>. Paths with trailing slashes are treated as directories.

**Note:** when updating or deleting a file, ETag behavior will apply. You can pass a `If-Match: "*"` header to disable the ETag check.

    GET /api/vfs/{path}
    Gets a file at path

    GET /api/vfs/{path}/
    Lists files at directory specified by path.

    PUT /api/vfs/{path}
    Puts a file at path.

    PUT /api/vfs/{path}/
    Creates a directory at path. The path can be nested, e.g. `folder1/folder2`.

    DELETE /api/vfs/{path}
    Delete the file at path.

## Zip
   The Zip API allows downloading folders as zip files, or expanding zip files into folders.

    GET /api/zip/{path}
    Zip up and download the specified folder. The zip doesn't include the top folder itself.

    PUT /api/zip/{path}
    Upload a zip file which gets expanded into the specified folder. Existing files are not deleted
    unless they need to be overwritten by files in the zip. The path can be nested (e.g. `folder1/folder2`), and needs to exist.

## Deployment

    GET /api/deployments
    Get the list of all deployments
    
    GET /api/deployments/{id}
    Get a deployment

    PUT /api/deployments/{id}
    This API has 2 behaviors based on the payload.

      - Redeploy a current or previous deployment
        {id} is optional. If not provided, redeploy the current head.
        payload (optional):
        {
          clean: false,        // optional, whether to run git clean -xdf before building
          needFileUpdate: true // optional, whether to run git checkout (good to skip checkout big repo) 
        }

      - Add a deployment status to history
        {id} is required.
        payload (required):
        {
          status: 3|4,                               // required, whether failed (3) or success (4) status
          message: "commit message",                 // required, commit message  
          author: "John Smith",                      // required
          deployer: "VSO",                           // required, deployer service      
          author_email: "john@smith.com",            // optional
          start_time: "2015-09-09T22:39:50.0052315Z",// optional, when the deployment started
          end_time: "2015-09-09T22:39:50.0052315Z",  // optional, when the deployment ended
          active: true,                              // optional, this is latest deployment
          details: "https://foo.com/details"         // optional, url to get more details
        }
        
    DELETE /api/deployments/{id}
    Delete a deployment
    
    GET /api/deployments/{id}/log
    Get the list of log entries for a deployment
    
    GET /api/deployments/{id}/log/{logId}
    Get the list of log entry details for a log entry

    POST /deploy
    Payload: { format: 'basic', url: 'https://github.com/SomeOrg/SomeRepo.git' } 
    Or when using ssh: { format: 'basic', url: 'git@github.com:SomeOrg/SomeRepo.git' }
    Trigger a new deployment from that repository 

    POST /deploy
    Payload: { format: 'basic', url: 'https://github.com/SomeOrg/SomeRepo.git#<commit_sha>' } 
    Trigger a new deployment from that repository for certain commit. Note that <commit_sha> needs to be a sha and not a branch/tag.

See this page for information on switching branch: [[Deployment branch|Deployment branch]]

## SSHKey

    GET /api/sshkey?ensurePublicKey=1
    Generate the private/public key pair (if not already created), and return the public key as a string. e.g. "ssh-rsa AAAAB3NzaC1yc..."

The PUT method still exists, but should be avoided as it involves sending the private key over. GET with `ensurePublicKey=1` should be used instead.

    PUT /api/sshkey	
    Set the private key. The supported key format is privacy enhanced mail (PEM)

    GET /api/sshkey	
    Get the public key.

If you plan to copy a private SSH key from one site to another, you may do as follow;

  - download key to local: `curl "https://user:password@siteA.scm.azurewebsites.net/api/vfs/.ssh/id_rsa" -o c:\temp\id_rsa`
  - upload key to site: `curl "https://user:password@siteB.scm.azurewebsites.net/api/sshkey" --upload-file c:\temp\id_rsa`
    	
## Environment

    GET /api/environment
    Get the Kudu version
    	
## Settings

    POST /api/settings
    Create or change one or more settings

    Sample body:
    {
       "foo": "hello",
       "bar": "bye"
    }
    
    GET /api/settings
    Get the list of all settings
    
    GET /api/settings/{key}
    Get the value of a setting
    
    DELETE /api/settings/{key}
    Delete a setting
    	
## Diagnostics

    GET /api/dump	
    Get all the diagnostic logs as a zip file

## Diagnostics/Settings

    POST /api/diagnostics/settings
    Create or change a setting
    
    GET /api/diagnostics/settings
    Get the list of all settings
    
    GET /api/diagnostics/settings/{key}	
    Get the value of a setting
    
    DELETE /api/diagnostics/settings/{key}
    Delete a setting

Sample of available settings.

    {
        "AzureDriveEnabled": false(|true),
        "AzureDriveTraceLevel": "Error(|Information|Warning|Verbose)",
        "AzureTableEnabled": false(|true),
        "AzureTableTraceLevel": "Error(|Information|Warning|Verbose)",
        "AzureBlobEnabled": false(|true),
        "AzureBlobTraceLevel": "Error(|Information|Warning|Verbose)",
    }

## Logs

    GET /api/logs/recent
    Retrieve application logs found in the LogFiles\Application folder, starting with the most recent entries. 
    The following query string parameter(s) are supported:
    - top: the number of log entries to return (default is 100, maximum is 1000)

## SiteExtensions

    GET /api/extensionfeed
    List all extension package infos available on the online (remote) server.  The following query strings are supported.
    - filter: matching string

    GET /api/siteextensions
    List all extension package infos currently installed.  The following query strings are supported.
    - filter: matching string

    GET /api/extensionfeed/{id}
    Get a package info with {id} from remote store.

    GET /api/siteextensions/{id}
    Get a package info with {id} currently installed.

    PUT /api/siteextensions/{id}
    Install or update the package to local machine. The payload is the package info returned by List/Get apis above.

    DELETE /api/siteextensions/{id}
    Uninstall the package with {id}.

## Sample of using REST API with PowerShell

    $username = "`$website"
    $password = "pwd"
    $base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(("{0}:{1}" -f $username,$password)))
    $userAgent = "powershell/1.0"

    $apiUrl = "https://{sitename}.scm.azurewebsites.net/api/zip/site/wwwroot"
    $filePath = "C:\Temp\books.zip"
    Invoke-RestMethod -Uri $apiUrl -Headers @{Authorization=("Basic {0}" -f $base64AuthInfo)} -UserAgent $userAgent -Method PUT -InFile $filePath -ContentType "multipart/form-data"