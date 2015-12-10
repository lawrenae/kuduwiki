There are various things that you can configure using App Settings, which you can find on the Configure page in the Windows Azure portal.

Note that in addition to using App Settings, you can specify those settings in your [[.deployment file|Customizing-deployments]]. This is useful if you want the setting to be part of your repository.


## Repository and deployment related settings

### Adding flags to the msbuild command line

Use this flag to add things at the end of the msbuild command line, such that it overrides any previous parts of the default command line.

e.g. to choose the Debug build configuration (default is Release) and apply a chained config transform, you could have:

    [config]
    SCM_BUILD_ARGS=-p:Configuration=Debug;PublishProfile=MyChainedTransform

### Taking over the script generator command line

Kudu uses the `azure site deploymentscript` command described [here](http://blog.amitapple.com/post/38418009331/azurewebsitecustomdeploymentpart2) to generate a deployment script. By default, it figures out what parameters to pass by looking at the files in the repo to determine the project type (e.g. Node, ASP.NET, ...).

But in some cases, you may want to override that and take control of the command line, which you can do using this setting.

e.g. to force your repo to be treated as a plain web site (no build), you can use:
 
    SCM_SCRIPT_GENERATOR_ARGS=--basic -p FolderToDeploy

### Changing the repo and deployment paths, and not using a repo at all

Please see [[Deploying inplace and without repository]] for information on using the `SCM_REPOSITORY_PATH`, `SCM_NO_REPOSITORY`, `PROJECT` and `SCM_TARGET_PATH` flags.

### Disabling the update of git submodules

By default, Kudu automatically update submodules before doing a deployment. To turn that off:

    SCM_DISABLE_SUBMODULES=1

Note: this is support as an Azure App Setting, but not in the .deployment file.

### Using a git shallow clone in Continuous Deployment scenarios

For large repos, you can make Kudu use a shallow clone when it clones your repo from GitHub or Bitbucket, which can save disk space. Shallow clones can be tricky, so make sure you understand what they are before using this. It is off by default. To turn it on:

    SCM_USE_SHALLOW_CLONE=1

Note: this is support as an Azure App Setting, but not in the .deployment file.

### Don't touch web.config at the end of the deployment

By default, it gets touched. Sometimes, it's sub-optimal as it can causes an unnecessary restart. To avoid it, set:

    SCM_TOUCH_WEBCONFIG_AFTER_DEPLOYMENT=0

### Customize post deployment action directory

After deployment, Kudu will execute, if exists, `postdeployment` script under /site/deployments/tools directory.  To customize to different directory, one can set the following setting.

    SCM_POST_DEPLOYMENT_ACTIONS_PATH=<path> 

### Don't build and deploy during git push

With this flag, a `git push` updates the server repo, but does not trigger a deployment. Note that this only applies when git pushing directly to the Kudu git endpoint, and not in continuous integration scenarios like GitHub/Bitbucket.

    SCM_DISABLE_DEPLOY_ON_PUSH=1

## Diagnostic related settings

### Changing the trace level

By default, it is set to 1, but you can get more tracing with higher values, up to 4. e.g.

    SCM_TRACE_LEVEL=4

### Changing the timeout before external commands are killed

By default, when your build process launches some command, it's allowed to run for up to 60 seconds without producing any output. If that is not long enough, you can make it longer, e.g. to make it 10 minutes:

    SCM_COMMAND_IDLE_TIMEOUT=600


### Changing the timeout of the log streaming feature

When using the log streaming feature, by default it times out after 30 minutes of inactivity. To change it to 15 minutes (unit is seconds):

    SCM_LOGSTREAM_TIMEOUT=900


### Using git.exe instead of libgit2sharp for git operations

    SCM_USE_LIBGIT2SHARP_REPOSITORY=0

## Runtime settings

The following settings must be set in the Azure App Settings, and cannot be overridden in the .deployment file (since they are not deployment settings)

### Change the Node version

Used the change the version of Node that is used by default

    WEBSITE_NODE_DEFAULT_VERSION=0.10.5

### Change the Npm version

Used the change the version of Npm that is used by default

    WEBSITE_NPM_DEFAULT_VERSION=1.2.30

### Add user profile support for a site

See [this forum thread](http://social.msdn.microsoft.com/Forums/windowsazure/en-US/b919f872-2d4c-4c94-9cf1-9494e97d35ce/deployments-fails-when-referencing-systemxaml?forum=azuregit) for details. Note that this is only available for sites running in Basic or Standard mode.

    WEBSITE_LOAD_USER_PROFILE=1

### Turning on the 'local cache' feature.

**note**: this feature is still experimental and not yet ready for production use!

This feature copies the site bits to the faster local drive before running them.

    WEBSITE_LOCAL_CACHE_OPTION=Always

Also, the size of the cache can be changed (default is 300MB):

    WEBSITE_LOCAL_CACHE_SIZEINMB=500

### Disable the use of private site extensions

    WEBSITE_PRIVATE_EXTENSIONS=0

### Set the time zone

    WEBSITE_TIME_ZONE=Eastern Standard Time

### Diagnostics related settings

The name (or relative path to the LogDirectory) of the file where internal errors are logged, for troubleshooting the listener:

    DIAGNOSTICS_LASTRESORTFILE=logging-errors.txt

The settings file, relative to the web app root:

    DIAGNOSTICS_LOGGINGSETTINGSFILE=..\diagnostics\settings.json

The log folder, relative to the web app root:

    DIAGNOSTICS_TEXTTRACELOGDIRECTORY=..\..\LogFiles\Application

Maximum size of the log file (Default: 128 kb):

    DIAGNOSTICS_TEXTTRACEMAXLOGFILESIZEBYTES=200000

Maximum size of the log folder (Default: 1 MB):

    DIAGNOSTICS_TEXTTRACEMAXLOGFOLDERSIZEBYTES=2000000

Timeout in milliseconds to keep application logging on (Default is 43200000, which is 12 hours):

    DIAGNOSTICS_TEXTTRACETURNOFFPERIOD=43200000

### Don't use the SCM site for WebDeploy

    WEBSITE_WEBDEPLOY_USE_SCM=false

### Use the same process for the user site and the scm site

    WEBSITE_DISABLE_SCM_SEPARATION=true

### Pre-start the SCM site as part of site creation (needs to be set in the site creation call)

    WEBSITE_START_SCM_ON_SITE_CREATION=true

### Site Extensions gallery feed url

Each site gets the site extensions feed from a configurable Url. If it is not set, the behavior is equivalent to

    SCM_SITEEXTENSIONS_FEED_URL=http://www.siteextensions.net/api/v2/