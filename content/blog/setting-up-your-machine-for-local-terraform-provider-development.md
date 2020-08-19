---
title: "Setting up your machine for local Terraform provider development"
date: 2020-08-19T18:27:09+02:00
tags: ["Terraform"]
---

If you've been developing [Terraform](https://terraform.io) providers, you might have upgraded your machine to use the newly released Terraform 0.13. This version introduces a lot of changes related to where providers can be found.

Before version 0.13 you would just put the binary of your provider in `~/.terraform.d/plugins` and you could start using your provider. There is a lot more to it in Terraform 0.13 and newer.

First, create a folder on your filesystem where you want to store the providers you're working on. This folder itself has to follow [a very specific structure and naming convention](https://www.terraform.io/docs/commands/cli-config.html#explicit-installation-method-configuration). I chose to put everything inside a directory called `terraform-providers` in my home directory.

```bash
mkdir -p ~/terraform-providers/local/providers/
```

Now create a subdirectory for each provider you're working on. Include your OS and architecture (mine is macOS 64-bit). The version doesn't really matter.

```bash
mkdir -p ~/terraform-providers/local/providers/someprovider/1.0.0/darwin_amd64
```

This directory has to contain your binary provider like before. You can symlink your binary to this directory so that you don't have to copy it after each compilation.

```bash
ln -s ~/repos/terraform-provider-someprovider/terraform-provider-someprovider ~/terraform-providers/local/providers/someprovider/1.0.0/darwin_amd64/terraform-provider-someprovider
```

Next, we have to define this directory in a file called `.terraformrc` in your home directory. I used VS Code to edit it, but feel free to use any text editor you like.

```bash
touch ~/.terraformrc
code ~/.terraformrc
```

This is what we have to add in that file:

```hcl
provider_installation {
  filesystem_mirror {
    path    = "~/terraform-providers/"
    include = ["local/providers/*"]
  }
  direct {
    exclude = ["local/providers/*"]
  }
}
```

This tells Terraform to not use the Terraform Registry for providers prefixed with `local/providers`, but use our local directory instead.

Now you can use the following configuration to point to your local copy:

```hcl
terraform {
  required_providers {
    someprovider = {
      source  = "local/providers/someprovider"
      version = "1.0.0"
    }
    # add other providers here
  }
  required_version = ">= 0.13"
}
```

Initialize your project with `terraform init` to make sure the setup is working properly. If you experience any issues, you probably made a mistake in the folder structure and it's worth looking into [the documentation](https://www.terraform.io/docs/configuration/provider-requirements.html#in-house-providers).