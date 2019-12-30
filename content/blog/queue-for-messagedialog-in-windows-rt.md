+++
date = "2014-08-20T00:00:00+00:00"
title = "Queue for MessageDialog in Windows RT"
tags = ["Windows RT", "CSharp"]
aliases = ["2014/queue-for-messagedialog/"]
+++

When I write Windows Store applications, I use MessageDialog a lot.
It's the easiest way to show a quick informative pop-up message or a question to the user.
However, when you tend to use this quite often, you'll probably run into a problem.

The Windows Runtime framework doesn't allow you to stack MessageDialogs, queue them etc.
So if you need to display one, you'll have to make sure that no other MessageDialog is already open.
Checking if there is an open MessageDialog is not that easy unless you keep references to all the MessageDialogs you create.
There are a few cases in which this would be annoying.

The singleton class below will make sure that you can queue MessageDialogs and that this problem will never occur.

Instead of using

```C#
myDialog.ShowAsync();
```

use

```C#
MessageDialogQueue.PushMessageDialog(myDialog);
```

The dialog will be displayed after all previous MessageDialogs are closed. Do this for all your MessageDialogs or you'll still run into the problem I mentioned.

{{< gist sdebruyn efaf5ae8e0af3ac2e0d9 >}}
