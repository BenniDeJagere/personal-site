---
title: "Installing the Azure Event Hubs Python SDK on Raspberry Pi OS 64-bit"
date: 2020-08-11T22:16:24+02:00
tags: ["Python", "Microsoft Azure", "Azure Event Hub", "Raspberry Pi", "Debian"]
---

Since we're going through some heat waves in Europe, I thought it might be interesting to start measuring the humidity, temperature and pressure in my apartment. To do so, I decided to use my Raspberry Pi 3 and the Pi Sense HAT running a Python script constantly sending measurements to an Azure Event Hub.

On my Raspberry Pi I'm using the beta 64-bit Raspberry Pi OS (used to be called Raspbian) so it's a Pi-flavoured version of Debian for aarch64 (ARM64).

This all seemed fairly simple until I had to install the Azure Event Hubs Python SDK on the Pi. The command to install the SDK is

```sh
pip3 install azure-eventhub
```

The SDK relies on [`uamqp`](https://github.com/Azure/azure-uamqp-python) and tries to install it as a dependency, but there are [no wheels yet for aarch64](https://github.com/Azure/azure-uamqp-python/issues/15). So it tries to compile the C library while installing the dependencies. This did not seem to work on my Raspberry Pi as the compilation went out of memory.

You can start by decreasing the GPU memory allocation to 16MB (freeing up RAM for the CPU) with the `raspi-config` utility.

Next, you can increase the size of the swap file by editing `/etc/dphys-swapfile`.

```sh
# /etc/dphys-swapfile - user settings for dphys-swapfile package
# author Neil Franklin, last modification 2010.05.05
# copyright ETH Zuerich Physics Departement
#   use under either modified/non-advertising BSD or GPL license

# this file is sourced with . so full normal sh syntax applies

# the default settings are added as commented out CONF_*=* lines


# where we want the swapfile to be, this is the default
#CONF_SWAPFILE=/var/swap

# set size to absolute value, leaving empty (default) then uses computed value
#   you most likely don't want this, unless you have an special disk situation
CONF_SWAPSIZE=2048

# set size to computed value, this times RAM size, dynamically adapts,
#   guarantees that there is enough swap without wasting disk space on excess
#CONF_SWAPFACTOR=2

# restrict size (computed and absolute!) to maximally this limit
#   can be set to empty for no limit, but beware of filled partitions!
#   this is/was a (outdated?) 32bit kernel limit (in MBytes), do not overrun it
#   but is also sensible on 64bit to prevent filling /var or even / partition
#CONF_MAXSWAP=2048
```

Next, I tried compiling the uamqp dependency first before retrying to install `azure-eventhub` again.

To do so, run the following commands:

```sh
# Install the required dependencies to compile uamqp
sudo apt install -y build-essential libssl-dev uuid-dev cmake libcurl4-openssl-dev pkg-config python3-dev python3-pip curl

# Clone the C library source and submodules
git clone --recursive https://github.com/Azure/azure-uamqp-c.git

# Create a build directory, switch to it and set it up
mkdir cmake
cd cmake
cmake ..

# Build
cmake --build .

# Install
sudo make install

# Install the Python package without the binary
pip3 install uamqp --no-binary :all:
```

If you now run `pip3 install azure-eventhub` again, the installation will probably succeed within a few seconds.