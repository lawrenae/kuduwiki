The Azure Portal supports setting up continuous deployment from a VSTS account. One tricky part is that it requires your VSTS account to be linked with one of your Azure subscription.

## Linking your VSTS account to your Azure subscription

The last step is that you need to link your VSTS account to your Azure subscription (see also [this post](https://www.visualstudio.com/en-us/get-started/setup/set-up-billing-for-your-account-vs) on this topic).

To do this, go to the Azure Portal, click Browse and search for 'Team':

![image](https://cloud.githubusercontent.com/assets/556238/13531726/d8c9608e-e1dc-11e5-83a0-d35df99cc62b.png)

Now select the relevant Team Services account, and click Link:

![image](https://cloud.githubusercontent.com/assets/556238/13531647/7fb69b24-e1dc-11e5-9bf1-c313cfe04cb6.png)

And you're done! You will now be able to set up continuous deploying to your git repos hosted in VSTS.

## I still cannot see my project in the choose project list

Below should help you trouble shoot the issues.

- Sign into `https://<youraccount>.visualstudio.com/` with the same account and browse the project list.  See if you can't see the project, you are either have insufficient permission to setup continuous with this project.
- Is your project a git project?  We only support linking with VSTS git project.
- Are you a Project Admin to the project?  We only support linking with VSTS git project that you are Project Admin.  To check, browse to the project and open the project settings tab (gear icon on the top center tab).  See if the `Service Hooks` tab is visible and you are able to navigate into it without issue.   If not, you are not a Project Admin of the project.   

 