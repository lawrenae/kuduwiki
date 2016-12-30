Note: This page is sourced from a 3-part [blog series](http://blog.amitapple.com/post/38417491924/azurewebsitecustomdeploymentpart1) which explains the feature in detail.

See also: [Customizing Deployments](https://github.com/projectkudu/kudu/wiki/Customizing-deployments)

-----

#Custom Deployment Scripts For Microsoft Azure Web Apps (Websites) Using Git Deployment

The coolest feature on Microsoft Azure Web Apps is the ability to deploy your website using git.
Do a `git push` and bam you're done and deployed within seconds.
The deployment process is automated, the process will look at the files on the git repository and decide which kind of website it is (asp.net, node, ...) and based on that will do the required steps for the deployment.

For example for an MVC web app it'll find the solution file and determine which project is the actual web app project and with those it'll msbuild that project, the artifacts from the build will be placed in a temporary folder and only the files that were changed will be copied to the wwwroot location for the actual deployment.

The nice thing is that files that were removed will also be removed on the wwwroot location but only if they were actually deployed the previous time (so files that are generated on the fly on the wwwroot directory won't be removed).

So what if you want to customize the deployment process, for example you want to run your tests before deploying (or after) and cancel the deployment if they fail?

That's what the custom deployment feature is about, you just need to add a file to the root of your repository with the name `.deployment` and the content:

    [config]
    command = YOUR COMMAND TO RUN FOR DEPLOYMENT

this command can be just running a script (batch file) that has all that is required for your deployment, like copying files from the repository to the web root directory for example.

You also get some environment variables that you need in order to get and put things in the right place:

* DEPLOYMENT_SOURCE - The path for the root of your repository (In Azure).
* DEPLOYMENT_TARGET - The wwwroot path (the deployment destination directory).
* DEPLOYMENT_TEMP - Path to a temporary directory that will be removed after the deployment.
* MSBUILD_PATH - Path to msbuild executable.

With all of those you can create your own deployment script, a simple one for example:

    @echo off
    echo Deploying files...
    xcopy %DEPLOYMENT_SOURCE% %DEPLOYMENT_TARGET% /Y

This script will copy all of the files from your repository to the wwwroot directory.

But to make things easier, you can use the azure-cli tool which will actually generate a deployment script for you that will do exactly the same deployment process as the default one but now you are able to update that script and add (or remove) your own steps.

-----

# Custom Deployment Script Generator


**With** Microsoft Azure Web Apps you can deploy your website by simply pushing your git repository, this will automatically deploy your website, and if you want to control this deployment flow you can use the custom deployment feature.

To make it easy on us there is a custom deployment script generator feature in the azure-cli tool that will simplify the whole process, basically it will generate a script that has the same logic as the automatic deployment one only now you can change it and also run it locally to test it.

All you have to do is:

a. Install [node.js](http://nodejs.org/), this is required to run the azure-cli tool.

b. Install the [azure-cli](http://www.windowsazure.com/en-us/manage/linux/other-resources/command-line-tools/) tool, it'll also give you some cool features on managing azure related resources directly from the command-line:

    npm install azure-cli -g

c. Switch azure console to _asm_ mode

    azure config mode asm

d. Go to the root of your repository (from which you deploy your site).

e. Run the custom deployment script generator command:

    azure site deploymentscript [options]

You can find help on the [options] by using the -h flag.

For example to create a script for deploying an asp.net web application you do the following:

    azure site deploymentscript --aspWAP pathToYourWebProjectFile.csproj -s pathToYourSolutionFile.sln

Or for deploying a node website:

    azure site deploymentscript --node

Any of these commands will generate the files required to deploy your site, mainly:

> * .deployment - Contains the command to run for deploying your site.
> * deploy.cmd - Contains the deployment script.

Now you can edit the deploy.cmd file and add your custom steps.

Looking at the file you'll notice that it may look a bit complicated at first but actually most of the code there is to make sure the script can run locally so the place to look for the deployment logic is here:

    ::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
    :: Deployment
    :: ----------

    ... [deployment steps]

    ::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

It'll contain very few steps and that's the right place to put your custom steps, for example: running your tests.

In this script you'll also notice the use of a tool called [KuduSync](https://github.com/projectkudu/KuduSync), this tool is the publishing tool which copies the website from the repository (or temporary directory) to the wwwroot directory.

The genius of this tool is that it'll only copy the files that were changed, it'll also remove the files that don't exist on the source but only those that were previously on the source so it won't remove any runtime generated files (it does that by keeping a files list of each deployment, this is what the manifest files are about).

After adding your own logic to the deployment script you can run in locally and test it to make sure it does what you need it to, it'll publish your site to a sub-directory called *artifacts* so make sure you're not add the files there to your repository.

When the script is tested, add the generated files to your repository (.deployment and deploy.cmd, for node also web.config and iisnode.yml) and push your repository to your Microsoft azure web app and see your custom deployment running.

-----

# Customizing site deployment based on site's app settings in Azure Web Apps (Websites)

**In** previous posts I talked about [Microsoft Azure Web App's custom deployment feature](/post/38417491924/azurewebsitecustomdeploymentpart1) and the [tool to easily generate a deployment script](/post/38418009331/azurewebsitecustomdeploymentpart2), in this post I'll do a step by step guide on writing a custom deployment script.

Let's say we have two websites in azure, one is a node.js website and the other is an mvc4 web application, both sites sources are in the same repository so we need a deployment script that handles differently based on which website it is deploying.

[Full sample repository](https://github.com/amitapl/CustomScriptSample)

Some Prerequisites:

1. Assuming git is installed and the websites are in a local git repository.
2. Install [node.js](http://nodejs.org/).
3. Install [azure-cli](http://www.windowsazure.com/en-us/manage/linux/other-resources/command-line-tools/) by 
running the following command: `npm install azure-cli -g`


### (a) First let's generate a deployment script for the node.js website:
- Go to the root of the repository.
- Enter the following command: `azure site deploymentscript --node --sitePath nodejs`
- Where "nodejs" is the path to the node.js website directory.

![2012-12-20-azure-website-custom-deployment-part-3_md1](https://cloud.githubusercontent.com/assets/4468967/10491864/aa9bc2c2-725d-11e5-9363-a65393e2c270.png)

- Notice the files that were generated:
> - .deployment - a file telling which command to run for deployment (currently deploy.cmd).
> - deploy.cmd - the deployment script.
> - nodejs\web.config - configuration for iis to run node.js.
> - nodejs\iisnode.yml - this file allows some configuration settings relating to node.js, [more info on iisnode.yml](http://tomasz.janczuk.org/2012/05/yaml-configuration-support-in-iisnode.html "more info on iisnode.yml")

* Rename deploy.cmd to deploy.node.cmd: `move deploy.cmd deploy.node.cmd`

### (b) Let's generate a deployment script for the mvc4 web application:

At the root of the repository enter the command:

    azure site deploymentscript --aspWAP mvc4\Mvc4WebApplication\Mvc4WebApplication.csproj -s mvc4\Mvc4WebApplication.sln

![2012-12-20-azure-website-custom-deployment-part-3_md2](https://cloud.githubusercontent.com/assets/4468967/10491869/aaa01520-725d-11e5-9a77-9d7410e05f30.png)

* Rename deploy.cmd to deploy.mvc4.cmd: `move deploy.cmd deploy.mvc4.cmd`

*NOTE: You can also edit this generated file (deploy.cmd) with any custom steps you have, you can also test it on your machine simply by running it, it will publish your website to %REPOSITORY_ROOT%\artifacts.*

### (c) To decide which script should run, based on the website we are currently deploying, we'll use the "app settings" feature in Microsoft Azure Web Apps:

* Create a deploy.cmd file under the root of the repository with the following:

<script src="https://gist.github.com/4349245.js"></script>

* %SITE_FLAVOR% will be set by Windows Azure with the value we'll give it in the management portal.

* Add all generated files and commit them to the repository.

### (d) Now let's try to push our repository to our Microsoft Azure Web App:

    git push WA master

![2012-12-20-azure-website-custom-deployment-part-3_md3](https://cloud.githubusercontent.com/assets/4468967/10491865/aa9d9868-725d-11e5-9ab0-9401ab59076f.png)

* We receive an error and the deployment fails since we still haven't set the app setting yet, so let's do that.

* Go to the website on windows azure management portal and add under the CONFIGURATION tab under "app settings" a setting with name SITE_FLAVOR and value nodejs/mvc3 (based on the current site we're configuring).

![2012-12-20-azure-website-custom-deployment-part-3_md4](https://cloud.githubusercontent.com/assets/4468967/10491923/0959494c-725e-11e5-84d6-4f2a723d248f.png)

![2012-12-20-azure-website-custom-deployment-part-3_md5](https://cloud.githubusercontent.com/assets/4468967/10491867/aa9dda8a-725d-11e5-9203-f54c55176936.png)

* Click on the "Save" button.

* Now we can either push our changes again (we'll need a new commit, even an empty one, otherwise it'll tell us that nothing has changed and the deployment won't reinitiate).
* Or we can go to the DEPLOYMENTS tab in Windows Azure portal, select the last deployment which failed and push the RETRY button to retry the deployment.

![2012-12-20-azure-website-custom-deployment-part-3_md6](https://cloud.githubusercontent.com/assets/4468967/10491868/aaa00efe-725d-11e5-9225-8764e54ab336.png)

### That's it, now we have a working mvc4/node.js website

![2012-12-20-azure-website-custom-deployment-part-3_md7](https://cloud.githubusercontent.com/assets/4468967/10491866/aa9ddada-725d-11e5-8311-827ba59a8f05.png)

![2012-12-20-azure-website-custom-deployment-part-3_md8](https://cloud.githubusercontent.com/assets/4468967/10491870/aaacacfe-725d-11e5-9395-04018262eb68.png)

**NOTE: Another improvement we could do here is to store the repository on GitHub/Bitbucket and connect them to our 2 sites, now every time we push to GitHub/Bitbucket, both of our sites will be deployed.**

The repository I've used can be cloned from [here](https://github.com/amitapl/CustomScriptSample).
