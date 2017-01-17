We get a lot of questions from users that look more or less like "*I deploy my site and it doesn't run correctly*" or "*I deployed and I still see old content when I browse*". When that happens, the first thing we need to do is determine whether it is a deployment issue or a runtime issue. This article explains the difference, and how to help isolate which one it is.

## Deployment issues

A deployment issue is an issue that causes the wrong set of files to get deployed to your site folder (typically `site\wwwroot`), or that causes some files not to get deployed at all. i.e. as a result of deployment, you expect a certain set of files to end up in that folder, but due to some problem during deployment, you end up with a different set of files.

As a result of a deployment issue, you will often end up with a non-running site, which makes it look like a 'runtime' issue. But the core issue is with the deployment. **When that happens, you should not ask yourself "*why isn't my site running correctly?*". Instead, the right question to ask is "*why aren't the right files ending up where they should be?*".**

Note that there can be all kind of deployment issues, and they can potentially happen with any deployment technology (e.g. Kudu, WebDeploy, FTP, VS Online).

When using Kudu, there are all kind of possibilities:
- something in your repo may not be quite right
- maybe you have a custom deployment script that doesn't quite work
- maybe you're running into a Kudu bug

We can help investigate those after you have made the determination that you are in fact dealing with a deployment issue.

When using VS Online, it could be a range of other things, best discussed on the [VS Online Forum](https://social.msdn.microsoft.com/Forums/vstudio/en-US/home?forum=TFService)

## Runtime issues

A runtime issue happens when the files in your wwwroot folder are exactly what they should be, but for some reason the site doesn't run correctly.

When that happens, it is no longer relevant what technique you used to deploy your site. All that matter is that you got the files in the right place. In this case, a description of your issue should focus on what your code is doing at runtime, and how it is failing, rather than on how you deployed it.

One important point here is that we are talking about the runtime **on Azure Websites**. So just because it works on your machine does not imply that you don't have a runtime issue On Azure Websites.

There are many things that could cause a runtime issue. e.g.
- your Website may not be configured correctly. e.g. wrong version of PHP, .NET runtime, etc...
- there could be an external dependency like a database that's not set up correctly
- your code could be making invalid assumptions about paths, like hard coding something that only exists on your machine
- something in the Websites runtime environment might block certain operations that work on your local box
- there could be OS differences if you are developing on a Mac, since Azure Websites runs on Windows

## Determining which type of issue you have

The simplest way is to use the [[Kudu Console]] to look at the files in your wwwroot folder. It even lets you easily download everything in that folder as a zip file, so you can analyze it locally.

It can take a little bit of effort, but it is a key step to take to help getting your issue properly investigated.

## Deployments and Web App restarts

A common misconception is that deploying content to a Web App causes the app to be restarted. The reality is that **deployment pretty much does only one thing: it deploys files into the `wwwroot` folder. It never *directly* does anything to restart the App.** 

This is true whether you use Visual Studio deployment (msdeploy), git/GitHub/etc deployment, FTP, or manually copy some files over using Kudu Console.

The key word above is *directly*, meaning the deployment doesn't make any magic API calls that cause a site restart. However, in some cases, the act of deploying files into `wwwroot` can cause some form of restart. In that sense, the deployment is `indirectly` causing a restart, but it really knows nothing about it. It's up to the Application's **runtime** to react to file change notifications and do what it thinks is right.

And different stacks do things very differently. e.g. to give a couple examples:

- In ASP.NET, touching `web.config` or any file in `bin` causes the App Domain to shut down (not restart!). And then the next request will cause a new App Domain to start.
- In a Node app, touching `web.config` causes the Node.exe process to terminate, and a new one to start.  

As such, when you have a restart issue, avoid stating the question as *"I'm deploying new version and my site is not restarting"*, as it is not directly meaningful.

Instead, the key things to state/ask are:

- What stack are you running: ASP.NET, .NET Core, PHP, Node, ...?
- What specific files did you deploy?
- Did you make sure that those new files actually made it to your `wwwroot` folder?
- What specific behavior are you expecting to see as a result of the new files, and how does it compare to what you are seeing?