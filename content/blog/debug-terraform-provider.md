---
title: "Debugging a Terraform provider"
date: 2020-08-05T22:13:55+02:00
tags: ["Terraform"]
---

While contributing to a couple of [Terraform providers](https://github.com/databrickslabs/terraform-provider-databricks) I often found the need to set some breakpoints in the provider code and inspect what was going at runtime.

However, debugging Terraform providers is not that easy apparently... I found a way to do this VS Code, but feel free to [drop me a line](https://twitter.com/s_debruyn) if you have some other tips & tricks and I'll update this post.

The guide below works for macOS and will probably also work on Linux.

So first, make sure your Go installation is okay and that you have the official Go extension for VS Code installed (extension ID: golang.go).

Next, compile your code and install the binary. Now, this would cost a lot of time each time you changed something in your code, so I'd recommend to just symlink the binary from your project folder to your Terraform plugin folder, e.g.:

```sh
ln -s ~/projects/databricks-terraform/terraform-provider-databricks ~/.terraform.d/plugins/terraform-provider-databricks
```

Then continue by adding a debug configuration to your project. These are stored in `.vscode/launch.json`. This is the one you need to add:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Attach to Process",
            "type": "go",
            "request": "attach",
            "mode": "local",
            "processId": 0
        }
    ]
}
```

This configuration allows you to attach the debugger to an existing process.

Now set your breakpoint where you want to debugger to become available and unless your name is Barry Allen, you might want to add a short sleep in your code because you can only attach the debugger when Terraform is already running.

```go
time.Sleep(10 * time.Second)
```

Increase the amount of seconds if you notice that copying and pasting the process ID takes longer.

Next, run your Terraform command (e.g. `terraform apply`) and open a second Terminal where you enter the following command:

```sh
ps | grep terraform-provider | grep -v "grep"
```

This gives you information about the process that is running your provider binary. The second grep removes grep itself from the output as grep is also a running process.

Continue by copying the process ID (the first value in the output) and paste it in the VS Code launch configuration after `processId`.

If you open the Debugger tab in VS Code, you will find a dropdown next to the green play icon that allows you to select the launch configuration we just created. Now you can just start the Debugger using the green play icon.

VS Code might ask you for extended permissions to get access to running processes. After approving that, you should see the debugger come alive and after a few seconds your breakpoint should be hit.

Happy debugging!