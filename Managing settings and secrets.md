When you deploy a site to Azure using git, all the files come from a git repository. If your site needs any secrets at runtime (e.g. credentials to access some external resource), it's generally a bad idea to commit those secrets to your repository. Even if it's a private repo, you may not want every developer working on the project to have access to those secrets.

In other cases, it's not really about the values being secrets, but simply about needing different values at runtime in Azure from the ones you use at development time on your own box.

The way to solve this in Azure Web Sites is to use runtime app settings. These are just a bunch of Key/Value pairs that you can find in the Configure section of the portal. They work in slightly different ways for .NET sites and for other sites.

## Using App Settings with .NET sites ##

Suppose you have a .NET site that has something like this in its web.config:

	  <appSettings>
	    <add key="MyKey" value="Some default value" />
	  </appSettings>

And somewhere in your code, you use `ConfigurationManager.AppSettings["MyKey"]` to access the value. The value you hard code into your web.config will typically be some default that is safe for the world to see. Or you may simply not have anything in there, and have the code fall back to some default behavior.

But when you deploy your site to Azure, you'd like this setting to take on a specific value, which may very well be a secret.

All you have to do here is add an entry in the portal's 'app settings' section (in the Confugure tab for an site). You make the Key be "MyKey", and the value is whatever you want it to be.

Then when the site runs in Azure, the call to `ConfigurationManager.AppSettings["MyKey"]` will return this value instead of the one that's hard coded in your web.config.

Note that in order to do this, **it never actually modifies your physical web.config file**. Instead, it directly affects the runtime behavior of the .NET configuration system to make it use those alternative values. You can verify this by downloading your webconfig file (see [[Accessing files via ftp]]), and seeing that it is unchanged.


## Using App Settings with arbitrary sites ##

Outside of the .NET world, sites tend to simply rely on environment variables for settings, instead of having them in a web.config-like file.

e.g. in a Node app, you might have:

	var mySetting = process.env.MyKey || "Some default value";

To modify this value in an Azure Web Site, you do the exact same thing as in the .NET case: you add the new key/value pair in the portal's 'app settings' section. This causes Azure to inject this environment variable into the process running your app.

Note that even in .NET app, you end up getting this environment variable. But in most cases, .NET apps ignore it and rely on web.config instead.
