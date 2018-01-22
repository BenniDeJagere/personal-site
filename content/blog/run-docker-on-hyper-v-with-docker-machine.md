+++
date = "2015-10-11T23:43:40+02:00"
title = "Run Docker on Hyper-V with Docker Machine"
tags = ["Docker", "virtualization", "Windows", "Hyper-V"]
+++

Docker is awesome, right? And thanks to [boot2docker](http://boot2docker.io) Windows users were no longer left out of the fun.

Still, setting everything up could be a PITA and you had to install Oracle VirtualBox to use it as Docker containers were actually run inside of a VM (which was the purpose of boot2docker). If you use Hyper-V regularly, you'll notice that you can't have it both ways. Scott Hanselman [figured out a way](http://www.hanselman.com/blog/SwitchEasilyBetweenVirtualBoxAndHyperVWithABCDEditBootEntryInWindows81.aspx) to make switching between the two a little bit less painful, but you still had to reboot your machine if you wanted to use VirtualBox.

Time to fix that! You can now use Hyper-V to run the boot2docker VM and connect to Docker on Windows. I guess that this has been possible since boot2docker was released, but it wasn't that straightforward to set up at the time. Enter [Docker Machine](https://docs.docker.com/machine/).

TL;DR: Docker Machine is a cross platform utility to help you set up, manage and connect to Docker. It automagically (well, most of the time) picks the best way to set up Docker in your environment. That's the boot2docker VM on Windows with either VirtualBox either Hyper-V, plain old Docker on Linux etc.

Docker still recommends using VirtualBox on Windows, but Docker Machine supports [tons of other drivers](https://docs.docker.com/machine/drivers/): AWS, Digital Ocean, Google Compute, Azure, Rackspace, Hyper-V...

So how do you install Docker Machine on Windows with Hyper-V? Here we go.

## Setting up Hyper-V and network connections

Make sure the Hyper-V role is installed on your computer and the *Hyper-V Manager* works fine. [I still had to fix a small issue on Windows 10]({{< relref "blog/fix-hyper-v-errors-on-windows-10.md" >}}).

Next, create a *Virtual Network Switch*. Pick *internal* as type.

Then go to your Network Connections, open the properties of your active internet connection and share the connection with the newly created virtual network switch. This will make sure that the IP of the boot2docker VM never changes and it still has internet access.

## Download and *install* Docker Machine

You don't need Docker Toolbox if you're not going to use Oracle VirtualBox. Just download the latest version of Docker Machine from [GitHub](https://github.com/docker/machine/releases/), rename it to `docker-machine.exe` and put it in your *PATH*. That's all there is to it.

## Create a new Docker Machine with the Hyper-V driver

Now open an administrative command prompt (you need to be an administrator to create new virtual machines) and execute the following command to create a new Docker Machine named *boot2docker*. Make sure to replace `My Internal Switch` with the name of the internal switch you created before. You can add `--hyper-v-memory xxxx` before the name of the machine to change the default amount of memory (it's dynamic) from 1024 to something else.

```shell
docker-machine create --driver hyperv --hyperv-virtual-switch "My Internal Switch" boot2docker
```
	
If all went well, you should get a message saying you still need to set the environment variables to connect to the machine. Running the following command will give you a few more commands to run which set the required variables.

```shell
docker-machine env --shell cmd boot2docker
```
	
## Connect to your boot2docker instance
	
You can now connect to your newly created boot2docker instance with the following command as it should already be running:

```shell
docker-machine ssh boot2docker
```
	
A few more helpful commands:

```shell
docker-machine --help
docker-machine stop boot2docker
docker-machine start boot2docker
docker-machine restart boot2docker
docker-machine kill boot2docker
```
	
If you need more fine-grained control, you can still use the *Hyper-V Manager* to adjust more settings.