Kudu is the engine behind [git deployments in Azure Web Sites](https://www.windowsazure.com/en-us/develop/nodejs/common-tasks/publishing-with-git/). It can also run outside of Azure.

## Getting help

* Post to the [Azure Web App forum on MSDN](https://social.msdn.microsoft.com/Forums/azure/en-US/home?forum=windowsazurewebsitespreview)
* Post an issue in this GitHub project
* Go to the [Kudu room on JabbR](http://jabbr.net/#/rooms/kudu)
* Ping [@davidebbo](https://twitter.com/davidebbo) or [@suwat_ch](https://twitter.com/suwat_ch) on Twitter


## Using git in Azure Web Sites

If you are using git to push to an Azure Web Site, this is the section you'll care most about.

### Version of Kudu currently live in Azure

See the [[Version history]] page for details.

### Features
* [Publishing a website from Source Control](https://azure.microsoft.com/en-us/documentation/articles/app-service-continous-deployment/) (official documentation)
* [[Deployment hooks]]: how to take over the deployment logic for a site
* [[Web Hooks]]
* [[Azure Site Extensions]] (see also [[Xdt Transform Samples]])
* [[Post Deployment Action Hooks]]
* [[WebJobs]]

### How-to guides

* [[Accessing the Kudu service]]
* [[Customizing Deployments]]: choosing which folder or project to deploy to the web site
* [[Deployment branch]]: choosing the branch that gets deployed when you push
* [[Deploying from GitHub]]
* [[Manually triggering a deployment]]
* [[Managing settings and secrets]]: how to modify the values when the app is deployed
* [Managing database connections in Azure Web Sites](http://blog.davidebbo.com/2012/09/managing-database-connections-in-azure.html)
* [[Accessing files via FTP]]
* [[Deploying inplace and without repository]]
* [[Using a custom web.config for Node apps]]
* [Sending an email when your Azure web site deployment completes](http://blog.amitapple.com/post/56390805814/deployment-email)

### Understanding Kudu and Windows Azure Web Sites

* [[Kudu architecture]]
* [[REST API]]
* [[Enabling continuous deployment from Github, Codeplex and Bitbucket|Continuous deployment]]
* [[File structure on Azure]]
* [[Azure Runtime Environment]]
* [[Azure Web App sandbox]]
* [[Understanding deployment credentials|Deployment credentials]]
* [[How Kudu deploys sites|Deployment]]
* [[Configurable settings]]
* [[Deployment Environment]]
* [[Information about other Azure Web Apps features]]

### Troubleshooting

* [[Known issues]]
* [[Investigating Issues]]
* Using the [[Kudu Console]]
* [[Diagnostic Log Stream]]
* [[Investigating Continuous Deployment]]
* [[Process Threads list and minidump/gcdump/diagsession]]

## Building and running Kudu from the github sources

Read this is you are interested in contributing to Kudu

* [[Contributing]]
* [[Getting Started]]
* Blog post: [Developing Kudu locally](http://blog.davidebbo.com/2012/06/developing-kudu-locally-and-on-azure.html)
* [[Deploy locally-built private Kudu to Azure]]
* [[Project Structure]]
* [[Running Tests]]
* [[Deploying to a server]]
* [[Git workflow]] for contributors

## Miscellaneous

* [[Blog posts and screencasts]]
* [[Cool tweets from the community|Cool user tweets]] [[And some specifically about WebJobs|Cool WebJobs tweets]]