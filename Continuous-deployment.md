## Setting up continuous deployment from the Azure portal

The Azure portal makes it easy to set up continuous deployment from GitHub or Bitbucket.

This documents discusses what happens under the cover. In most cases you don't need to worry about this. It can be useful to set up continuous deployment from a git host that the portal doesn't directly support, like GitLab.

## Setting up continuous deployment using Resource Explorer

You can set up continuous deployment using Resource Explorer (https://resources.azure.com/), which works at the Azure REST API level. While more 'low level' than the portal, it is in practice very easy to do.

Use the following steps:

- First, create your Web App using the portal or any other technique
- Go to https://resources.azure.com/
- Find your site, either using the search UI or by expand the tree
- Under the site, open up the `sourcecontrols/web` node in the tree
- Click on the Edit button (you may need to first set Explorer to Read/Write mode)
- In that json, you will see something that looks like this: 


```json
  "properties": {
    "repoUrl": null,
    "branch": null,
    "isManualIntegration": false,
    "deploymentRollbackEnabled": false,
    "isMercurial": false
  }
```

- Set the repoUrl property to your repo, e.g.

    "repoUrl": "https://github.com/davidebbo-test/Hello_World_Demo",

- (optional) if you want to use a non-default branch, set the `"branch"` property as well
- Click the PUT button to apply the change
- You're done! Now if you go to the portal, you should see that the site is hooked up, and the initial commit will get deployed

## Setting up continuous deployment using manual steps

**Note**: Unlike the Resource Explorer case above, the repoUrl property should not be changed when configuring manually. The webhook payload should already contain the remote repository URL.

### Create an Azure Web App and enable source control with an 'External Repository'

Use your repo's URL when setting this up. This will allow the 'sync' button in the portal to work.

### Set up a Web Hook so Kudu gets notified when a deployment happens

The hook URL is the `/deploy` path on the Kudu service. So it looks like `https://mysite.scm.azurewebsites.net/deploy`. But since the service uses basic auth, you need to pass the creds in the URL. Typically, you'll want to use the site-level credentials here and not the user credentials (see [[Deployment-credentials]]). So the full URL will look like `https://$mysite:BigRandomPassword@mysite.scm.azurewebsites.net/deploy`

The easiest way to get this URL is to directly copy it from the Azure Portal. You'll find it in the Configure tab, in the Deployment Trigger Url field.

Once you have the URL, you can set it as a GitHub/Bitbucket/GitLab hook.


### For private repos, set up a deploy key

If the repo is private, you will also need to set up an SSH 'deploy key' on GitHub/Bitbucket/GitLab.

This can be done using the following steps:
- Take the full deploy URL above, and replace `/deploy` with `/api/sshkey?ensurePublicKey=1`. So it'll look like `https://$mysite:BigRandomPassword@mysite.scm.azurewebsites.net/api/sshkey?ensurePublicKey=1`
- Run `curl` on this URL, which returns an SSH public key. It's returned as a JSON string, so you'll need to remove the quotes. It should look like `ssh-rsa AAAAB3NzaC1etc...`
- Set this string as a 'deploy key' on your GitHub/Bitbucket/GitLab repository.

### Test that everything works

The easiest way to test it is to got to GitHub/Bitbucket/GitLab and click 'test' on the Web Hook you set up. Or you can try simply pushing a chance to your repo. 