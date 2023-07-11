+++
date = "2018-10-15T23:00:52+01:00"
title = "Moving from a \"work\" to a \"personal\" Microsoft account"
tags = ["Microsoft", "Microsoft Azure"]
expiryDate = "2023-07-01"
images = ["/img/post/2018/10/15/msworkpersonal.png"]
+++
If you've found this blog post, then you probably have found yourself in Microsoft's account mess.

The following dialog is the cause of all evil:

![](/img/post/2018/10/15/msworkpersonal.png)

I'm not going into details on why this exists or how it works, but if you're using Microsoft Azure, you've probably ever ended up using the work account for some subscription. Which is not something you'd want. This is a separate account without decent security like 2-factor authentication.

## On Azure

In your Azure Active Directory user list, you'll notice that some of your users have access with their work account instead of their personal one. These are the ones marked with "External Azure Active Directory" and this is probably because they've ever had an Office 365 subscription of some sort on that email address. As you can see, you can also use a Microsoft Account. That's the personal one.

![](/img/post/2018/10/15/azureusers.png)

### Switch from work to personal account

#### Part 1: Azure administrator

First, remove the user that wants to switch from the complete active directory. You can simply click on the user in the list above and then click on _Delete user_.

Next, you'll need to go to the _Deleted users_ list where you have to permanently delete that user.

Now, make sure that guest users have enough permissions. In your Azure Active Directory, go to the _Users_ list and then continue to _User settings_. There's a link to _External collaboration settings_ specific for guest users.

![](/img/post/2018/10/15/azurecollaboration.png)In the next step, you have to invite that user again from the users list. But make sure to not immediately continue by accepting the invite.

#### Part 2: Azure user

Before opening the invitation link (which is sent by mail), open an incognito/private browsing window and log in to your personal Microsoft account at [myaccount.microsoft.com](https://myaccount.microsoft.com). Continue to the [Azure portal](https://portal.azure.com "Azure"). Mark all the checkboxes that ask to stay logged in.

Now, paste and navigate to the invitation link in the same incognito/private browsing tab so that you're 100% sure that it uses the same session.

Azure  now asks you to accept the invitation without asking you to log in. If it's asking for a password, then something went wrong.

#### Part 3: Azure administrator

Finally, restore the user's permissions/roles/groups and you'll notice that the user is now a Microsoft Account instead of an External Active Directory User.
